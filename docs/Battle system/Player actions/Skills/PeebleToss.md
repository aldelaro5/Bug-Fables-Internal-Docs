# `PeebleToss`

## About this action's usage
This action is special because it shares a lot of logic with lots of [items](../../../Enums%20and%20IDs/Items.md) usage in the game since it doubles as a generic item toss action.

This page documents the skills, but for clarity when documenting the items, the sections that are specific to this skills will be pointed out as the rest of the logic is either the same or extremely similar for the items which will be documented separately, but refer to this page for convenience.

The one notable change every items usage has is the sprite determination results in the corresponding sprite of the item that was used instead of a rock or Tanjerin/Cerise. The main changes specific items usage has is the object movement section may change in logic and the DoDamage call the skill does is replaced with the item's own logic.

## [DoDamage](../../Damage%20pipeline/DoDamage.md) calls (skill specific)

|#|Conditions|attacker|target|damageammount|property|overrides|block|
|-:|---|---|---|---|---|---|---|
|1|None|null|`availabletargets[target]`|1|[NoExceptions](../../Damage%20pipeline/AttackProperty.md)|null|false|

## No charges usage (items and skill)
This action sets `dontusecharge` to true and as such, it will not consume any attacker's `charge` as it would normally.

## Logic seqeunce

### Setup (items and skill)

- `dontusecharge` set to true
- Yield for 0.1 seconds
- attacker animstate set to 28 (`TossItem`)
- Yield for 0.1 seconds

### Rock Sprite determination (skill specific)

- The sprite of the object thrown is determined and it's `itemssprites[1, 13]` normally (`MightyPeeble`'s [medal](../../../Enums%20and%20IDs/Medal.md) sprite), but if [flags](../../../Flags%20arrays/flags.md) 275 is true (completed the Huuuuuuuuuu…!!! quest) and a 1/128 RNG test passes, it's overriden to a `Sprites/Entities/moth0` sprite with a uniform scale of 0.9x and with flipX. In such case, the index in the sprite array is determined by the following:
    - If [flags](../../../Flags%20arrays/flags.md) 555 is false (not yet reached postgame), it's index 93 (Tanjerin as they lift something with a patched horn)
    - Otherwise (reached the postgame), if [flags](../../../Flags%20arrays/flags.md) 682 is false (haven't yet talked to Cerise for the first time at Bandit Hideout), it's index 92 (Tanjerin as they lift something with a normal horn)
    - Otherwise (postgame after talking to Cerise at the Bandit Hideout), the index is randomly chosen between 92 (Tanjerin as they lift something with a normal horn) and 91 (Cerise gasping)

### Object movement (items and skill)

- Over the course of 40.0 frames, the object moves from the attacker + its `cursoroffset` - 1.0 in y to the `target` enemy party member + its `cursoroffset` + its `height` - 1.0 in y via a BeizierCurve3 with a ymax of 5.0 + the `target` enemy party member `height`. During the move, the z angle changes to (0.0 - framestep) * 20.0

### Damage logic (skill specific)

- DoDamage 1 call happens

### Destruction and ending (items and skill)

- If the rock sprite wasn't overriden earlier (or this is an item usage), it is destroyed immediately. Otherwise:
    - The sprite moves via an ArcMovement to (15.0, 0.0, 0.0) with a height of 5.0 over the course of 60.0 frames
    - The sprite gets a SpinAround with an `itself` of (0.0, 0.0, 20.0)
    - The sprite is destroyed in 2.0 seconds
- Yield for 0.5 seconds
