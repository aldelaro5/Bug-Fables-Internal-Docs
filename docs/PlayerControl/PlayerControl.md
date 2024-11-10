# PlayerControl
PlayerControl is a component that is essentially the player's surrogate in the overworld: it has everything the player can control directly or do whenever it has control. It is meant to be attached to an entity, but that entity for a multitude of reasons is assumed to be `playerdata[0]` which also assumes that it is part of the player party (so its entity.[animid](../Enums%20and%20IDs/AnimIDs.md) should only be `Bee`, `Beetle` or `Moth`).

Only at most one PlayerControl may exists at a time and it can be accessed via MainManager.`player`. The only time a PlayerControl should ever be null is when it hasn't been created yet (such as during the StartMenu) or when it is being recreated (which is the case during [map loading](../MapControl/Map%20loading.md)).

## Organisation and design
This component is relatively cohesive in the sense it only has one responsibility: handle player controls. This heavily involves processing inputs and reacting accordingly.

This is done the usual way through Unity events:

- [Start](Start.md): Only called when the player is created which normally happens on each map load
- [Update](Update.md): The main update cycle where regular movements inputs are read and other digital buttons inputs are reacted upon via [GetInput](GetInput.md), but it's possible everything is overriden by [DashBehavior](DashBehavior.md) when the player is `dashing`
- [LateUpdate](LateUpdate.md): The secondary update cycle where a bunch of state is kept updated after Update with the most notable logic being the [Movement](Movement.md) done from the movement inputs that was read (assuming the player isn't `dashing`)
- [FixedUpdate](FixedUpdate.md): This is where `digging` / `startdig` state update happens as well as some other miscellaneous update tasks
- [OnTriggerStay and OnTriggerExit](Trigger%20collider%20handling.md): The places where collisions with trigger colliders who have specific tags are handled. Typically, staying in a collider changes the state while exiting it reverts it back

Overall, the structure of the component is relatively straight forward. However, its logic is dense because it has to handle a lot of different player features many of them have various edgecases when they can interact with each other.

There is a very limited public API surface. Other than some methods mentioned on this page, the only method intended to be called only externally is [JumpTo](JumpTo.md) which is a scripted move animation of the player party and its followers.

It should be noted that just like with most of the game, the component becomes very inactive when in an instance.`minipause`, instance.`pause` or the [message](../SetText/Notable%20states.md#message) lock is grabbed.

## Concept of one player entity
The vast majority of PlayerControl only concerns a single entity: the [EntityControl](../Entities/EntityControl/EntityControl.md) attached to the same GameObject (which is normally always `playerdata[0]`.`entity`). The rest like the other `playerdata` and the `map`.`tempfollowers` aren't usually directly addressed by PlayerControl because there are other mechanisms where they are handled based on what PlayerControl ends up doing. The main exception to this rule is [JumpTo](JumpTo.md), but it's meant to be called externally.

For example, moving the player entity will cause the [Follow](../Entities/EntityControl/Notable%20methods/Follow.md#follow) system to kick in for the player party and their followers.

In a sense, it means that PlayerControl only concerns itself with what is refered to as the player party leader which is always `playerdata[0]`. It is the only entity required for PlayerControl to function.

## Relation with the player party
The player party is handled in an interesting way because while PlayerControl should always reside on `playerdata[0]`, it allows to switch the party in such a way that it makes it appear as if the entity that has PlayerControl changed, but that never happens.

A party switch is done in 2 phases from the moment the input is processed in [GetInput](GetInput.md):

1. The input gets processed and it sets `switchcooldown` to the amount of frames to wait before finalising the switch. This is kept track in LateUpdate and while movement is allowed during this time, some inputs processing are disabled. The only notable thing that happens besides some visual animations is the `playerdata`'s `animid` (the BattleData one, NOT the entity.[animid](../Enums%20and%20IDs/AnimIDs.md)) gets rotated between the party members. This doesn't actually change the leader or anyone visually yet
2. Once `switchcooldown` expires, LateUpdate finalises the switch by calling MainManager.RefreshEntities with onlyplayer which updates each `playerdata`'s entity.[animid](../Enums%20and%20IDs/AnimIDs.md) with their BattleData's `animid`.

What has effectively happened is the PlayerControl is still on `playerdata[0]`, but because its entity.`animid` changed, it will render as if it was another party member. The same goes for the entire party so it makes it seem as if the entire party rotated, but they haven't: only their rendering did. PlayerControl will check the BattleData `animid` of `playerdata[0]` attached to select the field abilities available.

## Field abilities
PlayerControl is also what directs the [field abilities](Field%20abilities.md) logic, but it also collaborates with [EntityControl](../Entities/EntityControl/EntityControl.md) for most of them (and with a [Beemerang](../Entities/NPCControl/ObjectTypes/Beemerang.md) with the case of the `Bee` hold action).

There's only 2 types of actions that can lead to a field ability being used, but all of them requires conditions to be used:

- [DoActionTap](Actions/DoActionTap.md): This is done when the ability input was held for less than 20.0 frames and then released. It's a coroutine which allows some action to handle a secondary variant via a second press of the ability input:
    - `Bee`: [Beemerang Toss](Field%20abilities.md#beemerang-toss) (Halt is handled separately in the [Beemerang](../Entities/NPCControl/ObjectTypes/Beemerang.md) logic which is created as part of the toss)
    - `Beetle`: [Horn Slash](Field%20abilities.md#horn-slash) ([Dash](Field%20abilities.md#dash) or [Horn Dash](Field%20abilities.md#horn-dash) if a second tap is detected within 15.0 frames which sets the player to be `dashing` and has [DashBehavior](DashBehavior.md) override most inputs processing)
    - `Moth`: [Freeze](Field%20abilities.md#freeze) ([Icicle](Field%20abilities.md#icicle) if a second tap is detected within 25.0 frames)
- [DoActionHold](Actions/DoActionHold.md): This is done when the ability input was held for 20.0 frames or more. It is a simple method that simply does the action or resumes an existing one:
    - `Bee`: [Bee Fly](Field%20abilities.md#bee-fly) which sets the player to be `flying`
    - `Beetle`: [Beetle Dig](Field%20abilities.md#beetle-dig) which sets the player to be in a `startdig` causing the `sprite` local y position to decrease in LateUpdate and eventually, it will hit a threshold where the player is now `digging`
    - `Moth`: [Shield](Field%20abilities.md#shield) which sets the player to be in a `shield`

At any point, an action in progress can be cancelled by the [CancelAction](Actions/CancelAction.md) method which will revert any state changes any actions has done and even call [StopDash](StopDash.md) to stop [DashBehavior](DashBehavior.md) if the player was `dashing`.

## Key states
The state of PlayerControl is complex and all its fields are outlined in the [PlayerControl fields](Class%20fields.md) documentation, but there are some key ones important to be summarised here:

- `entity`: The [EntityControl](../Entities/EntityControl/EntityControl.md) of the player, usually `playerdata[0]`.`entity`
- `submarine`: If true, the player is in a submarine which greatly limits their function and changes several aspects of PlayerControl such as moving a bit slower. More information can be found in the [submarine](Submarine.md) documentation
- `dashing`: If true, most input processing and movement are overriden by [DashBehavior](DashBehavior.md) at the expense that the movement speed increases by a lot. For more information, check the [Dash](Field%20abilities.md#dash) documentation
- `shield`: If true, it places the player party in a shield using `bubbleshield`. For more information, check the [Shield](Field%20abilities.md#shield) documentation
- `flying`: Only set to true by `Bee`'s hold action which lets the player travel for a short time (`flycooldown` which starts at 240.0 frames) while the entity.`rigid` gravity is disabled. For more information, check the [Bee Fly](Field%20abilities.md#bee-fly) documentation
- `digging`: Only set to true after some time after `startdig` which is only triggered by `Beetle`'s hold action. For more information, check the [Bettle dig](Field%20abilities.md#beetle-dig) documentation. NOTE: The semantic of this field changes when the player is in a `submarine`, check the [submarine](Submarine.md) documentation to learn more
- `npc`: A list maintained in collaboration with [NPCControl](../Entities/NPCControl/NPCControl.md)'s LateUpdate that orders NPCControl that are close enough to the player to [Interact](../Entities/NPCControl/Notable%20methods/Interact.md) with them. The list is ordered in ascending distance order to the player and `npc[0]` is the only NPCControl that can be interacted with
- `tattling`: If true, a [SetText](../SetText/SetText.md) call in [dialogue mode](../SetText/Dialogue%20mode.md) is ongoing that gives help to the player about a [map](../MapControl/SetText%20configuration.md#tattleid) or an [NPC](../Entities/NPCControl/NPC.md). Triggered by pressing the help input in [GetInput](GetInput.md)
