# UseCharm
This is a coroutine that is invoked at specific moments of [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md) and [AddExperience](../Battle%20flow/Terminal%20coroutines/AddExperience.md). It will, when applicable, process a charm if the player had some left with the amount remaining being tracked by [flagvar](../../Flags%20arrays/flagvar.md) 22. 

The coroutine is expected to be stored in `checkingdead` as it will be set to null at the very end so the caller can yield on it. As for who gets targetted by the charm, it's either the party or the `currentturn` player party member depending on the `Charm`, an enum that identifies a charm type.

```cs
private IEnumerator UseCharm(Charm type)
```

## Parameters

- `type`: The type of the charm to attempt to apply

## Procedure
In order for this coroutine to process the charm, all of the following must true when it is invoked (otherwise, nothing happens except `checkingdead` being set to null):

- [flagvar](../../Flags%20arrays/flagvar.md) 22 is above 0 (the player has at least 1 charm left)
- `charmcooldown` is 0
- A 6% random test must pass (generate a random integer number between 0 and 100 exclusive, that number converted to float must be \<= 5.5 which is a pass from 0 to 5 inclusive which makes it 6%)
- At least 1 player party member's `hp` is above 0 and isn't `eatenby`
- EnemyActingOutOfOrder returns false meaning no enemy party member's [hitaction](../BattleControl.md#hitactions) is true
- Either the type is `ExpUp` (meaning it was invoked from [AddExperience](../Battle%20flow/Terminal%20coroutines/AddExperience.md)) or it's any other type while `cancelupdate` is false (we aren't in a [terminal flow](Update%20flows/Terminal%20flow.md))
- If the type is `AttackUp`, `playerdata[currentturn]` must not have the `AttackUp` condition already

The following sections describes what happens when the charm is cleared to be processed.

### Charm animation
This section will be paraphrased as it contains verbose animation logic:

- The `Magic` sound is played
- A new nameless GameObject is created (locally refered to as dancer) that will have a SpriteRenderer initially using the `charmdance[0]` sprite (which is sprite 98 of `Sprites/Entities/moth0`) on layer 15 (`3DUI`) fully opaque with a SpinAround and a position of (0.0, 0.0, -0.5)
- All the intensity of Light GameObjects are saved locally
- Over the course of 40.0 frames (tracked with a local frame counter):
    - The dancer's alpha will be lerped from 0.0 to 0.6 with the factor being the time progression over the 40.0 frames
    - The SpinAround on the dancer has its `itself`'s Y component set to a lerp from 0.0 to 15.0 with a factor of the time progression over the 40.0 frames (the X and Z components are always set to 0.0)
    - All Light GameObject's intensity will be lerped from their initial value saved locally to half of it with a factor of the time progression over the 40.0 frames
- 0.5 seconds are yielded
- The dancer's sprite is set to `charmdance[1]` (which is sprite 99 of `Sprites/Entities/moth0`)
- The SpinAround of the dancer is destroyed
- The angles of the dancer are set to (0.0, 180.0, 0.0)

### Charm process
The logic here depends on the type:

#### `AttackUp`

- [StatEffect](../Visual%20rendering/StatEffect.md) is called on `playerdata[currentturn]` with type 0 (red up arrow)
- [SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called on `playerdata[currentturn]` giving the `AttackUp` condition for 1 actor turn
- The `StatUp` sound is played

#### `DefenseUp`

- All player party members whose `hp` is above 0 and aren't `eatenby` have the following happen to them:
    - [StatEffect](../Visual%20rendering/StatEffect.md) is called on the player party member with type 1 (blue up arrow)
    - [SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called on the player party member giving the `DefenseUp` condition for 1 actor turn
- The `StatUp` sound is played

#### `ExpUp`

- `expreward` is multiplied by 1.15 floored then clamped from 0 to instance.`neededexp`
- [RefreshEXP](../Visual%20rendering/RefreshEXP.md) is called

#### `HealHP`

- All player party members whose `hp` is above 0 and aren't `eatenby` have the following happen to them:
    - The amount of HP to heal is determined which is the player party member's `maxhp` * 0.2 floored clamped from 1 to the `maxhp`
    - The player party member's `hp` is increased by the amount to heal then clamped from 0 to its `maxhp`
    - [ShowDamageCounter](../Visual%20rendering/ShowDamageCounter.md) is called with type 1 with the amount of HP healed as the ammount starting at the player party member's battleentity position + Vector3.up to (2.0, 2.0, 2.0)
- The `StatUp` sound is played

#### `HealTP`
This type expects [flagvar](../../Flags%20arrays/flagvar.md) 1 to be set to the amount of TP to heal. This is the responsibility of the caller.

- [ShowDamageCounter](../Visual%20rendering/ShowDamageCounter.md) is called with type 2 with [flagvar](../../Flags%20arrays/flagvar.md) 1 as the ammount starting at the player party member's battleentity position + Vector3.up to (2.0, 2.0, 2.0)
- instance.`tp` is increaded by [flagvar](../../Flags%20arrays/flagvar.md) 1 then clamped from 0 to instance.`maxtp`
- The `Heal2` sound is played

### End animation
The following is paraphrased as it contains verbose animation logic:

- Over the course of 40.0 frames (tracked with a local frame counter):
    - The dancer's alpha will be lerped from 0.6 to 0.0 with the factor being the time progression over the 40.0 frames
    - All Light GameObject's intensity will be lerped from half of their their initial value saved locally to their original value before this UseCharm call with a factor of the time progression over the 40.0 frames
- All Light GameObject's intensity are restored to their original value before this UseCharm call
- The dancer is destroyed

### Last steps

- [flagvar](../../Flags%20arrays/flagvar.md) 22 is decremented (this consumes the charm)
- `charmcooldown` is set to a random number from 3 to 7 inclusive (this means a minimum amount of main turn needs to advance before the next charm is able to be processed even if it can be and the amount is between 3 and 7)
- If [flagvar](../../Flags%20arrays/flagvar.md) 22 reached 0 (meaning the last charm was just consumed):
    - A [SetText](../../SetText/SetText.md) call occurs in [dialogue move](../../SetText/Dialogue%20mode.md#dialogue-mode) using the text `|boxstyle,4||center||halfline||spd,0|` followed by `menutext[184]` (a message informing the player they no longer have any charms). The call also has these properties:
      - [fonttype](../../SetText/Notable%20states.md#fonttype) of 0 (`BubblegumSans`)
      - linebreak of `messagebreak`
      - No tridimensional
      - position of Vector3.zero
      - No cameraoffset
      - size of Vector3.one
      - No parent
      - No caller
    - All frames are yielded while the [message](../../SetText/Notable%20states.md#message) lock is grabbed
- `checkingdead` is set to null which informs the caller the coroutine is complete
