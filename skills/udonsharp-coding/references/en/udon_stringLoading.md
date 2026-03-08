# String Loading (VRCStringDownloader)

`VRCStringDownloader` allows you to retrieve text data from external servers (e.g., GitHub Pages, Pastebin) at runtime. This enables dynamic updates of world information based on external JSON data.

## Implementation

Use the `VRC.SDK3.StringLoading` namespace and initiate the download using the `VRCStringDownloader.LoadUrl` method. Results are received via `OnStringLoadSuccess` or `OnStringLoadError` callbacks.

### Basic Implementation Example

```cs
using UdonSharp;
using UnityEngine;
using VRC.SDKBase;
using VRC.SDK3.StringLoading;
using VRC.Udon.Common.Interfaces;

public class RemoteTextLoader : UdonSharpBehaviour
{
    [SerializeField] private VRCUrl _url;

    void Start()
    {
        // Start the download
        // The second argument specifies the UdonBehaviour that will receive the callbacks
        VRCStringDownloader.LoadUrl(_url, (UdonBehaviour)GetComponent(typeof(UdonBehaviour)));
    }

    // Success callback
    public override void OnStringLoadSuccess(IVRCStringDownload result)
    {
        string downloadedText = result.Result;
        Debug.Log($"Downloaded: {downloadedText}");
    }

    // Error callback
    public override void OnStringLoadError(IVRCStringDownload result)
    {
        Debug.LogError($"Failed to load string: {result.Error}");
    }
}
```

## Important Notes and Limitations

### 1. Trusted Domains
If a user has "Allow Untrusted URLs" disabled in their VRChat settings, data can only be retrieved from VRChat's officially whitelisted domains (e.g., GitHub, Discord, Imgur).

### 2. Rate Limiting
There is a rate limit of **one download request every 5 seconds**. Requests exceeding this frequency are queued (up to 1000 requests).

### 3. VRCUrl Handling
For security reasons, you cannot construct a `VRCUrl` from a `string` at runtime. URLs must be defined in the Inspector or received via a `VRCUrlInputField`.

### 4. Callback Target
When passing `this` as the second argument to `LoadUrl`, you must explicitly cast your UdonSharpBehaviour to `(UdonBehaviour)`.
