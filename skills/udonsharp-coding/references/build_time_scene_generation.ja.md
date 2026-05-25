# UdonSharp のシーンビルド時自動生成

この資料は、Editor 拡張でビルド時に GameObject、`UdonSharpBehaviour`、または backing `UdonBehaviour` を自動生成する場合の注意点をまとめたものです。

## 基本方針

シーンビルド時に Udon 関連コンポーネントを生成する場合は、`IProcessSceneWithReport` だけに依存せず、負の order を指定した `[PostProcessScene]` を使う。

VRChat SDK はシーン処理中に `UdonBehaviour` を列挙し、private な `serializedProgramAsset` に `programSource.SerializedProgramAsset` を設定する。UdonSharp も `[PostProcessScene]` を使って、proxy の `UdonSharpBehaviour` から backing `UdonBehaviour` へ値をコピーし、プレイヤービルド時には proxy を取り除く。

このため、これらの処理より後に `UdonBehaviour` が生成されると、ビルド後のシーンには存在しているが実行プログラムを読めないコンポーネントになることがある。

## 推奨する差し込み位置

UdonSharp の通常のシーンビルド処理より前に実行する。

```cs
using UnityEditor;
using UnityEditor.Callbacks;
using UnityEngine.SceneManagement;

public static class MyUdonSceneBuildGenerator
{
    /// <summary>
    /// UdonSharpがproxyをUdonBehaviourへ反映する前に、ビルド用オブジェクトを生成する。
    /// </summary>
    [PostProcessScene(-1000)]
    private static void OnPostProcessScene()
    {
        if (!BuildPipeline.isBuildingPlayer && !EditorApplication.isPlayingOrWillChangePlaymode)
        {
            return;
        }

        ProcessScene(SceneManager.GetActiveScene());
    }
}
```

ClientSim で自動生成オブジェクトが必要な場合は、Play Mode 進入時の処理も許可する。`BuildPipeline.isBuildingPlayer` だけを条件にすると、ClientSim の経路では生成処理が走らない。

一方で、通常の編集状態で意図せずシーンを汚さないように、ビルド中または Play Mode 進入中以外は return する。

## `serializedProgramAsset` の補完

このパスで `UdonBehaviour` を生成または変更した後は、`SerializedObject` 経由で private な `serializedProgramAsset` を明示的に補完する。これにより、VRChat SDK 側のコールバック順序に依存せず、生成した `UdonBehaviour` が実行プログラムを取得できる。

```cs
using UnityEditor;
using VRC.Udon;

private static void EnsureSerializedProgramAsset(UdonBehaviour behaviour)
{
    if (behaviour == null || behaviour.programSource == null)
    {
        return;
    }

    var serializedBehaviour = new SerializedObject(behaviour);
    var serializedProgramAsset = serializedBehaviour.FindProperty("serializedProgramAsset");

    if (serializedProgramAsset == null ||
        serializedProgramAsset.objectReferenceValue == behaviour.programSource.SerializedProgramAsset)
    {
        return;
    }

    serializedProgramAsset.objectReferenceValue = behaviour.programSource.SerializedProgramAsset;
    serializedBehaviour.ApplyModifiedPropertiesWithoutUndo();
}
```

通常は、生成した root オブジェクト配下に対して `GetComponentsInChildren<UdonBehaviour>(true)` を実行し、見つかったすべての `UdonBehaviour` にこの補完を適用する。

## 注意点

- シーンビルド時にだけ必要なオブジェクトは、通常の編集状態で永続化しないようにする。
- UdonSharp スクリプトを含む prefab から生成する場合は、UdonSharp の proxy 反映処理より前に生成する。
- コメントやドキュメントには「修正履歴」を残さない。コメントには、UdonSharp より前に動かす理由、ClientSim の Play Mode 経路を通す理由、VRChat SDK のコールバック順序に依存しないための意図を書く。
