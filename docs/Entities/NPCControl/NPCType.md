# NPCType
This is an enum that dictates the main capabilities of an NPCControl using the `entitytype` field of that type. Each value offers wildly different behaviors with one being reserved specifically for the [shop system](Shop%20system.md#shop-system). It is the most impactful field on the lifecycle of an NPCControl.

## Enum table
The following tables includes different columns to better illustrate the main properties differences between the different enum value:

- Tag: The gameObject tag assigned on Start
- Behaviors?: Whether or not [ActionBehaviors](ActionBehaviors.md) are enabled
- Dialogues?: Whether or not [GetDialogue](Notable%20methods/GetDialogue.md) gets called which initialises a standard system to have the NPCControl call [SetText](../../SetText/SetText.md) conditionally with the lines coming from the `dialogues` field.
- Interactions?: Whether or not [Interactions](Interaction.md) are supported which allows the player (or the game via a methoid call) to interact with the NPCControl in various ways.
- Emotes?: Whether or not [CheckEmoteFlag](Notable%20methods/CheckEmoteFlag.md) gets called during LateUpdate every 3 frames when `canpause` is true and the first NPCControl in the player.`npc` list isn't this one (or the list is empty). This method initialises a standard emoticon system (this doesn't include manual changes the game does in specific situations)
- `ccol`: The state of entity.`ccol` which is the [EntityControl](../EntityControl/EntityControl.md) standard non trigger CapsuleCollider.
- `scol`: The state of the `scol` which is a trigger SphereCollider NPCControl manages.
- `pusher`: The state of the `pusher` which is a trigger CapsuleCollider with a height twice the `ccol` and a raidus of twice as narrow. This is specifically present to have the player be pushed away if they collide with it. Its enablement is updated in LateUpdate while `dummy` is false and entity.`incamera` is true which keeps it enabled as long as we aren't `inevent`, [message](../../SetText/Notable%20states.md#message) is released and we aren't in a `minipause`, it is disabled otherwise. LateUpdate also ensures its center is set to (0.0, `colliderheight` + the entity `height`, 0.0).
- `rigid` mass: The mass of the entity.`rigid`.
- Far fading?: This is some logic in LateUpdate when `dummy` is false and entity.`incamera` is true where it will slightly fade out NPCControl too far away from the camera and fade them back in when getting close enough again. The logic adjusts the alpha of the `sprite` so that it is a lerp from the existing one to either 1.0 (opaque) or 0.3 (30% visible) with a factor of a 1/10 of the game's frametime. This fades in the entity or fades it out and what determines this is the entity `compos.z`: if it's 2.5 or above, we are fading in and fading out otherwise (
TODO: unsure if it works because EntityControl also seems to manage this). The specific logic happens if all of the following are true:
    - `startlife` is above 20.0
    - The entity [animid](../../Enums%20and%20IDs/AnimIDs.md) isn't `None`
    - There is no `model`
    - The `sprite` is present
    - It's not a `hologram`
    - The current `insideid` matches this entity

|Value|Name|Tag|Behaviors?|Dialogues?|Interaction?|Emotes?|entity.`ccol`|`scol`|`pusher`|entity.`rigid`.mass<sup>1</sup>|Far fading?|
|----:|----|---|----------|---------|------------|-------|------|------|--------|-----------------------|-----------|
|0|NPC|NPC|Yes<sup>5</sup>|Yes|Yes|Yes|Enabled|Disabled|Yes<sup>5</sup>|10000.0|Yes|
|1|Enemy|Enemy|Yes|No|No|No|Enabled<sup>2</sup>|Disabled<sup>2</sup>|No|100.0|Yes|
|2|Object|Object|No|No|No|No|disabled<sup>4</sup>|Enabled<sup>3</sup>|No|10000.0|No|
|3|SemiNPC|NPC|Yes<sup>5</sup>|No|Yes|No|Enabled<sup>6</sup>|Enabled<sup>3</sup>|No|10000.0|Yes|

1: The mass is 1.0 if it's an [item entity](../EntityControl/Item%20entity.md) regardless of the type.

2: The `secondcoll` replaces it with the same attributes than the `ccol` and the player `ccol` and `detect` (its wall detector) ignores any enemy `ccol` meaning the player can only collide with the `secondcoll`, but other RigidBody can collide with both. Additionally, the player `detect` ignores the `secondcoll`.

3: The `scol` is disabled if it's an [item entity](../EntityControl/Item%20entity.md)  (which is always the case in practice for a `SemiNPC`). Also, it is left to null (not even added as a component) if it's The player [Beemerang](ObjectTypes/Beemerang.md).

4: Exceptions applies where the `ccol` remains enabled depending on `objecttype`:

- [SavePoint](ObjectTypes/SavePoint.md)
- [RollingRock](ObjectTypes/RollingRock.md)
- [FixedAnim](ObjectTypes/FixedAnim.md) (managed by a data field)
- [Item](ObjectTypes/Item.md)
- The player [Beemerang](ObjectTypes/Beemerang.md) (only remains enabled if the entity `fixedentity` is false)

5: These features are excluded for any of the following cases (Behaviors are always enabled, but [SetInitialBehavior](Notable%20methods/SetInitialBehavior.md) is not called under these cases):

- This is an [item entity](../EntityControl/Item%20entity.md). This is always the case in practice for a `SemiNPC` meaning behaviors don't actually do anything useful on them
- entity.[animid](../../Enums%20and%20IDs/AnimIDs.md) is negative
- `interacttype` is [Shop](Interaction/Shop.md) or [CaravanBadge](Interaction/CaravanBadge.md)

6: While the general initialisation process don't leave them disabled, the only cases where `SemiNPC` are used comes with a disabling of the entity.`ccol` making it disabled in practice.

While a lot of the logic of an NPCControl is shared regardless of its `entitytype`, the vast majority is dictated to a specific one that the others only partially have or do not have at all.

For more informations on each NPCType, check each of their documentations:
- [NPC](NPC.md)
- [Enemy](Enemy.md)
- [SemiNPC](Shop%20system.md#seminpc) (this type is specifically used for the shop system)
