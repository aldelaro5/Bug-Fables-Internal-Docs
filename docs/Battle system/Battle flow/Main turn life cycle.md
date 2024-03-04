# Main turn life cycle
This page describes the life cycle of a main turn which is processed as the last step of a [controlled update flow](Update%20flows/Controlled%20flow.md). It means the steps presented in this page are parts of [Update](Update.md).

It is composed of 3 phases, starting from the first and each subsequent one needing the previous to finish:

1. Player phase (`enemy` is false or became false from the last main turn's end of turn phase)
2. Enemy phase (`enemy` became true after the end of the player phase)
3. End of turn phase (`enemy` is still true while no free enemy party members are left)

This cycle repeats continuously until the flow changes to a [Terminal flow](Update%20flows/Terminal%20flow.md).

## Player phase
This is where the player party portion of the turn is handled. It is denoted by `enemy` being false and once the player phase is over, `enemy` is set to true which moves the battle into enemies phase.

The following always happen first:

- [UpdateAnim](../Visual%20rendering/UpdateAnim.md) is called
- `blockcooldown` is set to 0.0

The phase only proceeds if no `enemydata` battleentity is in a `forcemove` (returned by EnemiesAreNotMoving).

From there, what happens in the phase depends on `currentturn` and `avaliableplayers`. The former manages the current player party member selected for the current action while the latter tells if player party members are still free.

`currentturn` being positive, but below the length of `playerdata` means that we already are selecting a player party member for the turn. In that case some logic happens:

- If instance.`hud` exists while not being empty, GUIMovement is called. This method will visually render the `fronticon` on the applicable `playerdata` element (`partypointer[0]`) on its lower right corner and also move the currently selected player's HUD element by setting its y local position to 0.0 - the abosolute value of Sin(Time.time / 2.0) / 7.0 (the other HUD elements have it reset to 0.0) This makes the HUD element move up and down.
- [PlayerTurn](PlayerTurn.md) is called

If `currentturn` is -1 however, it means no one is selected yet which indicates the phase needs to proceed to either select one or perform a post player action step. This starts by updating `availableplayers` to [GetFreePlayerAmmount](../Actors%20states/Player%20party%20members/GetFreePlayerAmmount.md).

From there, the next appropriate part of the player phase is performed which is the first of the next 4 parts. They can be thought of a sequence where the first applicable part occurs. A part may be skipped entirely, but eventually, the last one will occur which will end the player phase.

Here are the player phase parts and their conditions:

1. Player party member selection (`availableplayers` is above 0 meaning at least one player party member is free)
2. `chompy` action (no player party members are free, `chompy` exists, can act and haven't acted in this main turn)
3. `aiparty` action (no player party members are free, `aiparty` exists and haven't acted yet in this main turn)
4. End of the player phase (none of the above applied meaning the player phase is done)

### Player party member selection
This part only happens if `availableplayers` is above 0.

This is where the player party member selection happens:

- The player party member index selected for `currentturn` value is found by searching in `partypointer` order and finding the first that isn't [IsStopped](../Actors%20states/IsStopped.md), its `cantmove` is 0 or below (it has at least one action available) and it is cleared by CheckFreePlayers which means it isn't in `lastturns` when more than 1 player is free (it means it's the next player in the select cycle or it's the last free player)
- [UpdateText](../Visual%20rendering/UpdateText.md) is called
- If `playerdata` length is more than 1, AddLastTurn is called on the player index (by `partypointer`) which adds the player index to the next free slot or pushes it to the last one if all slots are taken, removing the oldest one (it just advances the select cycle)
- [PlayerTurn](PlayerTurn.md) is called

This cycle then ends. From now on, all cycles will see the `currentturn` being already set to a player and just process it

### `chompy` action
This part happens if all of the following are fufilled:

- `chompy` exists
- `chompyattacked` is false (this part hasn't happened yet)
- `chompylock` is false
- `enemydata` is not empty

This is what happens when the above are fufilled (otherwise, this part is skipped):

- If `spitout` is in progress, `chompyattacked` is set to true which skips the chompy action
- Otherwise, if `chompyattack` isn't in progress, it is set to a new [Chompy](Action%20coroutines/Chompy.md) action coroutine starting

### `aiparty` action
This part happens if all of the following are fufilled:

- `aiparty` exists
- `aiattacked` is false (this part hasn't happened yet)
- `enemydata` is not empty

The only thing that happens here is an [AIAttack](Action%20coroutines/AIAttack.md) action coroutine is started.

### End of the player phase
If none of the possible parts applied above, the following happens which ultimately ends the player phase:

- `enemy` is set to true
- All of the instance.`hud`'s first child has their local position reset to Vector3.zero
- `currentturn` is set to the length of `playerdata`

## Enemy phase
This is where the enemies party portion of the turn is handled. It is denoted by `enemy` being true while there are at least 1 free enemy obtained via GetFreeEnemies. 

A free enemy is one whose `cantmove` is 0 or below (it has at least one action available) and whose `hp` is above 0 (not dead yet). If there are no free enemies, this phase is skipped and we move on to the turn end phase.

If we somehow got here while GetAlivePlayerAmmount returns 0 (all `playerdata`'s `hp` is 0 or below or their `eatenby` isn't null which means they got eaten), this phase will end abruptly by doing the following:

- `mainturn` is set to an [AdvanceMainTurn](Action%20coroutines/AdvanceMainTurn.md) call if it wasn't in progress already
- `cancelupdate` is set to true which changes the flow to a [terminal flow](Update%20flows/Terminal%20flow.md)

If at least one player is alive, then the first enemy with a `cantmove` of 0 or below is found. In order for the enemy to get their action, they need to not be considered IsStoppedLite which uses the same standard than [IsStopped](../Actors%20states/IsStopped.md) with `actimmobile` check, but with one difference: if the enemy has the `Flipped` [condition](../Actors%20states/Conditions.md), it isn't considered stopped.

If the game finds it's stopped, their `cantmove` is set to 1 and the next enemy is checked instead. Effectively, this simply readjusts their actor turn counter because it isn't exhausted like it normally would have been in [EndEnemyTurn](EndEnemyTurn.md).

If the enemy isn't stopped, [DoAction](Action%20coroutines/DoAction.md) is called on the battleentity with actionid of the `enemydata` index. This changes to an [uncontrolled flow](Update%20flows/Uncontrolled%20flow.md). At most, this can happen only once and the next enemy in line will need to wait the current one's action is done.

This process is repeated for each enemy until all of them have a `cantmove` above 0 meaning no actions are available on any enemies and the game is ready to proceed to the turn end phase.

## Turn end phase
After both parties performed their phases, an [AdvanceMainTurn](Action%20coroutines/AdvanceMainTurn.md) action coroutine is started and set to `mainturn` if it wasn't in progress already. This immediately move the flow back to [uncontrolled flow](Update%20flows/Uncontrolled%20flow.md) while the coroutine handles the turn advancement.