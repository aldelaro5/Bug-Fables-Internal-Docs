# LateUpdate
Nothing happens if the player doesn't exist.

## `latestart`

- `tempfollowers` gets updated from instance.`extrafollowers`:
    - If the object's name is `0` (only the case with the `TestRoom` map), MainManager.AddFollower is called with instance.`extrafollowers` without caller (so every followers is allowed)
    - Otherwise, only the ones with an animid present in `canfollowID` will be added. When each is added, their `tempofollowerid` gets updated to the found `cantfollowID` index and their `onground` is set to false. If the animid is `Maki`, their `ccol` height is set to 3.0 and center to 1.5 in y. NOTE: `tempfollowerid` isn't set correctly because it's supposed to be `tempfollowers` index, NOT `canfollowID` index
- If flag 402 is true (Chompy is with Team Snakemouth) and the player isn't in a `submarine`:
    - MainManager.AddFollower is called with animid 169 (`ChompyChan`) without caller
    - `chompy` is set to the follower that was just added, their `onground` is set to false and their `tempfollowerid` is set to the matching `tempfollowers` index
- SetParticles called which, if MainManager.`particlelevel` is 0, will look for any GameObject with the `Particable` tag and disable all of them
- PreloadSprites is invoked in 0.1 seconds which will set `entitysprite` to each of `entities`'s `sprite`'s texture
- If `faderchange` is true:
    - `faders` is set to all Fader in this map
    - `fss` is set to all the enablement of all `faders`
- SetPlayerColliders is invoked in 0.2 seconds which will find all the first collider of all GameObject with the tag `EntityOnly` and assign them to `entityonly`. These colliders gets their staticFriction and dynamicFriction set to 0.0 and all `playerdata` and `tempfollowers` entities ignores these colliders
- `latestart` is set to true which blocks this logic from happening again

## Non `latestart`

- If not in a `minipause`, `pause`, `inevent` or `message`:
    - If `autoevent` exists, they are processed in order. Each x component correspond to a flag that must be false for the event id y to be started immediately without caller. If event y was started, its x flag slot is set to true (this means multiple events could start at the same time due to this)
    - If `hiddenitem` isn't null, the detector emote is set with player.entity.`emoticonid` set to 4 with an `emoticoncooldown` of 100.0. `hiddenitem` is set to null after
- CheckStencilSwitch called which finds the first StencilSwitch that's not `iskill` and is `hit`:
    - If one is found:
        - `GlobalIceRadius` shader global property gets set to the StencilSwitch's `internaltransform[0]` scale magnitude / 2.0
        - `CentralIcePoint` shader global property gets set to the StencilSwitch's position
        - `stencilid` gets set to the map entity id of the StencilSwitch
    - Otherwise (none is found):
        - `GlobalIceRadius` shader global property gets set to a lerp from it to 1/20 of the game's frametime ???
        - `stencilid` gets set to -1


- If the detector sound isn't playing, but if the detector emote is playing still (player.entity.`emoticonid` is 4 and `emoticoncooldown` hasn't expired yet), `Select1` sound is played on `sounds[11]` with 1.1 pitch and 0.25 volume
- All `digwall`'s enablement gets updated with the player.`digging` value: if the player is `digging`, they are disabled and if the player isn't `digging`, they are enabled

## `alivetime` is higher than 0
It is decreased by 1.0 and no further logic happens from here. Essentially, it blocks all the logic that would have happened below for a certain amount of frames including the first one.

## `alivetime` is 0

- Every 3 frames, if `faderchange` is true and there `faders`, each enabled one whose `fss` is true (it started enabled) has the following happen to them:
    - TODO: check Fader
- Every 2 frames, NPCControl updates happen for any that have their `requires`, `limit` and `regionalflag` condition fufilled to exist:
    - DoorOtherMap: The enablement is updated to be only enabled only if the player is less than 15.0 units away ignoring the y axis (disabled otherwise). If the enablement changes due to this, the `emoticon` of the entity is disabled
    - NPC (with an `originalid` defined and only if the map's `keepobjectsactive` is false): The enablement is updated to be only enabled only if the player is less than the NPC's `radius` * 2.0 units away ignoring the y axis (disabled otherwise). If the enablement changes due to this and `interacttype` isn't `Talk`, the `emoticon` of the entity is disabled