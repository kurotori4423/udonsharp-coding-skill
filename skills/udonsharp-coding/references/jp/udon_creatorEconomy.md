# Udon Creator Economy

## Creator Economyの概要
VRChatのCreator Economy SDKを使用すると、ワールド内で商品の販売、所有権の確認、および購入に関連するイベントの処理が可能になります。
これらはUdon Node GraphまたはUdonSharpから利用できます。

### 主要な型 (Types)
- **UdonProduct**: プロジェクト内で作成する `ScriptableObject`。ストアの商品をUdonで扱うための実体となります。
  - ワールド内でイベントを受信するには、少なくとも1つの `UdonBehaviour` でその `UdonProduct` が参照されている必要があります。
  - プロパティ:
    - `ID` (string): 商品の一意の識別子
    - `Name` (string): 商品名
    - `Description` (string): 商品の説明
  - メソッド:
    - `Equals(IProduct または UdonProduct)`: 商品IDが別の商品IDと等しいかを比較して `bool` を返します。
- **IProduct**: Udonのメソッドやイベントから返されるインターフェース。実行時の情報が含まれます。
  - プロパティ:
    - `ID` (string): 商品の一意の識別子
    - `Name` (string): 商品名
    - `Description` (string): 商品の説明
    - `Buyer` (VRCPlayerApi): 商品を購入したプレイヤー
  - メソッド:
    - `Equals(IProduct または UdonProduct)`: 商品IDが別の商品IDと等しいかを比較して `bool` を返します。

## UdonSharpでの使用方法
UdonSharpからCreator Economyの機能を利用する場合、`VRC.Economy` ネームスペースの機能を使用します。

### 基本的な実装例
```csharp
using VRC.SDKBase;
using VRC.Udon;
using VRC.Economy;

public class MyStoreScript : UdonSharpBehaviour
{
    // インスペクタで設定する商品データ
    public UdonProduct myProduct;

    // 購入が確認されたとき、または所有者がインスタンスに参加したときに呼ばれるイベント
    public override void OnPurchaseConfirmedMultiple(IProduct product, VRCPlayerApi player, bool isNewPurchase, int quantity)
    {
        if (product.ID == myProduct.ID)
        {
            Debug.Log($"{player.displayName} が {product.Name} を {quantity} 個所有しています。(新規購入: {isNewPurchase})");
            // ここにアイテム付与などのロジックを記述
        }
    }

    // ストアの購入ページを開くメソッド
    public void OpenStore()
    {
        Store.OpenListing(myProduct.ID);
    }
}
```

### 注意点
- **GameObjectの有効化**: `UdonBehaviour` がアタッチされたGameObjectが非アクティブな場合、イベントは受信できません。
- **ローカルテスト**: `Store.ListProductOwners` などをクライアントでローカルテストする場合、"VRCat, Fred, VRRat" といったプレースホルダー名が返されます。

## スクリプトリファレンス

### Store メソッド (`VRC.Economy.Store`)
`Store` クラスを介して様々な機能にアクセスできます。

#### 所有権の確認とプレイヤーの取得
- `Store.DoesPlayerOwnProduct(VRCPlayerApi player, UdonProduct product)` : `bool`
  - 特定のプレイヤーが商品を持っているか確認します。データ未ロードの可能性があるため `OnPurchasesLoaded` 以降での使用が推奨されます。引数のproductには `IProduct` も渡せます。
- `Store.DoesAnyPlayerOwnProduct(UdonProduct product)` : `bool`
  - 現在のインスタンス内の誰か一人がその商品を持っているか確認します。引数のproductには `IProduct` も渡せます。
- `Store.GetPlayersWhoOwnProduct(UdonProduct product)` : `VRCPlayerApi[]`
  - 商品を所有しているインスタンス内の全プレイヤーの配列を取得します。引数のproductには `IProduct` も渡せます。

#### メニュー・ストア表示
- `Store.OpenWorldStorePage()`
  - 現在のワールドのストアページを開きます。
- `Store.OpenGroupPage(string groupID)`
  - 指定したグループの Group Info ページを開きます。
- `Store.OpenGroupStorePage(string groupID)`
  - 指定したグループの Store ページを開きます。
- `Store.OpenListing(string listingID)`
  - 特定の出品物（Listing）の購入画面を開きます。
- `Store.OpenMarketplaceStore(string section, string listingID, string contextID = "")`
  - マーケットプレイスの特定セクション（Exclusive, Avatar, World, Group）を開きます。

#### データ取得・通信
- `Store.SendProductEvent(UdonBehaviour target, UdonProduct product)`
  - 商品所有者のみが実行可能なネットワークイベント（対象UdonBehaviourで `OnProductEvent` を発火）を送信します。引数のproductには `IProduct` も渡せます。
- `Store.ListPurchases(UdonBehaviour target, VRCPlayerApi player)`
  - 指定したプレイヤーの全購入データを取得し、対象のUdonBehaviourに `OnListPurchases` イベントをトリガーします。
- `Store.ListAvailableProducts(UdonBehaviour target)`
  - ワールド内で使用されている全商品を取得し、対象のUdonBehaviourに `OnListAvailableProducts` イベントをトリガーします。
- `Store.ListProductOwners(UdonBehaviour target, UdonProduct product)`
  - その商品の全所有者（インスタンス外も含む）のユーザー名リストを取得し、対象のUdonBehaviourに `OnListProductOwners` イベントをトリガーします。（Web上の設定で "Owners Names in Udon" の有効化が必要）。

### VRCOpenMenu メソッド
- `VRCOpenMenu.OpenAvatarListing(string avatarID)`
  - 特定の公開アバターの詳細/購入画面を開きます。

### イベント
特定の購入状態の変化やデータロード完了時にUdonに送られるイベントです。UdonSharpBehaviourでオーバーライドして使用します。

- `OnPurchasesLoaded(IProduct[] products, VRCPlayerApi player)`
  - プレイヤーの全購入データのロードが完了した時に実行されます。所有商品がない場合は空の配列が渡されます。
- `OnPurchaseConfirmed(IProduct product, VRCPlayerApi player, bool isNewPurchase)`
  - 購入がロード・確認された時に実行されます。
- `OnPurchaseConfirmedMultiple(IProduct product, VRCPlayerApi player, bool isNewPurchase, int quantity)`
  - 購入がロード・確認された時に実行されます（購入個数も取得可能）。個数も取れるためこちらの使用が推奨されます。
- `OnPurchaseExpired(IProduct product, VRCPlayerApi player)`
  - サブスクリプション商品などの期限が切れた時に実行されます。
- `OnProductEvent(IProduct product, VRCPlayerApi sender)`
  - `Store.SendProductEvent` によって送信されたネットワークイベントを受信した時に実行されます。
- `OnListPurchases(IProduct[] products, VRCPlayerApi player)`
  - `Store.ListPurchases` が呼び出された結果として実行されます。
- `OnListAvailableProducts(IProduct[] products)`
  - `Store.ListAvailableProducts` が呼び出された結果として実行されます。
- `OnListProductOwners(IProduct product, string[] ownerNames)`
  - `Store.ListProductOwners` が呼び出された結果として実行されます。
