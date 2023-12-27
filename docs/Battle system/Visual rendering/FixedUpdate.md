# FixedUpdate
The FixedUpdate of BattleControl contains logic relating to updating the `choicevine`, `cursor` and the color of the actor's sprites visually. It also updates `turncooldown`'s decreases. It does nothing however in a [terminal flow](../Battle%20flow/Update.md#terminal-flow).

## `choicevine` updates
This section only applies if `choicevine` exists which is after [StartBattle](../StartBattle.md) called CreateVine.

- If `currentaction` is `BaseAction` (the main vine action menu):
    - All children of `basevine` (which are all the actual vines object) has their scale set to a lerp from the existing one to (2.5, 2.5, 2.5) with a factor of 0.15
    - UpdateRotation is called which updates the y angle of the `choicevine` given the current `option`. The new y angle is a LerpAngle from the existing one to -(360 / `vineoption` * `option`) - 24.0 * (1.0 - the horizontal viewport position of the `choicevine`)
- Otherwise:
    - UpdateRotation is called which updates the y angle of the `choicevine` given the current `lastoption`. The new y angle is a LerpAngle from the existing one to -(360 / `vineoption` * `lastoption`) - 24.0 * (1.0 - the horizontal viewport position of the `choicevine`)
    - All children of `basevine` (which are all the actual vines object) has their scale set to a lerp from the existing one to (2.5, 1.5, 2.5) with a factor of 0.15 unless the vine index matches `lastoption` in which case, it's the same, but the y scale is 2.5 (2.0 if `currentaction` is `SelectPlayer`)
- If we are in an [uncontrolled flow](../Battle%20flow/Update.md#uncontrolled-flow), the `choicevine` position is set to offscreen at (0.0, 999.0, 0.0)

## `cursor` updates
This section only applies if `cursor` exists which is after [StartBattle](../StartBattle.md) called CreateCursor.

The `cursor` is only rendered on screen if we are in a [controlled flow](../Battle%20flow/Update.md#controlled-flow) while `currantaction` isn't `BaseAction` (the main vine action menu) and `itemarea` is `SingleAlly` or `SingleEnemy`. Otherwise, its position is set to offscreen at (0.0, 100.0, 0.0).

When it needs to be rendered, the `cursor` position setting logic depends on `currentaction`.

### `SelectEnemy`
The position is set to a lerp from the existing one to the actor's battleentity position + an offset vector with a factor of 0.2 unless the `cursor` y position is above 20.0 where the factor is 1.0 (instant) instead. The actor is `avaliabletargets[option]` if `currentchoice` is among the following (it's `enemydata[option]` otherwise):

- `Attack`
- `Skill`
- `Strategy`
- `Item`

As for the offset vector, it's Vector3.up if the actor's [position](../Actors%20states/BattlePosition.md) is `Underground`. If it's not, it's the actor's `cursoroffset` + Vector3.up * the actor's battleentity.`height`.

### `SelectPlayer`
The position is set to a lerp from the existing one to `playerdata[option].battleentity` position + `playerdata[option].cursoroffset` with a factor of 0.2 unless the `cursor` y position is above 20.0 where the factor is 1.0 (instant) instead

## `turncooldown`
If the field is above 0.0, it is decremented by 1.0. Effectively, this means that any [SetItem](../Player%20UI/SetItem.md) or [CancelList](../Player%20UI/CancelList.md) will prevent [GetChoiceInput](../Player%20UI/GetChoiceInput.md) calls for exactly 5 FixedUpdate cycle.

## Actor's visual updates
This section only happens if `halfload` is true which means most of [StartBattle](../StartBattle.md) is done as it guarantees all actors are fully loaded.

It calles UpdateEntities which goes through all actors in `alldata` and does the following on each in order:

### Sprite color update
If battleentity.`cotunknown` is true, battleentity.`sprite` color is set to pure black if the `sprite` exists and if battleentity.`refreshedcotu` is false (RefreshCOT wasn't called yet), RefreshCOT is called on the battleentity

Otherwise, an opacity number for the color is determined. It's 1.0 (fully opaque) unless battleentity.`hologram` is true or all of the following are true where the factor will be 0.55 instead (55% opaque):

- The actor is a player party member
- `playertargetID` is above -1 and it isn't the actor (meaning this actor is not targetted by the current enemy party's action)
- We aren't in a [terminal flow](../Battle%20flow/Update.md#terminal-flow) or [EventDialogue](../Battle%20flow/EventDialogue.md)
- The [message](../../SetText/Notable%20states.md#message) lock is released

From there, a bunch of mutually exclusive checks are performed in order and the first one that gets fufilled will have the battleentity.`sprite` color changed to a lerp from the existing one to some other colors with the opactity being the one determined above. The factor of the lerp varies depending on the check applied.

#### Player party member not being targetted by the enemy
This case will lerp to 0x595959 (35% of every color which is light grey) with a factor of 0.07 of the frametime. The conditions for that to happen is all of the following being true:

- The actor's `cantmove` is above 0 (it does not have any actions available)
- We are in the [enemy phase](../Battle%20flow/Update.md#enemies-phase)
- The actor is a player party member
- We aren't in a [terminal flow](../Battle%20flow/Update.md#terminal-flow) or [EventDialogue](../Battle%20flow/EventDialogue.md)
- The [message](../../SetText/Notable%20states.md#message) lock is released

#### Charged actor
This case will set the color to RainbowColor using variant 0, colorspeed of 5.9 * (actor's `charge` / 3.0), min of 0.5, limit of 1.0 and alpha being the opacity factor determined earlier. 

This is only done if the actor's `charge` is above 0.

#### `Poison` [Condition](../Actors%20states/Conditions.md)
This case will lerp to 0xD800D8 (bright magenta) with a factor of 1/5 of the frametime. The conditions for that to happen is the actor having the `Poison` condition.

#### `Fire` [Condition](../Actors%20states/Conditions.md)
This case will lerp to 0xBF5919 (bright orange) with a factor of 1/5 of the frametime. The conditions for that to happen is the actor having the `Fire` condition.

#### `Inked` [Condition](../Actors%20states/Conditions.md)
This case will lerp to 0x7200D8 (bright purple) with a factor of 1/5 of the frametime. The conditions for that to happen is the actor having the `Inked` condition.

#### Default case
This one happens if none of the above applied.

The lerp is done to `spritebasecolor` with a factor of framestep * 0.2.

### battleentity.`firepart` update
This section updates the state of the `firepart` object which is some particles that plays as part of a [condition](../Actors%20states/Conditions.md).

There are only 3 conditions where the `firepart` is involved alongside their corresponding particles prefab:

- `Fire`: `Prefabs/Particles/Flame`
- `Inked`: `Prefabs/Particles/InkDrip`
- `Sticky`: `Prefabs/Particles/StickyDrip`

If the actor doesn't have any of these conditions, `firepart` is destroyed here if it still exists.

If it does, the corresponding prefab is instantiated and set to `firepart`. It also gets some initialisation:

- Childed to the `sprite`
- The gameObject tag is set to `DelAftBtl`
- Local position set to Vector3.up
