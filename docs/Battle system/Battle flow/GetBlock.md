# GetBlock
This is a parameterless void returning method that is called from [Update](Update.md) when in an [uncontrolled flow](Update%20flows/Uncontrolled%20flow.md).

This logic is exclusive to an uncontrolled flow, but it only happens when `enemy` is true (we are in the player phase / a [delproj](../Actors%20states/Delayed%20projectile.md) is landing / an enemy party member is performing a [hitaction](../Actors%20states/Enemy%20features.md#hitaction)), `inevent` is false (no EventDialogue is in progress), `doingaction` is false (we aren't in a [DoCommand](../DoCommand.md)) and the [message](../../SetText/Notable%20states.md#message) lock is released (covers cases like [Tattle](Action%20coroutines/Tattle.md)).

GetBlock mainly does 2 things: process the blocking storing the result in `blockcooldown` and `commandsuccess` and update the player party members's [animstate](../../Entities/EntityControl/Animations/animstate.md) to different stages of the blocking.

## How blocking works
A block can be attempted by pressing any of the following inputs (this is the -3 input which aggregates any of this list):

- 4 (Confirm)
- 5 (Cancel)
- 6 (Switch party)
- 7 (Toggle HUD)

However, the block will only be accepted if both `caninputcooldown` and `blockcooldown` expires. If this happens:

- `blockcooldown` is set to 20.0 frames (effectively starts at ~19.0 because GetBlock decreases it later by MainManager.`framestep`). This is the amount of frames left to have a block count by the damage pipeline should a [DoDamage](../Damage%20pipeline/DoDamage.md) call occurs. If it still has 16 or more frames left by the time DoDamage occurs, it counts as a super block for the next 3.0 frames (tracked by `superblockedthisframe`, set during the damage pipeline when processing the super block in [CalculateBaseDamage](../Damage%20pipeline/CalculateBaseDamage.md) and decreased by [LateUpdate](../Visual%20rendering/LateUpdate.md)).
- `caninputcooldown` is set to 37.0 frames. (effectively starts at ~36.0 because GetBlock decreases it later by MainManager.`framestep`) This is the amount of frames that must pass in order for a block to be allowed. The purpose of this is to prevent the player from spamming blocks because it effectively leaves ~17.0 frames of vulnerability after `blockcooldown` expires where the player will not be allowed to block, but if [DoDamage](../Damage%20pipeline/DoDamage.md) gets called during those frames, the block check will fail.

Another important field is updated no matter if a block is processed or not: `commandsuccess`. This field in a blocking context tells if `blockcooldown` is above 0.0 meaning it hasn't expired so it's a quick bool value that tells if the player is currently blocking. The purpose of this is to pass this value to [DoDamage](../Damage%20pipeline/DoDamage.md) as the block parameter so the damage pipeline can act accordingly of whether or not a block was successful. The damage pipeline needs to check `blockcooldown` to know if it was a super block or a regular one.

On each GetBlock (which is normally called on each Update during an enemy action), `caninputcooldown` and `blockcooldown` are decreased by MainManager.`framestep`. This has the side effect that the cooldown starting values are decreased by ~1.0 from the ones actually written to initially. It's not exactly 1.0 because it depends on the last MainManager.`framestep`. For example, if the last frame took longer than the target framerate, it will be decreased by slightly more than 1.0.

## A summarised explanation of blocking
All of the above can be summarised by the following:

- A block of any kind can be performed by pressing 4 (Confirm), 5 (Cancel), 6 (Switch party) or 7 (Toggle HUD) if it's been less than ~36.0 frames since the last time these inputs were pressed (tracked by `caninputcooldown`, decreased each GetBlock which is called on each Update)
- An accepted block only counts for up to ~19.0 frames leading the [DoDamage](../Damage%20pipeline/DoDamage.md) call (tracked by `blockcooldown`, decreased each GetBlock which is called on each Update). DoDamage receives a block parameter where it is intended that `commandsuccess` is passed to it which tells if `blockcooldown` is still active or not
- If `blockcooldown` reaches 0.0, no blocking can be performed for the next ~17.0 frames due to `caninputcooldown`. Blocking never influences the damage pipeline if it's performed after the DoDamage call
- If an accepted block reaches a [DoDamage](../Damage%20pipeline/DoDamage.md) while there's still 16.0 or more frames left on `blockcooldown`, the block counts as a super block. This effectively leaves only the first ~4.0 frames window of the block to count it as a super block
- If a super block occurs and is verified by the damage pipeline, it is valid for up to 3.0 frames after the damage pipeline completes where any further [DoDamage](../Damage%20pipeline/DoDamage.md) calls will count a super block no matter what `blockcooldown` and `commandsuccess` reports. This is done to allow quick damages in succession to still count the super blocks as it wouldn't be possible for the player to block due to the block spam prevention. This is tracked by `superblockedthisframe` which decreases on every [LateUpdate](../../Entities/EntityControl/Update%20process/Unity%20events/LateUpdate.md) by MainManager.`framestep`

## Details on why this blocking system works
It's important to understand why this system works because it involves the asynchronous nature of the battle system.

Let's take a typical attack action as an example. [DoAction](Action%20coroutines/DoAction.md) is an action coroutine. It means that it runs concurently to the main Update loop. Since GetBlock runs on Update and enemy actions must go through DoAction to call [DoDamage](../Damage%20pipeline/DoDamage.md) as part of their action, it means GetBlock and DoAction runs concurently from each other.

This allows GetBlock to not touch any of the damage pipeline, but still report and track whether the player is blocking and for how long they been blocking. Meanwhile, DoAction runs indepedently from this which allows very tight control on the animations and the moment to call DoDamage which can have an intuitive visual and auditive timing to assist the player in performing the block. It is free to use the results GetBlock gathered at any time because it is guaranteed to be correct since Update always run before DoAction gets any attention if it was set to resume on that frame.

This results in the blocking timing to work no matter how the enemy action is performed. There is no need to define any timing anywhere because it's implicitly handled by the asynchronous nature of the system. The one downside however is the entire timing window leads up to the actual damage frame meaning that intuitively, the player could be a frame after and still miss the block while they were within 4 frames of the damage. There is no coyote frames to account for this (adding any would introduce visual delays to present the damage counter and other effects).

## Known issues with [DoCommand](../DoCommand.md)
This system behaves unexpectedly when interacting with DoCommand. Check its documentation to learn more as it can end up splitting the block timing into 2 distrinct moments.

## Animation updates
Other than managing blocking, it also manages animations of the player party members. The following happens happens for every player party members whose battleentity.`overrideanim` is false:

- If the `hp` is 0 while their battleentity isn't `dead` yet, battleentity.[animstate](../../Entities/EntityControl/Animations/animstate.md) is set to 11 (`Hurt`)
- Otherwise, if [IsStopped](../Actors%20states/IsStopped.md) returns false on the player party member:
    - battleentity.[animstate](../../Entities/EntityControl/Animations/animstate.md) is set to 24 (`Block`) if all of the following are true:
        - `hp` is above 0
        - `playertargetID` is the player party member or -1 (the whole player party)
        - battleentity.[animstate](../../Entities/EntityControl/Animations/animstate.md) isn't 11 (`Hurt`)
        - `blockcooldown` is above 0.0, 
    - Otherwise, [UpdateAnim](../Visual%20rendering/UpdateAnim.md) is called with onlyplayer to true
- Otherwise, if the player party member `isasleep`, battleentity.[animstate](../../Entities/EntityControl/Animations/animstate.md) is set to 14 (`Sleep`)
- Otherwise, if the player party member `isnumb` or it has the [Freeze](../Actors%20states/BattleCondition/Freeze.md) condition, battleentity.[animstate](../../Entities/EntityControl/Animations/animstate.md) is set to 11 (`Hurt`)
