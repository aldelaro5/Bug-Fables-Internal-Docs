# `PepTalk`

This action features no action commands or damages logic. It only does the following:

- If the `target` player party member's `hp` is 0 or below, [RevivePlayer](../../Actors%20states/Player%20party%20members/RevivePlayer.md) is called on them leaving them with 4 `hp` with showcounter

## No charges usage
This action sets `dontusecharge` to true and as such, it will not consume any attacker's `charge` as it would normally.

## Logic sequence

- `dontusecharge` set to true
- Camera moves slowely to look at the `target` player party member
- attacker MoveTowards the `target` player party member slightly to the right of it and behind with a stopstate of 5 (`Angry`)
- Yield until attacker's `forcemove` is done
- Yield for a frame
- attacker FaceTowards the `target` player party member
- `Taunt2` sound plays on loop
- Yield for 0.75 seconds
- `Taunt2` sound stops playing
- If `target` player party member's `hp` is above 0 (this shouldn't happen under normal gameplay because the skill can normally only be used on dead player party members):
    - `target` player party member animstate is set to the return of AngryAnim which is a value that depends on the player [animid](../../../Enums%20and%20IDs/AnimIDs.md): 10 (`Flustered`) for `Bee`, 5 (`Angry`) for `Beetle` and 102 for `Moth`
    Yield for 0.75 seconds
- Otheriwse:
    - Yield until the `target` player party member's `deathcoroutine` is done
    - ShakeSprite called on `target` player party member with an x intensity of 0.1 for 40.0 of frametimer
    - Yield for 0.66 seconds
    - [RevivePlayer](../../Actors%20states/Player%20party%20members/RevivePlayer.md) is called on `target` player party member leaving them with 4 `hp` with showcounter
    - `target` player party member's `overridejump` set to false
    - [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on `target` player party member
    - Yield for 0.8 seconds
- [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) called
