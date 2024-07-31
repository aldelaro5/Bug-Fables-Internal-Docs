# Combat tutorials EventDialogues
There are 2 combat tutorials in the game that involves many parts of the battle system.

The first one concerns basic combat, turn flow and skills. The second one is specifically for Turn Relay.

## Basic combat tutorial
The basic combat tutorial is a very complex process that involves the following:

- [flags](../../Flags%20arrays/flags.md) 15: Tells the game if the combat tutorial has already been given. It needs to be false for this combat tutorial process to do anything
- [flagvar](../../Flags%20arrays/flagvar.md) 11: When `flags` 15 is false, it tells the state of the combat tutorial:
    - 0: The initial state, no tutorials has been given yet
    - 1: The basic attack tutorial was given, but the turn flow / blocking tutorial and the skills tutorial wasn't given yet
    - 2: Both the basic attack tutorial and the turn flow / blocking tutorial were given, but the skills tutorial wasn't given yet
    - 3: All tutorials were given, nothing special happens from now on

The way this works is EventDialogue 0 to 2 and [CheckEvent](Update%20flows/Controlled%20flow.md#checkevent) works together with a special [enemy](../../Enums%20and%20IDs/Enemies.md): [MakiTutorial](../Enemy%20actions/Enemies/MakiTutorial.md). This enemy is special because its action logic is configured specifically to give tutorials, but act as a normal enemy party member otherwise. It is assumed that `MakiTutorial` is fought when `flags` 15 is false. The presence of this enemy is required to give the combat tutorial. It is also assumed that the player party is in its default order and composed only of `Bee` and `Beetle`.

`flagvar` 11 should always start with a value of 0 and it is how the game tracks the flow of the tutorial process which consists of many phases. Ultimately, it will leads to the battle premptively ending via [ReturnToOverworld](Terminal%20coroutines/ReturnToOverworld.md).

The following is how a complete combat tutorial process goes separated by phases.

### Start
On the first [controlled flow](Update%20flows/Controlled%20flow.md#controlled-flow) update, CheckEvent will run. It will detect that `flags` 15 is false so it will execute the tutorial flow logic on every controlled flow update from now on. The value of this flag is only set after the battle is over meaning it always will execute the logic from now on.

CheckEvent sees that `flagvar` 11 is 0 meaning we just started the tutorial. This causes EventDialogue 0 to happen. It is assumed that the player party is composed of `Bee` and `Beetle` in that order.

### EventDialogue 0
The following is the logic of EventDialogue 0 which gives the basic attack tutorial:

- `playerdata[0]` [FaceTowards](../../Entities/EntityControl/EntityControl%20Methods.md#facetowards) `playerdata[1]` (should always be `Bee` facing `Beetle`)
- [SetText](../../SetText/SetText.md) is called in [dialogue](../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
    - text: `commondialogue[12]` which is some dialogue followed by a [prompt](../../SetText/Individual%20commands/Prompt.md) giving the choice to skip the dialogue. If the player chooses to skip, a `|`[setvar](../../SetText/Individual%20commands/Setvar.md)`,11,1|` gets processed which sets `flagvar` 11 to 1
    - [fonttype](../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
    - linebreak: `messagebreak`
    - tridimensional: false
    - position: Vector3.zero
    - size: Vector3.one
    - parent: `playerdata[1]` (should always be `Beetle`)
    - caller: null
- Yield all frames until the [message](../../SetText/Notable%20states.md#message) lock is released
- All `playerdata` has their `flip` set to true
- Yield for a frame
- If `flagvar` 11 is still 0 (meaning the player didn't choose to skip the tutorial) some scripted player actions happens (the game controls everything here, no actual regular battle flow happens here):
    - `demomode` is set to true
    - `target` is set to 0
    - `availabletargets` is set to `enemydata`
    - `currentturn` is set to 1 (should always be `Beetle`)
    - [DoAction](Action%20coroutines/DoAction.md) is manually called with the entity being `playerdata[1]` (should always be `Beetle`) and actionid of -1 ([Beetle's basic attack](../Player%20actions/Basic%20attacks/Beetle.md)). The overall effect here is `Beetle` will use its basic attack on the first enemy which should always be `MakiTutorial`
    - Yield all frames until `action` is false (meaning DoAction is done)
    - [SetText](../../SetText/SetText.md) is called in [dialogue](../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
        - text: `commondialogue[19]` which is some dialogue about `Bee` being about to attack
        - [fonttype](../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
        - linebreak: `messagebreak`
        - tridimensional: false
        - position: Vector3.zero
        - size: Vector3.one
        - parent: `playerdata[0]` (should always be `Bee`)
        - caller: null
    - Yield all frames until the [message](../../SetText/Notable%20states.md#message) lock is released
    - `currentturn` is set to 0 (should always be `Bee`)
    - [DoAction](Action%20coroutines/DoAction.md) is manually called with the entity being `playerdata[0]` (should always be `Bee`) and actionid of -1 ([Bee's basic attack](../Player%20actions/Basic%20attacks/Bee.md)). The overall effect here is `Bee` will use its basic attack on the first enemy which should always be `MakiTutorial`
    - Yield all frames until `action` is false (meaning DoAction is done)
    - All `playerdata` have their `cantmove` set to 1, `lockskills` set to false and `lockitems` set to false. This simulates that they took an actor turn and the game will treat it the same way than if they actually attacked normally
- `demomode` is set to false
- `flagvar` 11 is set to 1 which advances to the next phase

Since `flagvar` is no longer 0, CheckEvent will no longer trigger EventDialogue 0 and the tutorial proceeds to the next phase.

### Turn flow tutorial
By this point, regardless if the player skipped the tutorial or not, `flagvar` should be 1 and the [player phase](Main%20turn%20life%20cycle.md#player-phase) should be over because all `playerdata` should have their `cantmove` set to 1.

This means that [DoAction](Action%20coroutines/DoAction.md) will be called on [MakiTutorial](../Enemy%20actions/Enemies/MakiTutorial.md) and their first actor turn has special logic if `flagvar` 11 is 1 which starts by starting EventDialogue 1 and yielding all frames until it is over (`inevent` goes to false).

### EventDialogue 1
The following is the logic of EventDialogue 1 which gives the turn flow tutorial:

- [SetText](../../SetText/SetText.md) is called in [dialogue](../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
    - text: `commondialogue[15]` which is some dialogue about turn flow
    - [fonttype](../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
    - linebreak: `messagebreak`
    - tridimensional: false
    - position: Vector3.zero
    - size: Vector3.one
    - parent: `playerdata[0]` (should always be `Bee`)
    - caller: null
- Yield all frames until the [message](../../SetText/Notable%20states.md#message) lock is released

### Block tutorial
We are back to `MakiTutorial`'s action. Right after the `inevent` yield, `playertargetID` is set to 0 (should always be `Bee`). Normally, the target is determined via [GetSingleTarget](../Actors%20states/Targetting/GetRandomAvaliablePlayer.md), but the enemy will specifically target `Bee` here because it is about to proceed with the blocking tutorial.

From there, the action logic continue as normal, except that after their animstate was set to 101 and after the 0.25 seconds yield, the following happens (normally, only a 0.2 seconds yield happen):

- [SetText](../../SetText/SetText.md) is called in [dialogue](../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
    - text: `commondialogue[16]` which is some dialogue about turn flow appended with `|`[noskip](../../SetText/Individual%20commands/Noskip.md)`|`
    - [fonttype](../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
    - linebreak: `messagebreak`
    - tridimensional: false
    - position: Vector3.zero
    - size: Vector3.one
    - parent: `playerdata[0]` (should always be `Bee`)
    - caller: null
- Yield all frames until the [message](../../SetText/Notable%20states.md#message) lock is released
- `commandsuccess` is set to true

From there, the [DoDamage](../Damage%20pipeline/DoDamage.md) proceeds as normal, but because `commandsuccess` was overriden to true, a regular block is guaranteed.

However, whether or not it is a regular or super block isn't guaranteed. The timing gets weirder than usual because of the SetText call done right before which skews the timing since [GetBlock](GetBlock.md) doesn't run while the [message](../../SetText/Notable%20states.md#message) lock is grabbed. Effectively, the timing to super block becomes the following:

- The last ~3 frames before the SetText call happened (most likely, they will be in the 0.25 seconds done right before)
- The frame corresponding to the exact moment the [message](../../SetText/Notable%20states.md#message) lock is released (GetBlock runs before DoAction resumes so it will see the lock being released before DoAction gets to continue)

Finally, at the very end of `MakiTutorial`'s action, since `flagvar` 11 is less than 3 (meaning the complete tutorial hasn't been given yet), it is set to 2 which advances to the next phase of the combat tutorial. Since `flagvar` 11 should be 3 by the next time `MakiTutorial` acts, this tutorial logic won't happen again.

### After block tutorial
We are now on the second main turn of the battle and CheckEvent runs. Since it sees that `flagvar` 11 is 2, EventDialogue 2 is called.

### EventDialogue 2
The following is the logic of EventDialogue 1 which gives the skills tutorial:

- If `flagvar` 11 is 2 (which should always be the case):
    - [SetText](../../SetText/SetText.md) is called in [dialogue](../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
        - text: `commondialogue[17]` which is some explanation about skills prepanded with `|`[boxstyle](../../SetText/Individual%20commands/Boxstyle.md)`,4|`
        - [fonttype](../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
        - linebreak: `messagebreak`
        - tridimensional: false
        - position: Vector3.zero
        - size: Vector3.one
        - parent: null
        - caller: null
    - Yield all frames until the [message](../../SetText/Notable%20states.md#message) lock is released
- [SetText](../../SetText/SetText.md) is called in [dialogue](../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
    - text: `commondialogue[18]` which is some more dialogues about skills
    - [fonttype](../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
    - linebreak: `messagebreak`
    - tridimensional: false
    - position: Vector3.zero
    - size: Vector3.one
    - parent: `playerdata[0]` (should always be `Bee`)
    - caller: null
- Yield all frames until the [message](../../SetText/Notable%20states.md#message) lock is released
- `enemydata[0]` (should be the `MakiTutorial`) has its animstate set to 13 (`BattleIdle`)
- All `playerdata` has their `lockskills` and `lockitems` are set to false
- `flagvar` 11 is set to 3

`flagvar` 11 being 3 advances to the last phase of the combat tutorial.

### Combat tutorial ending
From there, the battle is kind of in a stand still. `MakiTutorial` no longer has special logic and will attack as normal with the regualr block timing being in effect. Essentially, the battle is back to flow as normal.

This goes on until a special condition happens in CheckEvent which is that `turns` is 3. This means that 3 main turns total have passed. When this happens on the first controlled flow update, ExitBattle is called which start a [ReturnToOverworld](Terminal%20coroutines/ReturnToOverworld.md) without flee, set `cancelupdate` and `action` to true and `enemy` to false which also changes the flow to a [terminal](Update%20flows/Terminal%20flow.md) one.

This completely ends the tutorial process. It is expected that `flags` 15 gets set to true by the caller of this battle which prevents the tutorial to be given again.

## Turn Relay tutorial
The Turn Relay tutorial is isolated from the bigger combat one and it is relatively isolated overall. While the game heavily scripts the fight that this tutorial is given, the only requirements for this tutorial to work is to have a player party of `Bee`, `Beetle` and `Moth` in default party order.

The tutorial logic is checked by [CheckEvent](Update%20flows/Controlled%20flow.md#checkevent) and its logic runs only when [flag](../../Flags%20arrays/flags.md) 16 is true (Leif can fight in battle) and [flag](../../Flags%20arrays/flags.md) 24 is false (haven't received the Relay tutorial yet). This will run if applicable on the first [controlled flow](Update%20flows/Controlled%20flow.md) update. 

When these conditions are met, EventDialogue 3 starts.

### EventDialogue 3
The following is the logic of EventDialogue 3 which gives the Turn Relay tutorial:

- Both `playerdata[0]` (should always be `Bee`) and `playerdata[1]` (should always be `Beetle`) [FaceTowards](../../Entities/EntityControl/EntityControl%20Methods.md#facetowards) `playerdata[2]` (should always be `Moth`)
- [SetText](../../SetText/SetText.md) is called in [dialogue](../../SetText/Dialogue%20mode.md#dialogue-mode) with the following:
    - text: `commondialogue[20]` which is some dialogues about Turn Relay
    - [fonttype](../../SetText/Notable%20states.md#fonttype): 0 (`BubblegumSans`)
    - linebreak: `messagebreak`
    - tridimensional: false
    - position: Vector3.zero
    - size: Vector3.one
    - parent: `playerdata[2]` (should always be `Moth`)
    - caller: null
- Yield all frames until the [message](../../SetText/Notable%20states.md#message) lock is released
- Both `playerdata[0]` (should always be `Bee`) and `playerdata[1]` (should always be `Beetle`) have their `flip` set to true
- `flags` 24 is set to true
- Yield for 0.5 seconds
- [SetMaxOptions](../Player%20UI/SetMaxOptions.md) is called

Since `flags` 24 is true, the Turn Relay tutorial won't happen again.
