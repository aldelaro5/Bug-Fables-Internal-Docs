# [Kali](../../Enemy%20actions/Enemies/Kali.md) EventDialogue
`Kali` has an EventDialogue that is meant to trigger as their [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath). This is mostly a mini cutscene supplementing their death, but also to ensure that [Beetle](../../Enemy%20actions/Enemies/Beetle.md) also dies with them.

## Logic sequence

- [BreakIce](../../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#breakice) called on `Kali`
- `Kali`'s `isnumb` and `isasleep` are set to false
- `Kali` animstate set to 18 (`KO`)
- `Kali`'s `notalk` set to true
- Yield for a second
- [SetText](../../SetText/SetText.md) is called in [dialogue](../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
    - text: `commondialogue[160]`
    - [fonttype](../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
    - linebreak: `messagebreak`
    - tridimensional: false
    - position: Vector3.zero
    - size: Vector3.one
    - parent: `Kali`
    - caller: null
- Yield all frames until the [message](../../SetText/Notable%20states.md#message) lock is released
- Yield for 0.5 seconds
- If `Beetle` is still present (they could have already died):
    - Their `exp` is set to 0
    - Their animstate is set to 11 (`Hurt`)
    - Their `hp` is set to 0
- [EndBattleWon](../Terminal%20wrappers/EndBattleWon.md) is called with addexp and no skipids which will end the battle and switch to a [terminal flow](../Update%20flows/Terminal%20flow.md)
