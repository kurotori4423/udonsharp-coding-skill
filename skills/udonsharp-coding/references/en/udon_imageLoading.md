# Image Loading (VRCImageDownloader)

`VRCImageDownloader` allows you to retrieve images (Texture2D) from external servers and dynamically apply them to materials or UI at runtime.

## Implementation

Use the `VRC.SDK3.Image` namespace. Image loading is handled via a `VRCImageDownloader` instance, and results are processed through `OnImageLoadSuccess` or `OnImageLoadError` callbacks.

### Basic Implementation Example

```cs
using UdonSharp;
using UnityEngine;
using VRC.SDKBase;
using VRC.SDK3.Image;
using VRC.Udon.Common.Interfaces;

public class RemoteImageLoader : UdonSharpBehaviour
{
    [SerializeField] private VRCUrl _imageUrl;
    [SerializeField] private Renderer _targetRenderer;
    
    private VRCImageDownloader _downloader;

    void Start()
    {
        // Initialize the downloader
        _downloader = new VRCImageDownloader();
        
        // Start the image download
        // If a material is provided as the second argument, the main texture will be automatically updated upon success
        _downloader.DownloadImage(_imageUrl, _targetRenderer.material, (UdonBehaviour)GetComponent(typeof(UdonBehaviour)), null);
    }

    // Success callback
    public override void OnImageLoadSuccess(IVRCImageDownload result)
    {
        Debug.Log("Image loaded successfully!");
        // To get the texture explicitly:
        // Texture2D tex = result.Result;
    }

    // Error callback
    public override void OnImageLoadError(IVRCImageDownload result)
    {
        Debug.LogError($"Image load failed: {result.ErrorMessage}");
    }

    // To prevent memory leaks, you must call Dispose() when the object is destroyed
    private void OnDestroy()
    {
        if (_downloader != null) _downloader.Dispose();
    }
}
```

## Detailed Configuration (TextureInfo)

You can specify detailed image loading settings by passing a `TextureInfo` object as the fourth argument to `DownloadImage`.

```cs
TextureInfo info = new TextureInfo();
info.GenerateMipMaps = true;
info.FilterMode = FilterMode.Bilinear;
info.WrapModeU = TextureWrapMode.Repeat;
info.WrapModeV = TextureWrapMode.Repeat;
info.MaxTextureSize = 2048; // Max size is 2048

_downloader.DownloadImage(_imageUrl, _targetRenderer.material, (UdonBehaviour)GetComponent(typeof(UdonBehaviour)), info);
```

## Important Notes and Limitations

### 1. Memory Management (Critical)
Always call `.Dispose()` on your `VRCImageDownloader` instance when it is no longer needed to free GPU memory. Failure to do so can lead to client crashes during long sessions.

### 2. Rate Limiting
Similar to String Loading, there is a rate limit of **one request every 5 seconds**.

### 3. Image Size Limits
The maximum supported image size is **2048x2048**.

### 4. Instance Persistence
You should keep a reference to the `VRCImageDownloader` instance as a class field. Local variables may be garbage collected before the download completes, leading to failed loads.
