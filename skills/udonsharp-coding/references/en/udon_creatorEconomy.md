# Udon Creator Economy

## Overview of Creator Economy
The VRChat Creator Economy SDK allows you to sell products, check ownership, and handle purchase-related events within your world.
These features can be used via Udon Node Graph or UdonSharp.

### Primary Types
- **UdonProduct**: A `ScriptableObject` created in your project. This acts as the physical representation of a store product within Udon.
  - To receive events in a world, the `UdonProduct` must be referenced by at least one `UdonBehaviour`.
  - Properties:
    - `ID` (string): The unique identifier of your product.
    - `Name` (string): The name of your product.
    - `Description` (string): The description of your product.
  - Methods:
    - `Equals(IProduct or UdonProduct)`: Compares if the product ID is equal to another product's ID and returns a `bool`.
- **IProduct**: An interface returned by Udon methods and events. It contains runtime information.
  - Properties:
    - `ID` (string): The unique identifier of your product.
    - `Name` (string): The name of your product.
    - `Description` (string): The description of your product.
    - `Buyer` (VRCPlayerApi): The player who purchased this product.
  - Methods:
    - `Equals(IProduct or UdonProduct)`: Compares if the product ID is equal to another product's ID and returns a `bool`.

## How to use with UdonSharp
When using Creator Economy features from UdonSharp, you use functionalities under the `VRC.Economy` namespace.

### Basic Implementation Example
```csharp
using VRC.SDKBase;
using VRC.Udon;
using VRC.Economy;

public class MyStoreScript : UdonSharpBehaviour
{
    // Product data set in the inspector
    public UdonProduct myProduct;

    // Event called when a purchase is confirmed or when an owner joins the instance
    public override void OnPurchaseConfirmedMultiple(IProduct product, VRCPlayerApi player, bool isNewPurchase, int quantity)
    {
        if (product.ID == myProduct.ID)
        {
            Debug.Log($"{player.displayName} owns {quantity} of {product.Name}. (New purchase: {isNewPurchase})");
            // Add logic here to grant items, etc.
        }
    }

    // Method to open the purchase page of the store
    public void OpenStore()
    {
        Store.OpenListing(myProduct.ID);
    }
}
```

### Important Notes
- **GameObject Activation**: If the GameObject to which the `UdonBehaviour` is attached is inactive, events will not be received.
- **Local Testing**: When testing locally on a client using methods like `Store.ListProductOwners`, placeholder names such as "VRCat, Fred, VRRat" will be returned.

## Script Reference

### Store Methods (`VRC.Economy.Store`)
You can access various features via the `Store` class.

#### Checking Ownership and Getting Players
- `Store.DoesPlayerOwnProduct(VRCPlayerApi player, UdonProduct product)` : `bool`
  - Checks if a specific player owns a product. It is recommended to use this after `OnPurchasesLoaded` as data might be unloaded. You can also pass an `IProduct` as the `product` argument.
- `Store.DoesAnyPlayerOwnProduct(UdonProduct product)` : `bool`
  - Checks if any player in the current instance owns the product. You can also pass an `IProduct`.
- `Store.GetPlayersWhoOwnProduct(UdonProduct product)` : `VRCPlayerApi[]`
  - Gets an array of all players in the instance who own the product. You can also pass an `IProduct`.

#### Menus and Store Display
- `Store.OpenWorldStorePage()`
  - Opens the store page of the current world.
- `Store.OpenGroupPage(string groupID)`
  - Opens the Group Info page of the specified group.
- `Store.OpenGroupStorePage(string groupID)`
  - Opens the Store page of the specified group.
- `Store.OpenListing(string listingID)`
  - Opens the purchase screen of a specific listing.
- `Store.OpenMarketplaceStore(string section, string listingID, string contextID = "")`
  - Opens a specific section of the marketplace (Exclusive, Avatar, World, Group).

#### Data Retrieval and Communication
- `Store.SendProductEvent(UdonBehaviour target, UdonProduct product)`
  - Sends a network event (triggering `OnProductEvent` on the target UdonBehaviour) that only product owners can execute. You can also pass an `IProduct` as the `product` argument.
- `Store.ListPurchases(UdonBehaviour target, VRCPlayerApi player)`
  - Retrieves all purchase data of the specified player and triggers the `OnListPurchases` event on the target UdonBehaviour.
- `Store.ListAvailableProducts(UdonBehaviour target)`
  - Retrieves all products used in the world and triggers the `OnListAvailableProducts` event on the target UdonBehaviour.
- `Store.ListProductOwners(UdonBehaviour target, UdonProduct product)`
  - Retrieves a list of usernames of all owners of that product (including those outside the instance) and triggers the `OnListProductOwners` event on the target UdonBehaviour. (Requires enabling "Owners Names in Udon" in the Web settings).

### VRCOpenMenu Methods
- `VRCOpenMenu.OpenAvatarListing(string avatarID)`
  - Opens the detail/purchase screen of a specifically published avatar.

### Events
Events sent to Udon when specific purchase states change or data loading completes. Override them in `UdonSharpBehaviour` to use.

- `OnPurchasesLoaded(IProduct[] products, VRCPlayerApi player)`
  - Triggered when all of a player's purchases have finished loading. An empty array is passed if there are no owned products.
- `OnPurchaseConfirmed(IProduct product, VRCPlayerApi player, bool isNewPurchase)`
  - Triggered when a purchase is loaded/confirmed.
- `OnPurchaseConfirmedMultiple(IProduct product, VRCPlayerApi player, bool isNewPurchase, int quantity)`
  - Triggered when a purchase is loaded/confirmed (quantity can also be obtained). Using this is recommended since it provides the quantity.
- `OnPurchaseExpired(IProduct product, VRCPlayerApi player)`
  - Triggered when a product, such as a subscription, expires.
- `OnProductEvent(IProduct product, VRCPlayerApi sender)`
  - Triggered when a network event sent by `Store.SendProductEvent` is received.
- `OnListPurchases(IProduct[] products, VRCPlayerApi player)`
  - Triggered as a result of calling `Store.ListPurchases`.
- `OnListAvailableProducts(IProduct[] products)`
  - Triggered as a result of calling `Store.ListAvailableProducts`.
- `OnListProductOwners(IProduct product, string[] ownerNames)`
  - Triggered as a result of calling `Store.ListProductOwners`.