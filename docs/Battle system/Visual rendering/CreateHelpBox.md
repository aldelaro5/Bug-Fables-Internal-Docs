# CreateHelpBox
This method creates the `helpbox` which presents the action command help text with id `helpboxid` by using [SetText](../../SetText/SetText.md) in [non dialogue mode](../../SetText/Dialogue%20mode.md#non-dialogue-mode).

```cs
private void CreateHelpBox(int id)
```

## Parameters

- `id`: The `helpboxid` to use. There is an overload without this parameter that will use the existing value. NOTE: if this is sent and MainManager.`mashcommandalt` is true (sequential keys is set in the settings):
    - 4 (`TappingKey`) is overriden to 8 (`RandomTappingKeysTimer`)
    - 10 (`RandomTappingBar`) is overriden to 8 (`RandomTappingKeysTimer`)
    - 11 (`RandomPressKeyTimer`) is overriden to 8 (`RandomTappingKeysTimer`)
    - 9 (`RandomPressBar`) is overriden to 12 (`MultiPressBar`)

## Procedure
The method does nothing if `helpboxid` is negative or `helpbox` exists already.

- `helpbox` is created as the DialogueAnim of a new 9box with position (-3.0, -4.25, 10.0), size of (12.0, 1.75) with type 0, a sortingOrder of -5 with grow and a color of pure white
- SetText is called in non dialogue mode using MainManager.`commandhelptext[helpboxid]` as the text with the following properties:
    - [fonttype](../../SetText/Notable%20states.md#fonttype) of 0 (`BubblegumSans`)
    - linebreak of 9.0
    - No tridimensional
    - position of (-5.5, 0.1, 0.0)
    - No cameraoffset
    - size of (0.75, 0.75)
    - parent of `helpbox`
    - No caller
