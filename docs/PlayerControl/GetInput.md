# GetInput
This method is called by Update during the regular input processing. It processes all digital buttons inputs:

- Help
- Switch
- Pause
- Jump/Interact
- Ability
- HUD toggle

## Help, switch and pause inputs
These 3 inputs are processed in a mutually exclusive way meaning at most, only one of the 3 can get processed at once. Their priority are in this order: Help, Switch and Pause.

Each requires the input to be pressed without hold and each also requires a bunch of conditions for the processing to happen. This means that if multiple of the 3 inputs are pressed, the first one in the priority order whose conditions are all fufilled will get processed.

### Help input
The input is processed if all of the following conditions are true:

- [flag](../Flags%20arrays/flags.md) 10 is true (able to be `tattling`, remains true after receiving the tattle tutorial except during the Overseer rescue in chapter 3)
- The entity is `onground`
- `switchcooldown` is 0.0 or below (there's no ongoing switching)
- `actioncooldown` is 0.0 or below (there's no ongoing `tattling` or action)
- entity.`jumpcooldown` is 0.0 or below (the entity is able to jump)
- entity.`rigid` y velocity is at least -0.1 (meaning the entity can't be falling down by too much if they were on a sloped surface)
- The player isn't `digging`

If all of the above conditions are fufilled:

- entity.`detect` LookAt its position + `lastdelta` (so it alligns the wall detector in case it wasn't to where the entity was going)
- The followers are teleported by 5.0 away from the entity
- [CancelAction](Actions/CancelAction.md) called
- instance.`showmoney` set to 0.0 (hidden)

From there, the `tattling` will occur with a [SetText](../SetText/SetText.md) call in [dialogue mode](../SetText/Dialogue%20mode.md), but the [dialogue line id](../SetText/Common%20commands%20id%20schemes/Dialogue%20line%20id.md) used to resolve the main part of the text sent depends on how the `tattling` happens:

- If `npc[0]` exists and it has a `tattleid` that isn't -1 (not undefined), the id is `npc[0].tattleid`
- Otherwise, the id is the `map`'s [`tattleid`](../MapControl/SetText%20configuration.md#tattleid)

As for the parameters of the call, it's always the following (done in [dialogue mode](../SetText/Dialogue%20mode.md)):

- text: `|`[kinematicplayer](../SetText/Individual%20commands/Kinematicplayer.md),temp`|` followed by the string obtained by resolving the [dialogue line id](../SetText/Common%20commands%20id%20schemes/Dialogue%20line%20id.md) determined above
- [fonttype](../SetText/Notable%20states.md#fonttype): `BubblegumSans`
- linebreak: `messagebreak`
- tridimensional: false
- position: Vector3.zero
- cameraoffset: Vector3.zero
- size: Vector2.one
- parent: [entity id](../SetText/Common%20commands%20id%20schemes/Entity%20id.md) -5 (the entity with the `Beetle` [animid](../Enums%20and%20IDs/AnimIDs.md))
- caller: null

Finally, after starting the SetText call, `tattling` is set to true.

### Switch input
The input is processed if all of the following conditions are true:

- No Help input was processed above
- The entity is `onground`
- `action` is false (no [DoActionTap](Actions/DoActionTap.md) is in progress)
- `switchcooldown` is 0.0 or below (there's no ongoing switching)
- `beemerang` is null (the [Beemerang](../Entities/NPCControl/ObjectTypes/Beemerang.md) hasn't been thrown)
- `buttonhold` is false TODO: ???
- The player isn't `digging` and `startdig` is false
- The player isn't `flying`
- The player isn't in a `submarine`

If all of the above conditions are fufilled, the input is considered processed, but it will only cause SwitchOrder to be called if there are more than 1 `playerdata`. If there's only one, the input is still considered processed, but nothing will happen.

Here's what SwitchOrder does:

- `switchcooldown` set to 15.0 (the switch will lasts for 15.0 frames)
- MainManager.SwitchParty is called without battle which does the following when not `inevent`:
    - `Switch` sound plays
    - All `playerdata`'s `animid` (NOT their entity.`animid`) gets rotated with each other such that they get assigned to the next one's with wrap around to the first
    - instance.`switchicon` sprite is set to a `guisprite` whose index is 94 + `playerdata[0]`'s animid (the party member's gui icon) after it was rotated with a color of pure white at 50% opacity
    - `playerdata[0]`.entity.`emoticonoffset` is set to (0.0, 1.8 + 0.25 * `playerdata[0]`.animid, -0.1). This just ensures the `emoticonoffset` is consistent for the new party leader even if it probably was already at this value
- instance.`partyorder` is reset to have its elements contain the `animid` (the `trueid` in this case) of each `playerdata` in the order they appear in the array

It should be noted that this only starts the party switch. The `playerdata`'s entity.[animid](../Enums%20and%20IDs/AnimIDs.md) haven't been updated yet so even visually, the party's rendering haven't changed (though, LateUpdate will apply a y `spin` during the switch on each `playerdata` entity). The switch will only complete on the LateUpdate cycle that `switchcooldown` expires at which point, they will be updated and the switch will be fully done.

### Pause input

The input is processed if all of the following conditions are true:

- No Help input or Switch input was processed above
- The player `canpause`
- The entity is `onground`
- `buttonhold` is false TODO: ???
- The player isn't `digging`
- The player isn't in a `shield`
- The player isn't `flying`
- `pausecooldown` is 0.0 or below (it expired)

If all of the above conditions are fufilled, Pause is called which does the following:

- A new GameObject named `PauseMenu` is created rooted with a PauseMenu component that will manage the pause UI
- `StartOpen` sound plays at 0.5 volume
- We enter an instance.`pause`
- The `beemerang` is destroyed if it exists

Since we enter a pause, it will actually prevent anything further in GetInput to happen. This means that processing a pause input prevents further inputs to be processed at all.

## Jump/Interact input
This input is processed on its own if all of the following conditions are fufilled if the input was pressed without hold:

- We aren't in a instance.`pause`, instance.`minipause` or instance.`inevent`
- Either the entity is `onground` OR `jumpcooldown` expired while entity.`offgroundframes` is less than 3.0 (essentially, it accepts the input even if the entity left the ground since less than 3.0 frames so it's coyote time logic)
- `action` is false (no [DoActionTap](Actions/DoActionTap.md) is in progress)
- The player isn't `digging`
- `buttonhold` is false TODO: ???
- The player isn't in a `shield`
- The player isn't `flying`

If all of the above are fufilled, the meaning of the input is determined if it's an interaction or a jump or ignored entirely. It's an interaction if all of the following conditions are fufilled:

- `npc[0]` exists it's an [NPC](../Entities/NPCControl/NPC.md) or [SemiNPC](../Entities/NPCControl/Shop%20system.md#seminpc)
- `npc[0].insideid` matches the current [inside](../MapControl/Insides.md)
- The player entity is `onground` (so this doesn't accept the coyote time condition)
- `npc[0]`'s `nointeract` is false
- `switchcooldown` is 0.0 or below (there's no ongoing switching)

If all these conditions are fufilled, this is how the inteactions is done:

- [CancelAction](Actions/CancelAction.md) is called
- If `npc[0]`'s `interactcd` is 0.0 or below (the cooldown on its interaction expired), [Inteact](../Entities/NPCControl/Notable%20methods/Interact.md) is called on `npc[0]` with null as the args

If the interactions conditions aren't fufilled, then it's a jump if the player isn't in a `submarine`. In that case, DoJump is called which does the following:

- entity.`jumpcooldown` set to 10.0 (can't jump again for the next 10.0 frames)
- [Jump](../Entities/EntityControl/EntityControl%20Methods.md#jump) called on the entity
- `Jump` sound played on the entity
- DisableFly is invoked in 0.1 seconds which will set `canfly` to false

Otheriwse (the player is in a `submarine` and it's not an interaction), the input is ignored and nothing happens.

## Ability input
This input is processed on its own if all of the following conditions are fufilled (it doesn't require the input to be pressed in any way, check below for details):

- We aren't in a instance.`pause`, instance.`minipause` or instance.`inevent`
- `action` is false (no [DoActionTap](Actions/DoActionTap.md) is in progress)
- `beemerang` is null (the [Beemerang](../Entities/NPCControl/ObjectTypes/Beemerang.md) hasn't been thrown)
- `actioncooldown` is 0.0 or below (there's no ongoing `tattling` or action)
- `switchcooldown` is 0.0 or below (there's no ongoing switching)
- instance.[isholdingskip](../SetText/Related%20Systems/Text%20advance.md#text-skip) is false
- The player isn't `dashing`

If the above conditions aren't fufilled, but `isholdingskip` is still true while the input is released, then `isholdingskip` is reset to false. This is done because for some reasons, the field isn't reset after a SetText call in [dialogue mode](../SetText/Dialogue%20mode.md) so PlayerControl resets it here.

This input is special in the sense that PlayerControl only listens to the input being held and released, but it doesn't care if it was just pressed. It's monitoring the input for a much longer period of time than the others.

If all the above conditions are fufilled, it means that either the input is still being held or it had just been released and it's not related to [isholdingskip](../SetText/Related%20Systems/Text%20advance.md#text-skip).

If the input is still being held:

- `actionhold` is increased by `framestep`
- If `actionhold` just reached 20.0 or above and the player isn't in a `submarine`, [DoActionHold](Actions/DoActionHold.md) is called

It means that the threshold for the input to be considered a hold action is 20.0 frames of holding it or more.

However, even if the input isn't being held, DoActionHold can sell be called if the player is `digging` while `keepdig` is above 0.0 (still active). In that case, the hold action happens regardless if the input has been held for 20.0 frames or not.

If neither of the hold action case mentioned above happens, then the tap action logic is checked since it means the input was just released or it wasn't pressed in general since more than a frame:

- If `actionhold` is above 0.0, but below 20.0 while the entity is `onground`, `actionroutine` is set to a new [DoActionTap](Actions/DoActionTap.md) call
- `actionhold` is reset to 0.0
- `buttonhold` is reset to false TODO: ???
- If the player is `digging`:
    - `uproot` is set to true (it allows [DigSpot](../Entities/NPCControl/ObjectTypes/DigSpot.md) to reveal their object when colliding)
    - CancelUproot is invoked in 0.1 seconds which will set `uproot` back to false (so it only works for the next 0.1 seconds)
- If the player is `flying`, `digging` or in a `shield` while not in a `submarine`:
    - `canfly` is set to false (it will be reset to true on LateUpdate when the entity is `onground`)
    - [CancelAction](Actions/CancelAction.md) is called

## HUD toggle input
This input is processed on its own if all of the following conditions are fufilled if the input was pressed without hold:

- We aren't in a instance.`pause`, instance.`minipause` or instance.`inevent`
- The player `canpause`

If all of the above are fufilled:

- If instance.`hudcooldown` is 0.0 or below (the hud is hidden):
    - `HudDown` sound plays on `sounds[13]` with 0.15 volume
    - instance.`hudcooldown` is set to 300.0 so it stays revealed for around 5 seconds
    - instance.`showmoney` is set to 300.0 so it stays revealed for around 5 seconds
- Otherwise:
    - `HudUp` sound plays on `sounds[13]` with 0.15 volume
    - instance.`hudcooldown` is set to 1.0 so it will hide itself on the next frame
    - If instance.`money` is the same than instance.`moneyt` (meaning the UI berry count number is the same than the actual number), instance.`showmoney` is set to 1.0 so it will hide itself on the next frame
