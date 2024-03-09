# UpdateEntities
This method goes through all actors in `alldata` and does the following on each in order:

## Sprite color update
If battleentity.`cotunknown` is true, battleentity.`sprite` color is set to pure black if the `sprite` exists and if battleentity.`refreshedcotu` is false (RefreshCOT wasn't called yet), RefreshCOT is called on the battleentity

Otherwise, an opacity number for the color is determined. It's 1.0 (fully opaque) unless battleentity.`hologram` is true or all of the following are true where the factor will be 0.55 instead (55% opaque):

- The actor is a player party member
- `playertargetID` is above -1 and it isn't the actor (meaning this actor is not targetted by the current enemy party's action)
- We aren't in a [terminal flow](../Battle%20flow/Update%20flows/Terminal%20flow.md) or [EventDialogue](../Battle%20flow/EventDialogue.md)
- The [message](../../SetText/Notable%20states.md#message) lock is released

From there, a bunch of mutually exclusive checks are performed in order and the first one that gets fufilled will have the battleentity.`sprite` color changed to a lerp from the existing one to some other colors with the opactity being the one determined above. The factor of the lerp varies depending on the check applied.

### Player party member not being targetted by the enemy
This case will lerp to 0x595959 (35% of every color which is light grey) with a factor of 0.07 of the frametime. The conditions for that to happen is all of the following being true:

- The actor's `cantmove` is above 0 (it does not have any actions available)
- We are in the [enemy phase](../Battle%20flow/Main%20turn%20life%20cycle.md#enemy-phase)
- The actor is a player party member
- We aren't in a [terminal flow](../Battle%20flow/Update%20flows/Terminal%20flow.md) or [EventDialogue](../Battle%20flow/EventDialogue.md)
- The [message](../../SetText/Notable%20states.md#message) lock is released

### Charged actor
This case will set the color to RainbowColor using variant 0, colorspeed of 5.9 * (actor's `charge` / 3.0), min of 0.5, limit of 1.0 and alpha being the opacity factor determined earlier. 

This is only done if the actor's `charge` is above 0.

### [Poison](../Actors%20states/BattleCondition/Poison.md) Condition
This case will lerp to 0xD800D8 (bright magenta) with a factor of 1/5 of the frametime. The conditions for that to happen is the actor having the `Poison` condition.

### [Fire](../Actors%20states/BattleCondition/Fire.md) Condition
This case will lerp to 0xBF5919 (bright orange) with a factor of 1/5 of the frametime. The conditions for that to happen is the actor having the `Fire` condition.

### [Inked](../Actors%20states/BattleCondition/Inked.md) Condition
This case will lerp to 0x7200D8 (bright purple) with a factor of 1/5 of the frametime. The conditions for that to happen is the actor having the `Inked` condition.

### Default case
This one happens if none of the above applied.

The lerp is done to `spritebasecolor` with a factor of framestep * 0.2.

## battleentity.`firepart` update
This section updates the state of the `firepart` object which is some particles that plays as part of a [condition](../Actors%20states/Conditions.md).

There are only 3 conditions where the `firepart` is involved alongside their corresponding particles prefab:

- [Fire](../Actors%20states/BattleCondition/Fire.md): `Prefabs/Particles/Flame`
- [Inked](../Actors%20states/BattleCondition/Inked.md): `Prefabs/Particles/InkDrip`
- [Sticky](../Actors%20states/BattleCondition/Sticky.md): `Prefabs/Particles/StickyDrip`

If the actor doesn't have any of these conditions, `firepart` is destroyed here if it still exists.

If it does, the corresponding prefab is instantiated and set to `firepart`. It also gets some initialisation:

- Childed to the `sprite`
- The gameObject tag is set to `DelAftBtl` (detroyed on [ReturnToOverworld](../Battle%20flow/Terminal%20coroutines/ReturnToOverworld.md))
- Local position set to Vector3.up
