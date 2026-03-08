# String Loading (VRCStringDownloader)

`VRCStringDownloader`を使用すると、実行時に外部のサーバー（GitHub Pages, Pastebin等）からテキストデータを取得できます。これにより、外部のJSONデータに基づいた情報の更新などが可能になります。

## 実装方法

`VRC.SDK3.StringLoading`ネームスペースを使用し、`VRCStringDownloader.LoadUrl`メソッドでダウンロードを開始します。結果は、`OnStringLoadSuccess`または`OnStringLoadError`コールバックで受け取ります。

### 基本的な実装例

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
        // ダウンロードの開始
        // 第2引数には、コールバックを受け取るUdonBehaviourを指定します
        VRCStringDownloader.LoadUrl(_url, (UdonBehaviour)GetComponent(typeof(UdonBehaviour)));
    }

    // 成功時の処理
    public override void OnStringLoadSuccess(IVRCStringDownload result)
    {
        string downloadedText = result.Result;
        Debug.Log($"Downloaded: {downloadedText}");
    }

    // 失敗時の処理
    public override void OnStringLoadError(IVRCStringDownload result)
    {
        Debug.LogError($"Failed to load string: {result.Error}");
    }
}
```

## 注意点と制限事項

### 1. 許可されたドメイン
ユーザーの「Allow Untrusted URLs」設定が無効な場合、VRChatが公式に許可しているドメイン（GitHub, Discord, Imgur等）からのみ取得可能です。

### 2. レート制限
ダウンロードのリクエストは、**5秒間に1回**までという制限があります。これを超える頻度で呼び出した場合、リクエストはキューに蓄積されます（最大1000件）。

### 3. VRCUrl の扱い
セキュリティ上の理由から、実行時に `string` から `VRCUrl` を生成することはできません。URLはインスペクターで設定するか、`VRCUrlInputField` を使用してユーザーに入力してもらう必要があります。

### 4. コールバックのターゲット
`LoadUrl` の第2引数に `this` を渡す場合、UdonSharpBehaviour を明示的に `(UdonBehaviour)` にキャストする必要があります。
