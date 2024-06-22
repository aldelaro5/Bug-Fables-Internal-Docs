# Controlled flow
This is one of the flow of [Update](../Update.md) which is the default flow that hapens when the other aren't happening where Update has complete control over the battle and acts as a dispatcher.

It features complete control of the system by the Update event and features the full logic of the battle system. Eventually, this flow will delegate control to an action coroutine which will change to an [uncontrolled flow](Uncontrolled%20flow.md) while the coroutine handles whatever needs to be done.

As part of this flow, everything from the uncontrolled flow except [GetBlock](../GetBlock.md) is included with more logic.

## Pre turn update
The following sections covers everything that happens before an actual main turn is processed. This means everything here gets processed regardless of where in the turn we are in.

### RefreshEXP
If the `oldexp` value doesn't match the current `expreward`, [RefreshEXP](../../Visual%20rendering/RefreshEXP.md) is called. This method manages the visual rendering of the currently cumulated EXP and the different orbs that composes it. `expreward` is the actual EXP cumulated so far while `oldexp` is the last value observed since the last RefreshEXP. It will also create `expholder` if it wasn't created already.

### HP UI refresh
[RefreshEnemyHP](../../Visual%20rendering/RefreshEnemyHP.md) is called every 3 frames.

### UpdateSwitchIcon
UpdateSwitchIcon is called. It only applies if the `switchicon` exists.

The `switchicon` is enabled when all of the following conditions are true (on top of being in this battle flow, it is disabled otherwise):

