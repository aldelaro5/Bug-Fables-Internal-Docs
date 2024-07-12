# `MenderBot`

## Move selection
1 moves is possible:

1. Calls [SetText](../../../SetText/SetText.md) in dialogue mode with a random `commondialogue` string

Move 1 is always used

## Move 1 - Calls [SetText](../../../SetText/SetText.md) in dialogue mode
Calls [SetText](../../../SetText/SetText.md) in dialogue mode with a random `commondialogue` string. No damages are dealt.

### Logic sequence

- animstate set to 11 (`Hurt`)
- Yield for a frame
- [SetText](../../SetText/SetText.md) is called in [dialogue](../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
    - text: `|`[boxstyle](../../../SetText/Individual%20commands/Boxstyle.md)`,1||`[shaky](../../../SetText/Individual%20commands/Shaky.md)`||`[glitchy](../../../SetText/Individual%20commands/Glitchy.md)`|` followed by a `commondialogue` string whose index is random between 125 and 129 inclusive
    - [fonttype](../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
    - linebreak: `messagebreak`
    - tridimensional: false
    - position: Vector3.zero
    - size: Vector3.one
    - parent: This enemy
    - caller: null
- Yield all frames until the [message](../../../SetText/Notable%20states.md#message) lock releases
