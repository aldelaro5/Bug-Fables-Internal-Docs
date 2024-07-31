# [Spuder](../../Enemy%20actions/Enemies/Spuder.md) EventDialogues
`Spuder` has very complex action logic involving [EventDialgoue](../EventDialogue.md) because there are 3 different encounters behavior for them that greatly changes their action logic:

1. The first encounter happens when [flags](../../../Flags%20arrays/flags.md) 27 is false (not yet met with `Moth`) and [flagvar](../../../Flags%20arrays/flagvar.md) 11 is 0
2. The second encounter happens when [flags](../../../Flags%20arrays/flags.md) 27 is false (not yet met with `Moth`) and [flagvar](../../../Flags%20arrays/flagvar.md) 11 is 2
3. The third encounter happens when [flags](../../../Flags%20arrays/flags.md) 27 is true (met with `Moth`)

These encounters involve different EventDialogue and flows. The first 2 encounters logic differences will be documented here as they are more scripted alongside. For the third encounter, most of the logic is documented in the [Spuder action](../../Enemy%20actions/Enemies/Spuder.md) page, but it involves 2 EventDialogues that will be documented here since they still affect the logic greatly.

These EventDialogue frequently makes use of [CheckEvent](../Update%20flows/Controlled%20flow.md#checkevent) to guide the flow. They also makes use of `flagvar` 11 to track the state of the encounters accross them.

## First encounter
This is the first encounter that requires [flags](../../../Flags%20arrays/flags.md) 27 to be false (not yet met with `Moth`) and [flagvar](../../../Flags%20arrays/flagvar.md) 11 to be 0. It's assumed here that the player party is only composed of `Beetle`.

CheckEvent will detect these conditions when the first `enemydata` is `Spuder` and make it effectively unbeatable by setting its `hp` to 999 and its `def` to 99 as well as add an `AntiPierce` [weakness](../../Actors%20states/Enemy%20features.md#weakness) if it wasn't present which cancels out any `Pierce` attacks. Even if the player can defeat them, the event will proceed as normal.

From there, the battle goes on as normal, but the `Spuder` action receives some heavy restrictions due to the same conditions above:

- `Spuder` will always decide to use his bite attack move if their [position](../../Actors%20states/BattlePosition.md) is `Ground`
- Due to the above, `Spuder` will never attempt to rise in the air to `Flying`, but an exception to this occurs where it will always do it if `demomode` is true or `tempdata` is 1 (the latter is assumed to never be the case for this encounter)
- `Spuder` will never drop down to `Ground` if it has risen in the air to `Flying` even for more than 2 main turns

Effectively, until `demomode` is set to true, `Spuder` is stuck using his biting attack move.

This continues until `turns` is 2 meaning after 2 main turns, CheckEvent detects this and starts EventDialogue 4.

### EventDialogue 4
This EventDialogue represents the dialogue and scripted action of `Spuder` rising in the air:

- [SetText](../../../SetText/SetText.md) is called in [dialogue](../../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
    - text: `commondialogue[27]` which is some dialogue about the fight
    - [fonttype](../../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
    - linebreak: `messagebreak`
    - tridimensional: false
    - position: Vector3.zero
    - size: Vector3.one
    - parent: `playerdata[0]`
    - caller: null
- Yield all frames until the [message](../../../SetText/Notable%20states.md#message) lock is released
- instance.`camtargetpos` is set to `enemydata[0]`'s position (should always be `Spuder`) with an instance.`camspeed` set to 0.03
- Yield for 0.5 seconds
- `demomode` is set to true
- [DoAction](../Action%20coroutines/DoAction.md) is called with `enemydata[0]` as the entity and 0 as the actionid. Effectively, it forces `Spuder` to do an actor turn, but because `demomode` is true, he always will rise in the air to `Flying` in his action
- Yield all frames until `action` goes to false (until DoAction is done)
- `demomode` is set back to false
- [SetText](../../../SetText/SetText.md) is called in [dialogue](../../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
    - text: `commondialogue[28]` which is some dialogue about the action `Spuder` did
    - [fonttype](../../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
    - linebreak: `messagebreak`
    - tridimensional: false
    - position: Vector3.zero
    - size: Vector3.one
    - parent: `playerdata[0]` (should be `Beetle`)
    - caller: null
- Yield all frames until the [message](../../../SetText/Notable%20states.md#message) lock is released
- `enemydata[0]` (should always be `Spuder`) has its `cantmove` set to 0 which allows them to act for this main turn since it technically hasn't occured yet (the game forced them to act so this is needed to resync the `cantmove` counter to where it would be)
- `flagvar` 11 is set to 1 which advances to the next phase of the encounter

### Encounter end
Since `flagvar` is 11, this now means that CheckEvent will call [ExitBattle](../Terminal%20wrappers/ExitBattle.md) once `turns` is 3 meaning a full main turn passed since EventDialogue 4. This ends the first encounter.

## Second encounter
The second encounter is similar to the first, but with some key differences with the most notable one being it is assumed a `MothWeb` is present alongside `Spuder` as `enemydata[1]`. `MothWeb` is a special enemy because it has no action logic, but it is assumed to have an [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath) set to EventDialogue 6. Triggering this EventDialogue will ultimately end the battle and it is the overall goal of the second encounter. It is also assumed that the player party is composed of `Bee` and `Beetle` in default party order.

`flagvar` 11 must be 2 and `tempdata` must be 1. This is normally set by the event that started the battle. CheckEvent detects this which causes to trigger EventDialogue 5 when `turns` is 2 meaning 2 main turns passed. This EventDialogue is a reminder to defeat `MothWeb` via some dialogues.

This goes on infinetely because `Spuder`'s `hp` starts at 999 and `def` starts at 99. He is effectively not beatable (even if defeated, the event will proceed as normal). One thing does changes with its action logic compared to their first encounter: `tempdata` should be 1 at this point as it was set by the event that started this encounter.

This means that he will always rise in the air to `Flying` on their first main turn. He still will not drop down to `Ground` on his own even after 2 main turns, but if they fall as a result of a [DoDamage](../../Damage%20pipeline/DoDamage.md#dodamage), their move selection changes to be a 50/50 between their bite attack and rising in the air. This is because at the end of the air rising move, `tempdata` is set to 0 if it was 1 so it won't get overriden again.

### EventDialogue 5
This EventDialogue is some dialogues that reminds the player to defeat `MothWeb` after 2 main turns have passed:

- If `playerdata[1]` (should always be `Beetle`)'s `hp` is above 0:
    - [SetText](../../../SetText/SetText.md) is called in [dialogue](../../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
        - text: `commondialogue[29]` which is some dialogue to remind to defeat `MothWeb`
        - [fonttype](../../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
        - linebreak: `messagebreak`
        - tridimensional: false
        - position: Vector3.zero
        - size: Vector3.one
        - parent: `playerdata[1]` (should always be `Beetle`)
        - caller: null
    - Yield all frames until the [message](../../../SetText/Notable%20states.md#message) lock is released 
- `flagvar` 11 is set to 3 which prevents this EventDialogue from triggering again

### EventDialogue 6
This EventDialogue is assumed to be the [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath) of `MothWeb` and marks the end of the second encounter as it ultimately leads to the end of the battle. It is assumed that `enemydata[1]` is `MothWeb`:

- `MothWeb` animstate set to 11 (`Hurt`)
- `MothWeb`'s [model](../../../Entities/EntityControl/Notable%20methods/AddModel.md#addmodel) has all ParticlesSystem starting playing
- All `playerdata` has their animstate set to 13 (`BattleIdle`)
- `Spin2` sound played on loop using `sounds[9]` with pitch 0.7 and volume 0.9
- Yield for 1 second
- `sounds[9]` stopped
- `Spin4` sound plays with 1.1 volume
- A new entity is created via [CreateNewEntity](../../../Entities/EntityControl/EntityControl%20Creation.md#createnewentity) named `tmoth` with animid of 2 (`Moth`) at `playerdata[1]` position + (0.0, 3.5, 0.0) childed to the `battlemap`
- `tmoth` animstate set to 11 (`Hurt`)
- `tmoth` y spin set to -20.0
- `tomoth`'s `onground` set to false
- [CreateFeet](../../../Entities/EntityControl/EntityControl%20Methods.md#createfeet) called on `tmoth`
- `enemydata[1]` (should always be `MothWeb`)'s position is set offscreen at (0.0, -1000.0, 0.0)
- Yield for a frame
- `tmoth`'s `overridejump` set to true
- `tmoth` animstate set to 11 (`Hurt`)
- Yield all frames until `tmoth`'s `onground` gets to true. Before each yield, their `overridejump` is set to true and their animstate is set to 11 (`Hurt`)
- `tmoth`'s `spin` is zeroed out
- `tmoth` animstate set to 18 (`KO`)
- Yield for 0.5 seconds
- [SetText](../../../SetText/SetText.md) is called in [dialogue](../../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
    - text: `commondialogue[30]` which is some dialogue after defeating `MothWeb`
    - [fonttype](../../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
    - linebreak: `messagebreak`
    - tridimensional: false
    - position: Vector3.zero
    - size: Vector3.one
    - parent: `playerdata[0]` (should always be `Bee`)
    - caller: null
- Yield all frames until the [message](../../../SetText/Notable%20states.md#message) lock is released
- Yield for 0.25 seconds
- [ExitBattle](../Terminal%20wrappers/ExitBattle.md#exitbattle) is called which ends the battle and the second encounter

## Third encounter
This is essentially the typical encounter of `Spuder`. This encounter is in effect when [flags](../../../Flags%20arrays/flags.md) 27 is true (met `Moth`), but it is assumed that `flagvar` 11 has a value below 150 for the EventDialogues to work correctly. It is also assumed that `flags` 37 is false as if it is true, the EventDialogues won't occur. It is assumed `Spuder` is the only enemy party member of the battle.

CheckEvent will detect the third encounter when `turns` is 0 meaning the first main turn and `flagvar` 11 being lower than 150. When that happens, EventDialogue 8 starts followed by `flagvar` 11 being set to 150 so it only triggers once at the start of the battle.

### EventDialogue 8
This EventDialogue is a mini cutscene where `Spuder` gains 2 `moves`:

- `Roar` sound plays
- `enemydata[0]` (should always be `Spuder`) animstate set to 100
- All player party member whose `hp` is above 0 has their animstate set to 11 (`Hurt`)
- Yield for a second
- [StatEffect](../../Visual%20rendering/StatEffect.md#stateffect) called on `enemydata[0]` (should always be `Spuder`) with type 5 (up yellow arrow)
- `enemydata[0]` (should always be `Spuder`) animstate set to 102
- `enemydata[0]` (should always be `Spuder`)'s `moves` set to 2 giving them 2 actor turns per main turns
- [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) called
- Yield for 0.75 seconds
- UpdateConditionIcons is called which calls UpdateConditionBubbles on all battleentity (all `playerdata` with right to false and all `enemydata` with `hp` above 0 with right to true)

### First encounter phase
After, the encounter proceeds as normal. Most of the logic is described in the [Spuder](../../Enemy%20actions/Enemies/Spuder.md) action page, but it's mostly an unrestricted version of their first 2 encounters.

This occurs until their `hp` becomes lower than their `maxhp` / 2 floored. When this happens, CheckEvent detects this which causes EventDialogue 7 to start followed by `flagvar` 11 being set to 200 so it only triggers once for the rest of the battle. This EventDialogue transitions to the second phase of the encounter.

### EventDialogue 7
This EventDialogue transitions the third `Spuder` encounter to its second phase:

- [HealConditions](../../Actors%20states/Conditions%20methods/HealConditions.md) called on `enemydata[0]` (should always be `Spuder`)
- `Roar` sound plays
- `enemydata[0]` (should always be `Spuder`) animstate set to 100
- All player party member whose `hp` is above 0 has their animstate set to 11 (`Hurt`)
- Yield for a second
- [AddNewEnemy](../../Actors%20states/Enemy%20party%20members/AddNewEnemy.md) called with id 8 (`ArmoredPoly`) and position (20.0, 0.0, 0.0)
- [AddNewEnemy](../../Actors%20states/Enemy%20party%20members/AddNewEnemy.md) called with id 1 (`Mushroom`) and position (20.0, 0.0, 0.0)
- Yield for a frame
- `enemydata[1]`'s `battlepos` set to (0.85, 0.0, 0.0)
- `enemydata[2]`'s `battlepos` set to (7.2, 0.0, 0.0)
- Both `enemydata[1]` and `enemydata[2]` has their `cantmove` set to 1 meaning they will need to wait a main turn before being able to act
- Over the course of 1 / 0.015 frames (~66.6666667), every enemy party members has their position lerped from (20.0, 0.0, 0.0) to their `battlepos`. Before the position change on each frame, their `rigid` gravity gets disabled and `overrideanim` set to true (the `Mushroom` also gets their animstate set to 23 (`Chase`))
- All enemy party members has their `rigid` gravity enabled, animstate set to 0 (`Idle`) and `overrideanim` set to false
- [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) called
- `enemydata[0]` (should be `Spuder`) animstate set to 102
- [ReorganizeEnemies](../../Actors%20states/Enemy%20party%20members/ReorganizeEnemies.md) called wit order {1, 0, 2}
- Yield for 0.75 seconds