- There are more than 1 `availableplayers`
- `enemy` is false (we are during the player phase of the turn)
- `inevent` is false (we aren't in an EventDialogue)
- The [message](../../../SetText/Notable%20states.md) lock is released (covers cases like tattling)
- `currentaction` is `BaseAction` (the main vine action menu)

If it gets disabled, it is placed offscreen by setting the local position to (0.0, 99.0, 0.0).

If it gets enabled instead, the local y and z position are set to 4.25 and 5.0 respectively, For the x, it's -7.0 + 3.625 * the length of `playerdata`, but if `longcancel` is true, 0.3 is added to this.

### CheckEvent
This method will check for any special conditions happening and if one occurs, it will result in some logic, frequently involving an EventDialogue starting.

Nothing happens here if `enemydata` is empty.

Here is a list of the conditions and what happens with them (only the first one takes effect):

- `calleventnext` is not negative: [EventDialogue](../EventDialogue.md) is called with the value of `calleventnext` followed by the value being set to -1. This is only used for the [eventonfall](../../Actors%20states/Enemy%20features.md#eventonfall) feature of enemies
- [flags](../../../Flags%20arrays/flags.md) 15 is false (combat tutorial hasn't been given yet): multiplie events can occur with an additional condition:
    - [flagvar](../../../Flags%20arrays/flagvar.md) 11 is 0: [EventDialogue](../EventDialogue.md) 0 is started
    - [flagvar](../../../Flags%20arrays/flagvar.md) 11 is 2: [EventDialogue](../EventDialogue.md) 2 is started
    - [flagvar](../../../Flags%20arrays/flagvar.md) 11 is 3 and `turns` is 3: ExitBattle is called which start a [ReturnToOverworld](../Terminal%20coroutines/ReturnToOverworld.md) without flee, set `cancelupdate` and `action` to true and `enemy` to false which also changes the flow to a [terminal](Terminal%20flow.md) one
- The first `enemydata` has an `animid` (an [enemy](../../../Enums%20and%20IDs/Enemies.md) id) of `Spuder` and [flag](../../../Flags%20arrays/flags.md) 27 is false (haven't gotten past the cutscene of meeting Leif yet), some logic occurs:
    - The `Spuder` enemy gets their `hp` set to 999 and `def` to 99
    - If the `Spuder` enemy didn't had a weakness of `AntiPierce`, it is added to [weakness](../../Actors%20states/Enemy%20features.md#weakness)
    - If `turns` is 2 and [flagvar](../../../Flags%20arrays/flagvar.md) 11 is 0, [EventDialogue](../EventDialogue.md) 4 is started
    - Otherwise, if `turns` is 3 and [flagvar](../../../Flags%20arrays/flagvar.md) 11 is 1, [ExitBattle](../Terminal%20wrappers/ExitBattle.md) is called which will change to a [terminal flow](Terminal%20flow.md)
    - Otherwise, if `turns` % 2 is 0 while being above 0 and [flagvar](../../../Flags%20arrays/flagvar.md) 11 is 2, [EventDialogue](../EventDialogue.md) 5 is started
- [flag](../../../Flags%20arrays/flags.md) 16 is true (Leif can fight in battle) and [flag](../../../Flags%20arrays/flags.md) 24 is false (haven't received the Relay tutorial yet), [EventDialogue](../EventDialogue.md) 3 is started
- [EventDialogue](../EventDialogue.md) 7 is started followed by [flagvar](../../../Flags%20arrays/flagvar.md) 11 being set to 200 when all of the following are true:
    - [flag](../../../Flags%20arrays/flags.md) 37 is false (using the B.O.S.S system in EX mode)
    - [flagvar](../../../Flags%20arrays/flagvar.md) 11 is below 200, [flag](../../../Flags%20arrays/flags.md) 27 is true (got past the cutscene of meeting Leif for the first time)
    - The first `enemydata` has an `animid` (an [enemy](../../../Enums%20and%20IDs/Enemies.md) id) of `Spuder` with an `hp` below half of its `maxhp` floored, but above 0
    - [GetFreePlayerAmmount](../../Actors%20states/Player%20party%20members/GetFreePlayerAmmount.md) returns 0
- [EventDialogue](../EventDialogue.md) 8 is started followed by [flagvar](../../../Flags%20arrays/flagvar.md) 11 being set to 150 when all of the following are true:
    - [flag](../../../Flags%20arrays/flags.md) 37 is false (using the B.O.S.S system in EX mode)
    - [flagvar](../../../Flags%20arrays/flagvar.md) 11 is below 150, [flag](../../../Flags%20arrays/flags.md) 27 is true (got past the cutscene of meeting Leif for the first time)
    - The first `enemydata` has an `animid` (an [enemy](../../../Enums%20and%20IDs/Enemies.md) id) of `Spuder`
    - `turns` is 0

### EXP counter updates
The first time this section run, `hexpcounter` wouldn't exist yet so it will get created by doing the following:

- `hexpcounter` is set to a new UI object named `expcounter` with tag `DelAftBtl` (destroyed on [ReturnToOverworld](../Terminal%20coroutines/ReturnToOverworld.md)) childed to the `GUICamera` with position (7.4, -6.5, 10.0) with size (0.5, 0.6, 1.0) using `guisprites[4]` (a HUD backgrond strip) and sortingOrder of 10
- The color of the SpriteRenderer of `hexpcounter` is set to instance.`charcolor[0]` (hardcoded to be pure yellow)
- A new UI object is created called `icon` childed to `hexpcounter` with position (-2.5, 0.0, 0.0) with size (1.25, 1.0, 1.0) using `guisprites[27]` (the EXP icon) and sortingOrder being 1 over the `hexpcounter`'s (11)
- [SetText](../../../SetText/SetText.md) is called in [non dialogue mode](../../../SetText/Dialogue%20mode.md#non-dialogue-mode):
    - text: `|`[font](../../../SetText/Individual%20commands/Font.md)`,2||`[sort](../../../SetText/Individual%20commands/Sort.md)`,20||`[color](../../../SetText/Individual%20commands/Color.md)`,4|` + instance.`partyexp` padded with 2 `0` + `/|`[size](../../../SetText/Individual%20commands/size.md)`,0.8|` + instance.`neededexp`
    - [fonttype](../../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
    - No linebreak
    - No tridimensional
    - position: (-1.2, -0.3, 0.0)
    - No cameraoffset
    - size: Vector3.one
    - parent: `hexpcounter`
    - no caller
- The second child of `hexpcounter` (the SetText holder object from the call above) scale is set to (1.75, 1.75, 1.0)

From there, this section is done, but the logic will change on further updates since `hexpcounter` now exists.

Basically, the game will manage the `idletimer` field which will control the positioning of the `hexpcounter` and the `expholder`. The `idletimer` is reset to 0 constantly until the [currentaction](../../Player%20UI/Pick.md) becomes `BaseAction` (the main vine menu). When that happens, it will get incremented once on this cycle until it reaches 250.

What will happen from there is the `hexpcounter` local posiiton will be set to a lerp from the existing one to (7.4, -6.5, 10.0) if the `idletimer` hasn't reached 200 or to (7.4, -4.4, 10.0) if it reached it. All with a factor of a 1/10 of the game's frametime in either case. This essentially will hide the counter offscreen at the bottom until at least 200 frames passed when the player was naviguating the main vine action menu where it will smoothly reveal itself.

If the `expholder` still exists, its x local position gets set to 0.0 if `idletimer` hasn't reached 200 yet or to -3.25 if it did. This moves the orbs to the left when the `hexpcounter` is shown so they don't overlap.

### Enemies [hitaction](../../Actors%20states/Enemy%20features.md#hitaction)
`hitaction` is a field on [BattleData](../../Actors%20states/BattleData.md) that tells if an enemy wants to perform an action immediately on the next controlled update. These actions are performed out of the main turn flow because they are ran during the player phase. It's essentially a way for an enemy to temporarilly seize control of the turn flow to perform their action. This section handles these.

All `enemydata` elements are checked if any has `hitaction` set to true. If none do, nothing happens. For each that does have it set to true, the following happens:

- If the enemy [IsStopped](../../Actors%20states/IsStopped.md) returns true (without [actimmobile](../../Actors%20states/Enemy%20features.md#actimmobile) check), their `hitaction` gets set to false and nothing happens as it gets skipped
- Otherwise, `enemy` is set to true and a [DoAction](../Action%20coroutines/DoAction.md) call occur with the battleentity with actionid being the `enemydata` index. This will also end the update cycle

For more information on how this works, check the [hitaction](../../Actors%20states/Enemy%20features.md#hitaction) documentation.

This cycle repeats for all applicable enemies untill all `hitaction` are set to false at which point, the main turn procedure can continue.

## GUI cooldown
This only decreases `guicooldown` by the game's frametime if it hasn't expired yet. However, this cooldown isn't actually used anywhere making this dead logic.

## Main turn procedure
This is where the main turn logic happens.

This procedure is documented in the [main turn life cyle](../Main%20turn%20life%20cycle.md) document.
