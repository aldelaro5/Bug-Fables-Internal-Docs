# Update
This Unity event is the main dispatcher of the battle flow. It only handles the synchronous high level flow that runs as long as control wasn't delegated to an action coroutine.

There are 3 possible flows to a BattleControl update: terminal, uncontrolled and controlled.

## Terminal flow
This flow happens if `cancelupdate` is true or instance.`pausemenu` exists. 

The former is only set to true when some kind of terminal coroutine occurs (such as a game over) where we know the battle will end in some ways (but it may be retried). The latter only happens after selecting the change loadout and retry option after a game over which brings up a cut down pause menu.

In this flow, absolutely no update logic occurs. The flow is completely controlled by whichever coroutine set `cancelupdate` or the PauseMenu. This flow is only entered in terminal cases where we know for sure the battle will end or retried.

It is also possible to enter this flow by setting instance.`inbattle` to false which happens when we know the battle will end without retry, but this alone won't technically be the same without setting `cancelupdate` to true. Doing this will still allow [RefreshEXP](../Visual%20rendering/RefreshEXP.md) calls that normally happens in the other 2 flows only. However, because they only happen if `oldexp` doesn't match `expreward`, it ends up not mattering because no EXP can be gained in any terminal cases (since the battle is already scheduled to end in some ways). It should be noted that the only terminal case where this happens while `cancelupdate` remains false is a sucessfull flee attempt.

## Uncontrolled flow
This flow happens if `action` or `inevent` is true which means that an action coroutine or an EventDialogue took control of the battle flow. This means that Update relegated its control and it features a very reduced logic compared to a controlled flow. It is meant to be entered temporarily where control will eventually be given back to a controlled flow with the exception of terminal cases which ends or retries the battle.

The main feature of this flow is less UI being shown as they are mostly disabled.

Here are all the logic included in this flow.

### RefreshEXP
If the `oldexp` value doesn't match the current `expreward`, [RefreshEXP](../Visual%20rendering/RefreshEXP.md) is called. This method manages the visual rendering of the currently cumulated EXP and the different orbs that composes it. `expreward` is the actual EXP cumulated so far while `oldexp` is the last value observed since the last RefreshEXP. It will also create `expholder` if it wasn't created already.

### HP UI refresh
[RefreshEnemyHP](../Visual%20rendering/RefreshEnemyHP.md) is called every 3 frames. It should be noted that in this flow, this will always results in the `hpbar` of the enemies to be disabled. This means this refresh is less useful here, but it's more useful in a controlled flow.

### UpdateSwitchIcon
UpdateSwitchIcon is called. It should be noted that in this flow, this will always results in the `switchicon` to be disabled. This means this refresh is less useful here, but it's more useful in a controlled flow.

