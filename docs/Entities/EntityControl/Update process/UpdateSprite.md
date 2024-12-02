# UpdateSprite
This update method is called by [LateUpdate](Unity%20events/LateUpdate.md). It handles `sprite` related adjustments as well as complex animation related updates.

Since this is a complex update method, a more detailed series of steps will be described, but basically, this method has 3 segments.

The first one involves updating the sprite and its animation under most conditions. The second segment involves disabling the sprites and animations if those conditions weren't met earlier and the `animid` was -1 (None). Finally, the third sets the `sprite`'s material renderQueue under specific conditions.

## Sprites and animation updates
This is the main update segment of the method where `animid`, `animstate` and sprites related matters are updated. It is done for any `animid` above -1 (meaning it's not None) and also whenever any of the following changes:

* `inice` (only counts as long as the entity `hasiceanim` otherwise, this is ignored)
* `animid`
* [animstate](../Animations/animstate.md)
* `backsprite`
* `talking`
* `flyinganim`

The way the game tracks changes to these is with internal fields that stores the previous value of each of these. They are only assigned at the end of this update segment no matter the entity. They are (in the same order):

* `lastice`
* `oldid`
* `oldstate`
* `oldback`
* `oldtalk`
* `oldfly`

It means essentially that this segment only applies if any of these fields changed and that the `animid` is defined.

### Item animation updates

If it's an [item entity](../Item%20entity.md), the update is mostly simplified to [UpdateItem](UpdateItem.md) which sets the sprite of the item to `itemstate` as the [Items](../../../Enums%20and%20IDs/Items.md) or [Medal](../../../Enums%20and%20IDs/Medal.md) id and using the `animid` as the type of item (0 is standard item, 1 is key items, 2 is medals and 3 is Crystal Berry which disables the sprite). This method also makes sure that the MYSTERY? icon is rendered when applicable. Finally, if the sprite is kept enabled by the end of that method, its local position's y is set to the sprite's extents.y.

If it's not an item, several updates are done.

### Non item animation updates
First, if the [animid](../../../Enums%20and%20IDs/AnimIDs.md) changed and there is no `model`, then this is handled by first calling `SetDialogueBleep` which updates the bleeps for the new `animid`. Then, `anim` is ensured to be initialised by either called `ForceAnimator` if it wasn't or `SetAnimator` + [UpdateAnimSpecific](../Animations/AnimSpecific.md#updateanimspecific) if it was. The only difference is that `ForceAnimator` will create the `anim` if needed, but the rest ends up being the same because `ForceAnimator` also calls [UpdateAnimSpecific](../Animations/AnimSpecific.md#updateanimspecific) after a `SetAnimator` call.

#### SetAnimator
What `SetAnimator` does is to ensure the `anim`'s controller is correctly set. This is also where PUSHROCK overrides and where the Hard Mode medal on HARDEST overrides the controller when applicable. Essentially, the logic works like this:

* If the `original` id is 132 (`Tanjerin`) under PUSHROCK, it is overriden to the `Beetle` one with the `animid` being 1
* If HARDEST is active on the `playerentity` with Hard Mode equipped, the controller is overriden to the `Pillbug` one
* Under PUSHROCK, for any entity that doesn't have a `model`, the controller is overriden to `Tanjerin` except for `Cerise` and `TangySeller` as well as `overridefly` and `overridejump` being set to true
* If none of the above applies, the controller is set to be the [AnimID](../../../Enums%20and%20IDs/AnimIDs.md)'s normal one

Next comes the digging handling. If the entity is `digging` and it has a `diganim`, its [animstate](../Animations/animstate.md) is overridden to `Dig` if it was `Idle` or `DigMove` if it was `Walk`.

Finally, this is where the [SetAnim](../Animations/SetAnim.md) call happens. The logic goes like this (checked in order, only the first branch that applies executes):

* The `f` argument is sent if `overridefly` is false, the `height` is higher than 0.1 and the [animstate](../Animations/animstate.md) is among the following: `Idle`, `Walk`, `ItemGet`, `Chase`, `ItemWalk`, `Sleep`, `Hurt`, `BattleIdle` or `Woobly`. The `ft` argument is sent if on top of this if `notalk` is false and the entity is `talking`.
* Only the `t` argument is sent if the [animstate](../Animations/animstate.md) is a [Predefined animations names](../Animations/animstate.md#predefined-animations-names), the entity is `talking` while `notalk` is false. This also set `backsprite` to false.
* Only the `b` argument is sent under the following conditions:
    * We aren't in PUSHROCK
    * We either aren't in a battle or we are in a battle event
    * `backsprite` is true
    * The [animstate](../Animations/animstate.md) is `Idle`, `Walk` or `Jump`
    * The `originalid` is `Bee`, `Beetle`, `Moth`, `Madeleine` or `BLANK`
* If none of the above applies, no arguments are sent

After the [SetAnim](../Animations/SetAnim.md) call, `backsprite` is set to false for any [animstate](../Animations/animstate.md) that isn't `Idle`, `Walk`, `Jump` or `Fall`.

### Sprite enabled adjustment according to the model
This part is done regardless if the entity is an `item` or not.

Whether or not the `sprite` can get enabled depends on the values of `nomodel` and whether there is a `model` present. `nomodel` being true forces the `sprite` to be enabled, but if it's not, it is enabled if `model` isn't present and disabled otherwise.

However, this adjustment won't happen if `overrideanim` is true or that there is an `npcdata` of the `Enemy` with a `disguideobj` present.

### Final adjustements
This part is done regardless if the entity is an `item` or not.

[AnimSpecificQuirks](../Animations/AnimSpecific.md#animspecificquirks) is called if `inince` or the [animstate](../Animations/animstate.md) has changed. If this happens, this is the second call of [LateUpdate](Unity%20events/LateUpdate.md) because this method was called before.

Finally, this is where the old fields are set to their current one and it ends this segment.

## Disabling sprites and animations
This happens if the regular update segment didn't occured as long as the `animdid` is -1 (none). First, the `anim`'s runtimeAnimatorController is set to null if we didn't had an `npcdata` or the `originalid` was -1 (None) to start with.

Lastly, unless `overrideanim` is true, both the actual sprite of the `sprite` is set to null and `sprite` is disabled if it wasn't.

## renderQueue adjustment
This segment has 2 possible logic:

- `hologram` is false and that either it's a `battle` entity or that it has no `npcdata` or it has one, but it's either not an NPC type or if it is one, its `startlife` is less than 50.0.
- `hologram` is true and it's a `battle` entity

For the first one, the `sprite`'s material renderQueue is set to 2450 if the alpha was higher than 0.9 or it is set to 3000 otherwise.

For the second one, the `sprite`.material.renderQueue is set to 2500.
