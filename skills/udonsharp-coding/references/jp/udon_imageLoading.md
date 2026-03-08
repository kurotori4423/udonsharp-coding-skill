# Image Loading (VRCImageDownloader)

`VRCImageDownloader`を使用すると、実行時に外部のサーバーから画像（Texture2D）を取得し、マテリアルやUIに動的に適用できます。

## 実装方法

`VRC.SDK3.Image`ネームスペースを使用します。画像読み込みは `VRCImageDownloader` インスタンスを通じて行い、読み込み後の結果は `OnImageLoadSuccess` または `OnImageLoadError` コールバックで処理します。

### 基本的な実装例

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
        // ダウンローダーの初期化
        _downloader = new VRCImageDownloader();
        
        // 画像のダウンロード開始
        // 第2引数のマテリアルを指定すると、成功時に自動的にメインテクスチャが更新されます
        _downloader.DownloadImage(_imageUrl, _targetRenderer.material, (UdonBehaviour)GetComponent(typeof(UdonBehaviour)), null);
    }

    // 成功時の処理
    public override void OnImageLoadSuccess(IVRCImageDownload result)
    {
        Debug.Log("Image loaded successfully!");
        // 明示的にテクスチャを取得する場合：
        // Texture2D tex = result.Result;
    }

    // 失敗時の処理
    public override void OnImageLoadError(IVRCImageDownload result)
    {
        Debug.LogError($"Image load failed: {result.ErrorMessage}");
    }

    // メモリリークを防ぐため、オブジェクトが破棄される際に必ず Dispose() を呼び出す必要があります
    private void OnDestroy()
    {
        if (_downloader != null) _downloader.Dispose();
    }
}
```

## 詳細設定 (TextureInfo)

`DownloadImage` の第4引数に `TextureInfo` オブジェクトを渡すことで、読み込む画像の設定を詳細に指定できます。

```cs
TextureInfo info = new TextureInfo();
info.GenerateMipMaps = true;
info.FilterMode = FilterMode.Bilinear;
info.WrapModeU = TextureWrapMode.Repeat;
info.WrapModeV = TextureWrapMode.Repeat;
info.MaxTextureSize = 2048; // 最大2048まで

_downloader.DownloadImage(_imageUrl, _targetRenderer.material, (UdonBehaviour)GetComponent(typeof(UdonBehaviour)), info);
```

## 注意点と制限事項

### 1. メモリ管理 (重要)
`VRCImageDownloader` のインスタンスは、使用しなくなったら必ず `.Dispose()` を呼び出してください。これにより GPU メモリが解放されます。これを忘れると、長時間のセッションでプレイヤーの環境がクラッシュする原因となります。

### 2. レート制限
`String Loading` と同様、**5秒間に1回**のリクエスト制限があります。

### 3. 画像サイズの制限
読み込める画像の最大サイズは **2048x2048** です。

### 4. インスタンスの保持
`VRCImageDownloader` のインスタンスは、メンバ変数（クラスフィールド）として保持してください。ローカル変数として定義すると、ダウンロード完了前にガベージコレクションによって破棄され、テクスチャが正しく読み込まれない場合があります。
