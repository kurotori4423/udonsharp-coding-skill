# UdonでのuGUI

## イベント

Button, Slider, Toggleなど、クリックや変化のイベントを取得できますが、UdonSharpでは通常のUnity C#のように直接コールバックを指定できません。uGUIのイベントからUdonSharpのメソッドを呼ぶ場合は、対象のスクリプト本体に対してメソッドを選ぶのではなく、UdonBehaviourコンポーネントの`SendCustomEvent(string)`を呼び出し、文字列引数にUdonSharp側のイベントメソッド名を指定します。

この関係で、SliderやToggleの値をイベント引数として受け取る形にはできません。`On Value Changed`でも`SendCustomEvent`のStatic String引数にはイベント名を入れ、値はインスペクタにセットしたSliderやToggleの参照から現在値を読み取ります。

### インスペクタでの設定

Buttonの`On Click()`やSlider/Toggleの`On Value Changed()`には、以下のように設定します。

1. `+`でPersistent Callを追加する。
2. Object欄には、UdonSharpスクリプトが付いたGameObjectではなく、そのGameObject上のUdonBehaviourコンポーネントを割り当てる。
3. 関数欄では、UdonSharpスクリプトのメソッドを直接選ばず、`UdonBehaviour.SendCustomEvent`を選ぶ。
4. Static String引数に、呼び出したいUdonSharp側のpublicな引数なしメソッド名を入力する。例: `OnClick`, `OnPetColorValueChanged`

### 値の読み取り

SliderやToggleの変更イベントでは、イベントから値を受け取るのではなく、UdonSharp側に保持したUI参照から現在値を読み取ります。

```cs
// Sliderの参照
public Slider slider;

// SliderのOn Value ChangedからUdonBehaviour.SendCustomEvent("OnSliderChange")で呼び出す
public void OnSliderChange()
{
    var value = slider.value;
}
```

### 現在シーンの設定例

現在のシーン`Assets/Scenes/VRCDefaultWorldScene.unity`では、`PetAgentRoot/PetMenuPivot/PetMenu`配下のUIがこの形で設定されています。例えば、各タブボタンやToy/Foodボタンは対象が`VRC.Udon.UdonBehaviour`、メソッドが`SendCustomEvent`、Static Stringが`OnClick`です。`PetColorSlider`も同様に、対象が`PetMenu`のUdonBehaviour、メソッドが`SendCustomEvent`、Static Stringが`OnPetColorValueChanged`になっており、値は`PetMenu`側が保持しているSlider参照から読む構成です。

### VRChat向けのUI設定

プレイヤーが操作するuGUIは、以下の設定を基本にします。

- Button、Slider、Toggle、Scrollbar、InputField/TMP_InputFieldなどのSelectable系UI要素は`Navigation`を`None`にする。VRChatでは移動入力などがUIナビゲーションにも伝わるため、AutomaticやExplicitのままだと意図せず選択中のUIが動くことがあります。
- プレイヤーが触るCanvasには`VRCUiShape`を付ける。`VRCUiShape`がないCanvasは、見えていてもVRChat上でプレイヤーが操作可能なUIとして扱われません。
- プレイヤー操作用Canvasの`Render Mode`は、基本的に`World Space`にする。ワールド内の位置・距離・向きが明確になり、VRChatのUIレイキャスト対象として扱いやすくなります。
- VRChatではInputField/TMP_InputFieldのイベントは`On Value Changed`（`onValueChanged`）だけが呼ばれる前提で実装する。入力確定や終了系のイベントに処理を置くと、VRChat上で実行されないことがあります。
- ScrollRectは`Scroll Sensitivity`を`0`にする。移動入力などがスクロールにも伝わるため、値が残っているとプレイヤーの移動操作に合わせてスクロールが動くことがあります。
