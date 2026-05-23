# uGUI in Udon

## Events

You can receive click/change events from UI components such as Button, Slider, and Toggle. However, in UdonSharp you cannot assign callbacks directly like regular Unity C# scripts. When a uGUI event needs to call an UdonSharp method, do not select the method on the script itself. Assign the UdonBehaviour component as the event target, then call `UdonBehaviour.SendCustomEvent(string)` with the UdonSharp event method name as the string argument.

Because of this, you cannot receive the Slider/Toggle value as an event argument. Even for `On Value Changed`, put the UdonSharp event name in the Static String argument of `SendCustomEvent`, then read the current value from the Slider or Toggle reference assigned in the Inspector.

### Inspector setup

For Button `On Click()` and Slider/Toggle `On Value Changed()`, configure the persistent call as follows.

1. Add a persistent call with `+`.
2. Assign the UdonBehaviour component on the target GameObject, not the UdonSharp script method directly.
3. Select `UdonBehaviour.SendCustomEvent` in the function dropdown.
4. Enter the public no-argument UdonSharp method name in the Static String argument. Examples: `OnClick`, `OnPetColorValueChanged`

### Reading values

For Slider and Toggle change events, do not read the value from the event argument. Store a reference to the UI component on the UdonSharp behaviour, then read the current value from that reference.

```cs
// Reference to the Slider
public Slider slider;

// Called from the Slider's On Value Changed via UdonBehaviour.SendCustomEvent("OnSliderChange")
public void OnSliderChange()
{
    var value = slider.value;
}
```

### Current scene example

In the current scene, `Assets/Scenes/VRCDefaultWorldScene.unity`, the UI under `PetAgentRoot/PetMenuPivot/PetMenu` follows this pattern. For example, the tab buttons and Toy/Food buttons target a `VRC.Udon.UdonBehaviour`, call `SendCustomEvent`, and pass `OnClick` as the Static String. `PetColorSlider` also targets the `PetMenu` UdonBehaviour, calls `SendCustomEvent`, and passes `OnPetColorValueChanged`; the value is then read through the Slider reference held by `PetMenu`.

### VRChat UI setup

For uGUI that players can interact with, use these settings by default.

- Set `Navigation` to `None` on Selectable UI elements such as Button, Slider, Toggle, Scrollbar, and InputField/TMP_InputField. In VRChat, movement input can also drive UI navigation, so Automatic or Explicit navigation can move the selected UI unexpectedly.
- Add `VRCUiShape` to any Canvas that players need to interact with. Without `VRCUiShape`, a visible Canvas is not treated as player-operable UI in VRChat.
- Use `World Space` as the default `Render Mode` for player-interactable Canvas objects. This gives the UI a clear world position, distance, and direction, and works well with VRChat UI raycasts.
- In VRChat, implement InputField/TMP_InputField handling with the assumption that only `On Value Changed` (`onValueChanged`) is invoked. Logic placed in submit/end-edit style events may not run in VRChat.
- Set `Scroll Sensitivity` to `0` on ScrollRect. Movement input can also feed scrolling, so a non-zero value can make the ScrollRect move while the player is trying to move.
