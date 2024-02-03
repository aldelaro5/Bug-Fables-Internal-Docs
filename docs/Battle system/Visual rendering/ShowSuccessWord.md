# ShowSuccessWord
This coroutine allows to visually display a battle message which are words informing the success of a block or action command. The progression of the coroutine is meant to be tracked by `wordroutine` as it is set to null after this coroutine is over.

```cs
private IEnumerator ShowSuccessWord(EntityControl t, bool block, bool super)
```

## Parameters

- `t`: The entity the word should be rendered close to
- `block`: Whether or not this word relates to blocking (takes priority over everything else)
- `super`: Whether or not this word relates to using an attack targetting a weakness (ignored if `block` is true or the `LuckierDay` [medal](../../Enums%20and%20IDs/Medal.md) is equipped)

## Procedure

- `commandword` is set to a new UI object named `word` childed to the `battlemap` with a local position of t's position + (1.0, 3.5, 0.0) + Vector3.up + (0.0, 0.0, 10.0) with a scale of Vector3.zero, a material renderQueue of 50000, a SpriteRenderer and a DialogueAnim
- The sprite of the `word` is set depending on different conditions (only the first one that applies in order is used):
    - block is true: MainManager.`battlemessage[4]` (BLOCK) (overriden to MainManager.`battlemessage[3]` (UNBEELIEVABLE) if the `LuckierDay` [medal](../../Enums%20and%20IDs/Medal.md) is equipped). This also sets `commandword`'s local position to t's position + (2.0, 2.0, 0.0) + (0.0, 0.0, 10.0)
    - The `LuckierDay` [medal](../../Enums%20and%20IDs/Medal.md) is equipped: MainManager.`battlemessage[3]` (UNBEELIEVABLE)
    - super is true: MainManager.`battlemessage[5]` (SUPER)
    - None of the above applied: MainManager.`battlemessage` whose index is `combo` (clamped from 0 to 3). 0 is OKAY, 1 is GREAT, 2 is AWESOME and 3 is UNBEELIEVABLE
- A second is yielded
- The targetscale of the `word`'s DialogueAnim is set to (1.0, 0.0, 1.0)
- The shrink of the `word`'s DialogueAnim is set to true (this means the word will collapse to nothing vertically)
- `commandword` is destroyed in 1 second
- `wordroutine` is set to null signaling it is no longer in progress
