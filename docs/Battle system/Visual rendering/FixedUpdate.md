# FixedUpdate
The FixedUpdate of BattleControl contains logic relating to updating the `choicevine`, `cursor` and the color of the actor's sprites visually. It also updates `turncooldown`'s decreases. It does nothing however in a [terminal flow](../Battle%20flow/Update%20flows/Terminal%20flow.md).

## `choicevine` updates
This section only applies if `choicevine` exists which is after [StartBattle](../StartBattle.md) called CreateVine.

- If [currentaction](../Player%20UI/Pick.md) is `BaseAction` (the main vine action menu):
    - All children of `basevine` (which are all the actual vines object) has their scale set to a lerp from the existing one to (2.5, 2.5, 2.5) with a factor of 0.15
    - UpdateRotation is called which updates the y angle of the `choicevine` given the current `option`. The new y angle is a LerpAngle from the existing one to -(360 / `vineoption` * `option`) - 24.0 * (1.0 - the horizontal viewport position of the `choicevine`)
- Otherwise:
    - UpdateRotation is called which updates the y angle of the `choicevine` given the current `lastoption`. The new y angle is a LerpAngle from the existing one to -(360 / `vineoption` * `lastoption`) - 24.0 * (1.0 - the horizontal viewport position of the `choicevine`)
    - All children of `basevine` (which are all the actual vines object) has their scale set to a lerp from the existing one to (2.5, 1.5, 2.5) with a factor of 0.15 unless the vine index matches `lastoption` in which case, it's the same, but the y scale is 2.5 (2.0 if [currentaction](../Player%20UI/Pick.md) is `SelectPlayer`)
- If we are in an [uncontrolled flow](../Battle%20flow/Update%20flows/Uncontrolled%20flow.md), the `choicevine` position is set to offscreen at (0.0, 999.0, 0.0)

## `cursor` updates
This section only applies if `cursor` exists which is after [StartBattle](../StartBattle.md) called CreateCursor.

The `cursor` is only rendered on screen if we are in a [controlled flow](../Battle%20flow/Update%20flows/Controlled%20flow.md) while [currentaction](../Player%20UI/Pick.md) isn't `BaseAction` (the main vine action menu) and [itemarea](../AttackArea.md) is `SingleAlly` or `SingleEnemy`. Otherwise, its position is set to offscreen at (0.0, 100.0, 0.0)
When it needs to be rendered, the `cursor` position setting logic depends on [currentaction](../Player%20UI/Pick.md).

### `SelectEnemy`
The position is set to a lerp from the existing one to the actor's battleentity position + an offset vector with a factor of 0.2 unless the `cursor` y position is above 20.0 where the factor is 1.0 (instant) instead. The actor is `avaliabletargets[option]` if [currentchoice](../Player%20UI/Actions.md) is among the following (it's `enemydata[option]` otherwise):

- `Attack`
- `Skill`
- `Strategy`
- `Item`

As for the offset vector, it's Vector3.up if the actor's [position](../Actors%20states/BattlePosition.md) is `Underground`. If it's not, it's the actor's `cursoroffset` + Vector3.up * the actor's battleentity.`height`.

### `SelectPlayer`
The position is set to a lerp from the existing one to `playerdata[option].battleentity` position + `playerdata[option].cursoroffset` with a factor of 0.2 unless the `cursor` y position is above 20.0 where the factor is 1.0 (instant) instead

## `turncooldown`
If the field is above 0.0, it is decremented by 1.0.

## Actor's visual updates
[UpdateEntities](UpdateEntities.md) is called only if `halfload` is true which means most of [StartBattle](../StartBattle.md) is done as it guarantees all actors are fully loaded.
