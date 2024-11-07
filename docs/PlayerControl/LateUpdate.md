# PlayerControl LateUpdate
LateUpdate mostly updates a wide variety of states about the player including abilities usage, cooldowns and the `npc` interact list.

There's many different tasks done so they will be divided by section and they happen in order.

## `digicon` update
This only happens if `digicon` exists (should always be the case since it was initialised on Start):

- `digicon[0]` (the arrow) gets enabled if the player is `digging` and `keepdig` is above 0.0 (it's active). It is otherwise disabled. If it's enabled as a result of this, its color is also set to a lerp from pure white to pure green with a factor of Sin(Time.time * 10.0) which means it will oscillate between white and green periodically
- `digicon[1]` (the `Beetle`'s UI icon)'s enablement is set to the `digicon[0]`'s enablement as determined above

## `canfly` update
This only happens if the entity is `onground`.

The only meaningful logic is `canfly` gets set to true. There is logic surrounding `setfolloweronground`, but this field isn't practically used so the logic never happens since the field needed to be true and it never is.

## `flying` update
This only happens if the player is `flying` and `buttonhold` TODO: ??? is false: 

- If `startheight` isn't null and the player y position is less than `startheight` + 1.0, the y player position is lerped from the existing one to `startheight` + 1.0 with a factor of 0.05
- entity.`overridejump` set to true
- `flycooldown` decreases by `framestep`
- `BeeFly2` sound plays on the entity at 1.05 volume if entity.`sound` wasn't playing already
- entity.`rigid` y velocity is zeroed out
- If `flycooldown` reached 0.0 or below (it expired) the flight ends:
    - `canfly` is set to false
    - [CancelAction](../SetText/Individual%20commands/Cancelaction.md) called
    - entity.`overridejump` reset to false
    - `buttonhold` set to true TODO: ???

## `bubbleshield` update
This only happens if `bubbleshield` exists (should always be the case since it was initialised on Start):

- `bubbleshield` angles are reset to Vector3.zero
- `bubbleshield` scale is set to a lerp from the existing one. The destination vector and the factor depends on if the player is in a `shield` (it essentially reveals or hide the shield depnding if the ability is in use or not):
    - true: The lerp is to (3.0, 3.0, 2.0) with a factor of TieFramerate(0.05)
    - false: The lerp is to Vector3.zero with a factor of TieFramerate(0.0125)
- If the player isn't in a `shield` and `bubbleshield` scale magnitude is less than 0.15, its scale is snapped back to Vector3.zero
- `bubbleshield` local x position is set to a new Vector3: 
    - x: lerp from the existing one to a value that depends on entity.`flip` (-0.5 if true, 0.5 if false) with a factor of TieFramerate(0.1)
    - y: 1.25
    - z: 0.0

## `Beetle`'s digging animation update
This only happens when not in a `minipause` and entity.[animid](../Enums%20and%20IDs/AnimIDs.md) is `Beetle`:

- If entity.`sprite` local y position is less than -0.1 (we are going underground):
    - entity.[animstate](../Entities/EntityControl/Animations/animstate.md) set to 101
    - entity.`overrideanim` set to true
    - entity's y `spin` set to 20.0
- Otherwise, if entity.`overrideanim` is true (meaning that what was done above previously needs to be undone):
    - entity.`overrideanim` is reset to false
    - entity.`spin` is zeroed out

## Various unpaused updates
This section includes many updates that only happen when not in an instance.`minipause` or instance.`pause`.

### Cooldowns and timers update
Several cooldowns are decreased by `framestep` if they were above 0.0 (not expired yet):

- `boulderbreak`
- `interactcd`
- `keepdig`

If `idletime` is above 250.0, both instance.`showmoney` and instance.`hudcooldown` are set to 10.0 which reveals the HUD and the money counter HUD.

Finally, if the player isn't in a `submarine` and entity.`icooldown` expired (no longer invulnerable), entity.`sprites` gets enabled if `digging` is false, disabled if it's true. TODO: ??? 

### `Pushable` emoticon update
This happens only if all of the following are true:

- The player isn't `digging` (or `startdig`)
- The player isn't `flying`
- The player isn't in a `shield`
- The player isn't in a `submarine`
- entity.`emoticoncooldown` is 0.0 or below (it expired)
- [flag](../Flags%20arrays/flags.md) 17 is true (got Horn Slash)
- There is a GameObject with a `Hornable` tag that exists within a distance of less than 2.5 from the player

If all of the above are fufilled, [Emoticon](../Entities/EntityControl/EntityControl%20Methods.md#emoticon) is called on the entity with the `Pushable` emoticon with a time of 5.

### `npc` interact list update
This only happens if all of the following conditions are true:

- `npc` isn't empty
- The [message](../SetText/Notable%20states.md#message) lock is released
- The player isn't `digging` (or `startdig`)
- The player isn't `flying`
- The player isn't in a `shield`

Here's what happens if all of the above are fufilled:

- Every 3 frames, if there's more than 1 `npc`, the list is reordered in ascending distance order from the player position
- If `npc[0]` is `dead` or `iskill`, it is removed from `npc`. Otherwise, its NPCControl is collected for potential adjustements:
    - If the NPCControl is an [NPC](../Entities/NPCControl/NPC.md) or [SemiNPC](../Entities/NPCControl/Shop%20system.md#seminpc), their `emoticonid` and `emoticoncooldown` may change depending on their [Interaction](../Entities/NPCControl/Interaction.md) (otherwise, if the NPCControl is an [Object](../Entities/NPCControl/Object.md) or [Enemy](../Entities/NPCControl/Enemy.md), it is removed from `npc`):
        - For [Talk](../Entities/NPCControl/Interaction/Talk.md), [ShopKeeper](../Entities/NPCControl/Interaction/ShopKeeper.md), [StorageAnt](../Entities/NPCControl/Interaction/StorageAnt.md) or [VenusHeal](../Entities/NPCControl/Interaction/VenusHeal.md), `emoticonid` is set to 0 (speaking emote) and `emoticoncooldown` is set to 2.0
        - For [LockedDoor](../Entities/NPCControl/Interaction/LockedDoor.md), [Check](../Entities/NPCControl/Interaction/Check.md), [Shop](../Entities/NPCControl/Interaction/Shop.md), [QuestBoard](../Entities/NPCControl/Interaction/QuestBoard.md), [Event](../Entities/NPCControl/Interaction/Event.md) or [CaravanBadge](../Entities/NPCControl/Interaction/CaravanBadge.md): `emoticonid` is set to 1 (green ! mark) and `emoticoncooldown` is set to 2.0
    - If the NPCControl has a [Shop](../Entities/NPCControl/Interaction/Shop.md) or [CaravanBadge](../Entities/NPCControl/Interaction/CaravanBadge.md) `Interaction`, instance.`showmoney` changes depending on their `shopkeeper`.`dialogues[1]` being 1:
        - If it's 1 (the [shop system](../Entities/NPCControl/Shop%20system.md) does accept crystal berries), it's set to 0.0 which hides the berry count HUD
        - If it's not 1 (the [shop system](../Entities/NPCControl/Shop%20system.md) only takes regular berries), it's set to 10.0 which reveals the berry count HUD

## Party switch update
What happens here depends on `switchcooldown` (nothing happens if it's -99999.0 which means no switch is happening and the one that last happened was finalised):

- Higher than 0.0 (a switch is ongoing):
    - `switchcooldown` is decreased by `framestep`
    - Every `playerdata` entity has their y `spin` set to -20.0 if their `flip` is false or to 20.0 if their `flip` is true
- 0.0 or below, but not equal to -99999.0 (the switch needs to be finalised):
    - All `playerdata` entities have their `spin` zeroed out and [SetDialogueBleep](../Entities/EntityControl/EntityControl%20Methods.md#dialogue-bleep) called on them
    - MainManager.RefreshEntities is called with onlyplayer which will update each `playerdata`'s entity.[animid](../Enums%20and%20IDs/AnimIDs.md) to their BattleData's `animid` and reset their `hitwall` to false
    - `switchcooldown` is set to -99999.0 which finalises the switch so these switch updates can't happen anymore

## `pausecooldown` update
This only happens if `pausecooldown` is above 0.0 (it hasn't expired) while the player `canpause`.

The only thing that happens is `pausecooldown` is decreased by `framestep`.

## `map`.[`ylimi`](../MapControl/Map%20entities.md#ylimit) update
This only happens when the `map`.`ylimit` constraint is violated on the player which is the case if the player y position is lower than the `map`.`ylimit`:

- Player position set to `lastpos` which teleports them to the last recorded respawn point
- entity.`rigid` velocity zeroed out

## Movement update
This only happens when the player isn't `dashing` as the movement there is entirely handled by [DashBehavior](DashBehavior.md).

The only thing that happens here is a call to [Movement](Movement.md) which is what handles all the movement logic from what Update last gathered when processing the movement inputs.

## `actioncooldown` update
This only happens when `actioncooldown` is above 0.0 (it hasn't expired).

The only thing that happens is `actioncooldown` is decreased by `framestep`

## entity.`speed` update
This always happen and it involves a call to RefreshSpeed which is a method that sets entity.`speed` depending on the player's state. The new value is determined by looking through the cases below and the first case that applies below will be chosen:

- If the player is in a `submarine`, the new speed is `basespeed` / 1.8 (should always be 2.7777777778)
- If the player is `dashing`, the new speed is `basespeed` * 2.5 (should always be 12.5)
- If neither of the cases above applies, the new speed is (`basespeed` + entity.`ccol`.material.dynamicFriction) * 1.3. In practice, because `basespeed` is always 5.0 and the `ccol` dynamic friction is always either 0.0 or 1.0, it means that the new speed is always 6.5 or 7.8

## `tattling` end update
This only happens when the player was `tattling`, but the [message](../SetText/Notable%20states.md#message) just released which means the `tattling` needs to end:

- If we aren't instance.`inevent` or in an instance.`minipause`, the entity.`rigid` is placed out of kinematic mode (this is because the tattling had its SetText called prefixed with [kinematicplayer](../SetText/Individual%20commands/Kinematicplayer.md) which placed it in kinematic mode)
- `tattling` is set to false

## `submarine` sound update
This is always done:

- If the player is in a `submarine` and isn't `digging`: TODO: this looks sus to check digging
    - The `Submarine` sound is played on the entity on loop if entity.`sound` wasn't looping already
    - entity.`sound`.pitch is set to entity.`rigid` velocity's magnitude clamped from 0.65 to 1.0
- If the player isn't in a `submarine`, but entity.`sound` is still looping, it stops looping
