# Build-Time Scene Generation with UdonSharp

Use this reference when editor code generates GameObjects, `UdonSharpBehaviour`, or backing `UdonBehaviour` components during scene build processing.

## Core Rule

Prefer an editor-only `[PostProcessScene]` callback with a negative order over relying only on `IProcessSceneWithReport`.

VRChat's Udon build preprocessor populates each `UdonBehaviour`'s private serialized program asset during scene processing. UdonSharp also uses `[PostProcessScene]` to copy proxy data into backing `UdonBehaviour` components and strip proxies for player builds. If generated `UdonBehaviour` components appear after those passes, they can be present in the built scene without a runnable program.

## Recommended Hook

Run before UdonSharp's default scene build pass:

```cs
using UnityEditor;
using UnityEditor.Callbacks;
using UnityEngine.SceneManagement;

public static class MyUdonSceneBuildGenerator
{
    /// <summary>
    /// Creates build-time scene objects before UdonSharp copies proxy data to UdonBehaviour.
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

Allow Play Mode entry as well as player builds when generated objects are needed by ClientSim. Do not gate only on `BuildPipeline.isBuildingPlayer`, because ClientSim depends on Unity's play-mode scene post-processing path.

Use the guard above to avoid running in ordinary edit-time scene processing paths while still supporting player builds and ClientSim.

## Populate `serializedProgramAsset`

After creating or modifying `UdonBehaviour` components in this pass, explicitly populate their private `serializedProgramAsset` through `SerializedObject`. This prevents the generated component from depending on VRChat SDK callback ordering.

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

Run this for every generated or build-time-modified `UdonBehaviour`, usually by walking the generated root object with `GetComponentsInChildren<UdonBehaviour>(true)`.

## Notes

- Keep generated scene objects temporary and created only in scene post-processing unless they intentionally need to persist as assets.
- If the generated objects come from prefabs containing UdonSharp scripts, create them before UdonSharp's proxy-to-Udon pass.
- Do not use comments or docs to record that this pattern was added as a fix. Comments should describe implementation intent: ordering before UdonSharp, ClientSim support, and independence from VRChat SDK callback ordering.
