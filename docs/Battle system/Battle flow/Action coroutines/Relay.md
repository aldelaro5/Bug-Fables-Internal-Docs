# Relay
This is an action coroutine that invokes the relay mechanic which allows a player party member to donate their actor turn to another. It also handles the animations and the `RelayTransfer` [medal](../../../Enums%20and%20IDs/Medal.md) effects when applicable.

The coroutine excepts to have the player party member whose index is `currentturn` to relay to the one whose index is `option`.

## Setup

- `action` is set to true changing to an [uncontrolled flow](../Update%20flows/Uncontrolled%20flow.md)
- [UpdateText](../../Visual%20rendering/UpdateText.md)
- A frame is yielded
- The `Relay` sound is played

## Relay animation

- Both the `currentturn` and `option`'s matching player party member's battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) are set to 4 (`ItemGet`) and their battleentity.`spin` to (0.0, -20.0, 0.0)
- 0.3 seconds are yielded
- During the course of 10.0 frames (tracked by a local counter advancing by 1/10 of the game's frametime starting at 0.0 reaching 1.0), both the `currentturn` and `option`'s matching player party member's battleentity.`spin` are lerped from (0.0, -20.0, 0.0) to Vector3.zero linearly over the course of those 10.0 frames
- Both the `currentturn` and `option`'s matching player party member's battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) are set to 0 (`Idle`) and their battleentity.`spin` are zeroed out
- `receivedrelay[option]` is set to true (this allows the `tiredpart` to be rendered on the player party member whenever their `tired` goes above 0 during [UpdateAnim](../../Visual%20rendering/UpdateAnim.md))
- A frame is yielded

## The actual relay

- The `currentturn`'s player party member's `haspassed` is set to true (indicating it already relayed disallowing to relay again)
- The `option`'s player party member's `cantmove` is decremented (this adds advances the actor turn, but it needs to be at 0 or above for an action to be possible still)

## `RelayTransfer` [medal](../../../Enums%20and%20IDs/Medal.md) effects
If that medal is equipped on the `currentturn`'s player party member's `trueid`, the effects will be performed (this section is skipped if it's not equipped):

Essentially, how it works is the `currentturn`'s player party member's [condition](../../Actors%20states/Conditions.md) are checked and all of the ones that aren't in the following list will be removed (by reseting the `condition` field to a new array without the ones to remove):

- `Topple`
- `Flipped`
- `Shield`
- `Eaten`
- `EventStop`
- `Reflection`

The conditions removed will be added to the `option`'s player party member via [SetCondition](../../Actors%20states/Conditions%20methods/SetCondition.md) using the same `BattleCondition` and actor turn counter. This does imply they can amend existing conditions.

If at least one condition is transfered, the `Charge10` sound will play when the first condition is transfered so it plays only once.

It also transferts `charge`: the `currentturn`'s player party member's `charge` is set to the existing one + the `option`'s clamped from 0 to 3 while the `option`'s player party member's `charge` gets set to 0.

The logic ends by calling [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) followed by UpdateConditionIcons being called which calls UpdateConditionBubbles on all battleentity (all `playerdata` with right to false and all `enemydata` with `hp` above 0 with right to true).

## Finalising the relay

- [EndPlayerTurn](../EndPlayerTurn.md) is called which advances the actor turn
- [CancelList](../../Player%20UI/CancelList.md) is called
- A frame is yielded
- `action` is set to false changing to a [controlled flow](../Update%20flows/Controlled%20flow.md)
- [UpdateText](../../Visual%20rendering/UpdateText.md) is called