### GetBlock
This logic is exclusive to an uncontrolled flow, but it only happens when `enemy` is true (we are in the player phase), `inevent` is false (no EventDialogue is in progress) and the [message](../../SetText/Notable%20states.md#message) lock is released (covers cases like tattling).

TODO

## Controlled flow
This flow happens if the other 2 didn't making it the default battle flow. It features complete control of the system by the Update event and features the full logic of the battle system. Eventually, this flow will delegate control to an action coroutine which will make it uncontrolled while the coroutine handles whatever needs to be done.

As part of this flow, everything from the unctrolled flow except GetBlock is included with more logic.

### Pre turn update
The following sections covers everything that happens before an actual turn is processed. This means everything here gets processed regardless of where in the turn we are in.

#### RefreshEXP
If the `oldexp` value doesn't match the current `expreward`, [RefreshEXP](../Visual%20rendering/RefreshEXP.md) is called. This method manages the visual rendering of the currently cumulated EXP and the different orbs that composes it. `expreward` is the actual EXP cumulated so far while `oldexp` is the last value observed since the last RefreshEXP. It will also create `expholder` if it wasn't created already.

#### HP UI refresh
[RefreshEnemyHP](../Visual%20rendering/RefreshEnemyHP.md) is called every 3 frames, but this time, it won't always disable the enemies's `hpbar`.

#### UpdateSwitchIcon
UpdateSwitchIcon is called, but this time, it won't always disable the `switchicon`. It only applies if the `switchicon` exists.

The `switchicon` is enabled when all of the following conditions are true (on top of being in this battle flow, it is disabled otherwise):

- There are more than 1 `availableplayers`
- `enemy` is false (we are during the player phase of the turn)
- `inevent` is false (we aren't in an EventDialogue)
- The [message](../../SetText/Notable%20states.md) lock is released (covers cases like tattling)
- `currentaction` is `BaseAction` (the main vine action menu)

If it gets disabled, it is placed offscreen by setting the local position to (0.0, 99.0, 0.0). 

If it gets enabled instead, the local y and z position are set to 4.25 and 5.0 respectively, For the x, it's -7.0 + 3.625 * the length of `playerdata`, but if `longcancel` is true, 0.3 is added to this.

#### CheckEvent
This method will check for any special conditions happening and if one occurs, it will result in some logic, frequently involving an EventDialogue starting.

Nothing happens here if `enemydata` is empty.

Here is a list of the conditions and what happens with them (only the first one takes effect):

- `calleventnext` is not negative: EventDialogue is called with the value of `calleventnext` followed by the value being set to -1. This is only used for the `eventonfall` feature of enemies
- [flags](../../Flags%20arrays/flags.md) 15 is false (combat tutorial hasn't been given yet): multiplie events can occur with an additional condition:
    - [flagvar](../../Flags%20arrays/flagvar.md) 11 is 0: EventDialogue 0 is started
    - [flagvar](../../Flags%20arrays/flagvar.md) 11 is 2: EventDialogue 2 is started
    - [flagvar](../../Flags%20arrays/flagvar.md) 11 is 3 and `turns` is 3: ExitBattle is called which start a ReturnToOverworld without flee, set `cancelupdate` and `action` to true and `enemy` to false which also changes the flow to a terminal one
- The first `enemydata` has an `animid` (an [enemy](../../Enums%20and%20IDs/Enemies.md) id) of `Spuder` and [flag](../../Flags%20arrays/flags.md) 27 is false (haven't gotten past the cutscene of meeting Leif yet), some logic occurs:
    - The `Spuder` enemy gets their `hp` set to 999 and `def` to 99
    - If the `Spuder` enemy didn't had a weakness of `AntiPierce`, it is added to `weakness`
    - If `turns` is 2 and [flagvar](../../Flags%20arrays/flagvar.md) 11 is 2, EventDialogue 4 is started
    - Otherwise, if `turns` is 3 and [flagvar](../../Flags%20arrays/flagvar.md) 11 is 1, ExitBattle is called which start a ReturnToOverworld without flee, set `cancelupdate` and `action` to true and `enemy` to false which also changes the flow to a terminal one
    - Otherwise, if `turns` % 2 is 0 while being above 0 and [flagvar](../../Flags%20arrays/flagvar.md) 11 is 2, EventDialogue 5 is started
- [flag](../../Flags%20arrays/flags.md) 16 is true (Leif can fight in battle) and [flag](../../Flags%20arrays/flags.md) 27 is false (haven't received the Relay tutorial yet), EventDialogue 3 is started
- [flag](../../Flags%20arrays/flags.md) 37 is false (using the B.O.S.S system in EX mode), [flagvar](../../Flags%20arrays/flagvar.md) 11 is below 200, [flag](../../Flags%20arrays/flags.md) 27 is true (got past the cutscene of meeting Leif for the first time), the first `enemydata` has an `animid` (an [enemy](../../Enums%20and%20IDs/Enemies.md) id) of `Spuder` with an `hp` below half of its `maxhp` floored, but above 0 and [GetFreePlayerAmmount](../Actors%20states/GetFreePlayerAmmount.md) returns 0:
    - EventDialogue 7 is started
    - [flagvar](../../Flags%20arrays/flagvar.md) 11 is set to 200
- [flag](../../Flags%20arrays/flags.md) 37 is false (using the B.O.S.S system in EX mode), [flagvar](../../Flags%20arrays/flagvar.md) 11 is below 150, [flag](../../Flags%20arrays/flags.md) 27 is true (got past the cutscene of meeting Leif for the first time), the first `enemydata` has an `animid` (an [enemy](../../Enums%20and%20IDs/Enemies.md) id) of `Spuder` and `turns` is 0:
    - EventDialogue 8 is started
    - [flagvar](../../Flags%20arrays/flagvar.md) 11 is set to 150

#### EXP counter updates
The first time this section run, `hexpcounter` wouldn't exist yet so it will get created by doing the following:

- `hexpcounter` is set to a new UI object named `expcounter` with tag `DelAftBtl` childed to the `GUICamera` with position (7.4, -6.5, 10.0) with size (0.5, 0.6, 1.0) using `guisprites[4]` (a HUD backgrond strip) and sortingOrder of 10
- The color of the SpriteRenderer of `hexpcounter` is set to instance.`charcolor[0]` (hardcoded to be pure yellow)
- A new UI object is created called `icon` childed to `hexpcounter` with position (-2.5, 0.0, 0.0) with size (1.25, 1.0, 1.0) using `guisprites[27]` (the EXP icon) and sortingOrder being 1 over the `hexpcounter`'s (11)
- [SetText](../../SetText/SetText.md) is called in [non dialogue mode](../../SetText/Dialogue%20mode.md#non-dialogue-mode):
    - text: `|`[font](../../SetText/Individual%20commands/Font.md)`,2||`[sort](../../SetText/Individual%20commands/Sort.md)`,20||`[color](../../SetText/Individual%20commands/Color.md)`,4|` + instance.`partyexp` padded with 2 `0` + `/|`[size](../../SetText/Individual%20commands/size.md)`,0.8|` + instance.`neededexp`
    - [fonttype](../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
    - No linebreak
    - No tridimensional
    - position: (-1.2, -0.3, 0.0)
    - No cameraoffset
    - size: Vector3.one
    - parent: `hexpcounter`
    - no caller
- The second child of `hexpcounter` (the SetText holder object from the call above) scale is set to (1.75, 1.75, 1.0)

From there, this section is done, but the logic will change on further updates since `hexpcounter` now exists.

Basically, the game will manage the `idletimer` field which will control the positioning of the `hexpcounter` and the `expholder`. The `idletimer` is reset to 0 constantly until the `currentaction` becomes `BaseAction` (the main vine menu). When that happens, it will get incremented once on this cycle until it reaches 250.

What will happen from there is the `hexpcounter` local posiiton will be set to a lerp from the existing one to (7.4, -6.5, 10.0) if the `idletimer` hasn't reached 200 or to (7.4, -4.4, 10.0) if it reached it. All with a factor of a 1/10 of the game's frametime in either case. This essentially will hide the counter offscreen at the bottom until at least 200 frames passed when the player was naviguating the main vine action menu where it will smoothly reveal itself.

If the `expholder` still exists, its x local position gets set to 0.0 if `idletimer` hasn't reached 200 yet or to -3.25 if it did. This moves the orbs to the left when the `hexpcounter` is shown so they don't overlap.

#### Enemies `hitaction`
`hitaction` is a field on `BattleData` that tells if an enemy wants to performa an action immediately on the next controlled update. These actions are performed out of the main turn flow because they are ran during the player phase. It's essentially a way for an enemy to temporarilly seize control of the turn flow to perform their action. This section handles these.

All `enemydata` elements are checked if any has `hitaction` set to true. If none do, nothing happens. For each that does have it set to true, the following happens:

- If the enemy [IsStopped](../Actors%20states/IsStopped.md) returns true, their `hitaction` gets set to false and nothing happens as it gets skipped
- Otherwise, `enemy` is set to true and a [DoAction](Action%20coroutines/DoAction.md) call occur with the battleentity with actionid being the `enemydata` index. This will also end the update cycle

Setting `enemy` to true here is temporary: it will go back to false as part of DoAction. DoAction is also responsible for setting the enemy's `hitaction` back to false. The overall effect is DoAction temporarily seize control of the battle flow, but for the game, it's as if we were in an enemy phase. The flow will go back to where it was in the player phase once it's handled. Due to `hitaction` being true, DoAction is able to handle this special case in a separate fashion.

This cycle repeats for all applicable enemies untill all `hitaction` are set to false at which point, the main turn procedure can continue.

### GUI cooldown
This only decreases `guicooldown` by the game's frametime if it hasn't expired yet. However, this cooldown isn't actually used anywhere making this dead code.

### Player phase
This is where the player party portion of the turn is handled. It is denoted by `enemy` being false and once the player phase is over, `enemy` is set to true which moves the battle into enemies phase.

The following always happen first:

- [UpdateAnim](../Visual%20rendering/UpdateAnim.md) is called
- `blockcooldown` is set to 0.0

The phase only proceeds if no `enemydata` battleentity is in a `forcemove` (returned by EnemiesAreNotMoving).

From there, what happens in the phase depends on `currentturn` and `avaliableplayers`. The former manages the current player selected for the current action while the latters tells if players are still free.

`currentturn` being positive, but below the length of `playerdata` means that we already are selecting a player for the turn. In that cause some logic happens:

- If instance.`hud` exists while not being empty, GUIMovement is called. This method will visually render the `fronticon` on the applicable `playerdata` element (`partypointer[0]`) on its lower right corner and also move the currently selected player's HUD element by setting its y local position to 0.0 - the abosolute value of Sin(Time.time / 2.0) / 7.0 (the other HUD elements have it reset to 0.0).
- [PlayerTurn](PlayerTurn.md) is called

If `currentturn` is -1 however, it means no one is selected yet which indicates the phase needs to proceed to either select one or perform a post player actions step. Here is the procedure in that case:

- `availableplayers` is updated to [GetFreePlayerAmmount](../Actors%20states/GetFreePlayerAmmount.md)
- If it's above 0 (we need to process a player selection):
    - The player index selected for `currentturn` value is found by searching in `partypointer` order and finding the first that isn't [IsStopped](../Actors%20states/IsStopped.md), its `cantmove` is 0 or below (it has at least one action available) and it is cleared by CheckFreePlayers which means it isn't in `lastturns` when more than 1 player is free (it means it's the next player in the select cycle or it's the last free player)
    - [UpdateText](../Visual%20rendering/UpdateText.md) is called
    - If `playerdata` length is more than 1, AddLastTurn is called on the player index (by `partypointer`) which adds the player index to the next free slot or pushes it to the last one if all slots are taken, removing the oldest one (it just advances the select cycle)
    - [PlayerTurn](PlayerTurn.md) is called
    - This cycle ends. From now on, all cycles will see the `currentturn` being already set to a player and just process it
- Otherwise, if `chompy` exists, `chompyattacked` / `chompylock` are false and `enemydata` is not empty (chompy needs to do her action):
    - If `spitout` is in progress, `chompyattacked` is set to true which skips the chompy action
    - Otherwise, if `chompyattack` isn't in progress, it is set to a new [Chompy](Action%20coroutines/Chompy.md) action coroutine starting
- Otherwise, if `aiparty` exists, `aiattacked` is false and `enemydata` isn't empty (the AI needs to do their action), an [AIAttack](Action%20coroutines/AIAttack.md) action coroutine is started

### Enemies phase
This is where the enemies party portion of the turn is handled. It is denoted by `enemy` being true while there are at least 1 free enemy obtained via GetFreeEnemies. A free enemy is one whose `cantmove` is 0 or below (it has at least one action available) and whose `hp` is above 0 (not dead yet). If there are no free enemies, this phase is skipped and we move on to the turn end phase.

If we somehow got here while GetAlivePlayerAmmount returns 0 (all `playerdata`'s `hp` is 0 or below or their `eatenby` isn't null which means they got eaten), this phase will end abruptly by doing the following:

- `mainturn` is set to an [AdvanceMainTurn](Action%20coroutines/AdvanceMainTurn.md) call if it wasn't in progress already
- `cancelupdate` is set to true which changes the flow to a terminal one

If at least one player is alive, then the first enemy with a `cantmove` of 0 or below is found. In order for the enemy to get their action, they need to not be considered stopped which uses the same standard than [IsStopped](../Actors%20states/IsStopped.md) with `actimmobile` check, but with one difference: if the enemy has the `Flipped` [condition](../Actors%20states/Conditions.md), it isn't considered stopped. This is because the check involves IsStoppedLite which is the same than IsStopped without the `Flipped` condition and the `actimmobile` checks, but it also combines this with an `actimmobile` check so the overall effect is only the `Flipped` condition check is ommited. 

If the game finds it's stopped, their `cantmove` is set to 1 and the next enemy is checked instead.

If the enemy isn't stopped, [DoAction](Action%20coroutines/DoAction.md) is called on the battleentity with actionid of the `enemydata` index. At most, this can happen only once and the next enemy in line will need to wait the current one's action is done.

This process is repeated for each enemy until all of them have a `cantmove` above 0 meaning no actions are available on any enemies and the game is ready to proceed to the turn end phase.

### Turn end phase
After both parties performed their turn, an [AdvanceMainTurn](Action%20coroutines/AdvanceMainTurn.md) action coroutine is started and set to `mainturn` if it wasn't in progress already. This immediately move the flow back to uncontrolled while the coroutine handles the turn advancement.