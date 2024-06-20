# Tattle
This action coroutine allows a player party member to spy an enemy party member if they pass an action command. The spy will only proceeed if `disablespy` is false, otherwise, the logic is limited to play the buzzer sound and calling [UpdateAnim](../../Visual%20rendering/UpdateAnim.md).

The coroutine expects `playerdata[currentturn]` to be the player party member that attemps the spying and for `avaliabletargets[option]` to be the enemy party member being spied.

## Action command setup

- DestroyHelpBox is called which sets `helpboxid` to -1 and destroys `helpbox` if it existed in 0.5 seconds with shrink before setting it to null
- `action` is set to true changing to an [uncontrolled flow](../Update%20flows/Uncontrolled%20flow.md)
- `caninputcooldown` is set to 0.0
- `blockcooldown` is set to 0.0
- `combo` is set to 0
- `killinput` is set to false
- `nonphyscal` is set to false
- `commandsuccess` is set to false
- [UpdateText](../../Visual%20rendering/UpdateText.md) is called

From there, if the `HPScope` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped, this will bypass the action command portion and the logic will be limited to playing the `AtkSuccess` sound and everything will continue as if `commandsuccess` was true.

Otherwise, the action command is done by calling [DoCommand](../../DoCommand.md) with the timer being 60.0, the type being `Crosshais` and the data being {3.0, the battleentity.`battleid` of the target enemy party member converted to float, 3.25, 10.0}. From there, all frames are yielded while `doingaction` is true.

## Spying process
This section only happens if the `HPScope` [medal](../../../Enums%20and%20IDs/Medal.md) was equipped or `commandsuccess` is true meaning the player passed the action command. If these conditions are fufilled, the logic of this section is limited to setting the player party member's battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) to 11 (`Hurt`).

- The `currentturn` player party member's battleentity.`spin` is set to (0.0, 20.0, 0.0)
- The `currentturn` player party member's battleentity.`animstate` is set to 4 (`ItemGet`)
- 0.65 seconds are yielded
- The `currentturn` player party member's battleentity.`animstate` is set to 13 (`BattleIdle`)
- The `currentturn` player party member's battleentity.`spin` is zeroed out
- The [SetText text advance](../../../SetText/Related%20Systems/Text%20advance.md)'s `skiptext` is set to false
- [waitinput](../../../SetText/Notable%20states.md#waitinput) is set to false
- instance.`inputcooldown` is set to 15.0
- A frame is yielded
- [SetText](../../../SetText/SetText.md) is called in [dialogue mode](../../../SetText/Dialogue%20mode.md#dialogue-mode) with the text being `|`[fwait](../../../SetText/Individual%20commands/Fwait.md)`,0.075||`[spd](../../../SetText/Individual%20commands/Spd.md),`-1||`[stopskip](../../../SetText/Individual%20commands/Stopskip.md)`|` followed by the correspondong tattle line from the `currenturn` player party member's `trueid` from [enemytattle](../../../TextAsset%20Data/Enemies%20data.md#enemytattle-data) data using the enemy party member's `animid` as the [enemy](../../../Enums%20and%20IDs/Enemies.md) id. The call also has these properties:
    - [fonttype](../../../SetText/Notable%20states.md#fonttype) of 0 (`BubblegumSans`)
    - linebreak of `messagebreak`
    - No tridimensional
    - position of Vector3.zero
    - No cameraoffset
    - size of Vector3.one
    - parent being `currentturn`'s player party member's battleentity
    - No caller
- A frame is yielded
- A nameless pure white 9Box is created with position (20.0, 0.0, 10.0), size of (10.75, 5.0) type 4 (torn piece of paper), sortorder of -20 without grow and a DialogueAnim attached with a targetpos of (3.5, -1.75, 10.0) and a speed of (0.1) (this DialogueAnim makes the 9Box appear from offsreen to the right slightly down)
- A new UI object named `enemyimage` is created childed to the 9Box with local position (-2.85, 0.0, 0.0), size of (3.0, 3.0, 3.0) and a sortOrder of 0 using the sprite defined as the portrait from [enemydata](../../../TextAsset%20Data/Enemies%20data.md#about-the-portrait-srpite-index) using the enemy party member's `animid` as the [enemy](../../../Enums%20and%20IDs/Enemies.md) id
- A string of text is prepared which are all the following appended together:
    - `|`[single](../../../SetText/Individual%20commands/Single.md)`|`
    - If the [languageid](../../../SetText/languageid.md) is `Japanese` and the enemy party member's `entityname` is 6 or more letters long, `|`[size](../../../SetText/Individual%20commands/size.md)`,X,1|` is appended where `X` is 1.0 - 0.075 * (the enemy party member's `entityname`'s length - 5) all clamped from 0.5 to 0.65 (notthing is appended otherwise)
    - `|`[line](../../../SetText/Individual%20commands/Line.md)`||`[halfline](../../../SetText/Individual%20commands/Halfline.md)`||`[size](../../../SetText/Individual%20commands/size.md)`,1,1,lock|`
    - `menutext[14]` (HP)
    - `: `
    - The enemy party member's `maxhp`. If the `DoublePain` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped or [flag](../../../Flags%20arrays/flags.md) 614 is true (HARDEST is active), it's the `maxhp` + `hardhp` instead
    - `|`[line](../../../SetText/Individual%20commands/Line.md)`|`
    - `menutext[17]` (Defense)
    - `: `
    - The enemy party member's `def`. If the `DoublePain` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped or [flag](../../../Flags%20arrays/flags.md) 614 is true (HARDEST is active), it's the `def` + `harddef` instead. There is an exception however: if that number is negative, `???` is appended instead of the number
    - `|`[line](../../../SetText/Individual%20commands/Line.md)`|`
    - If the [languageid](../../../SetText/languageid.md) is `Japanese`, `|`[size](../../../SetText/Individual%20commands/size.md)`,0.8,1,force|` is appended (nothing otherwise)
    - `menutext[137]` (Seen)
    - `: `
    - The seen counter of the matching [bestiary entry](../../../Enums%20and%20IDs/librarystuff/Bestiary%20entry.md) of the enemy party member
    - `|`[line](../../../SetText/Individual%20commands/Line.md)`|`
    - `menutext[138]` (Defeated)
    - `: `
    - The defeated counter of the matching [bestiary entry](../../../Enums%20and%20IDs/librarystuff/Bestiary%20entry.md) of the enemy party member
- [SetText](../../../SetText/SetText.md) is called in [non dialogue mode](../../../SetText/Dialogue%20mode.md#non-dialogue-mode) using the string assembled with the following properties:
    - [fonttype](../../../SetText/Notable%20states.md#fonttype) of 0 (`BubblegumSans`)
    - No linebreak
    - No tridimensional
    - position of (-0.65, 1.3. 0.0)
    - No cameraoffset
    - size of Vector3.one
    - parent being the 9Box
    - No caller
- All frames are yielded while the [message](../../../SetText/Notable%20states.md#message) lock is grabbed
- The [bestiary entry](../../../Enums%20and%20IDs/librarystuff/Bestiary%20entry.md) of the [enemy](../../../Enums%20and%20IDs/Enemies.md) is updated via UpdateJounal to unlock it in `librarystuff`
- The 9Box's DialogueAnim's `targetpos` is set to (20.0, 0.0, 10.0) (this sets it to move offscreen to the right)
- The 9Box is destroyed in 1.0 second

## Cleanup

- 0.5 seconds are yielded
- Unless the `HPScope` [medal](../../../Enums%20and%20IDs/Medal.md) was equipped, [EndPlayerTurn](../EndPlayerTurn.md) is called which advances the player party member's actor turn
- [CancelList](../../Player%20UI/CancelList.md) is called
- A frame is yielded
- `action` is set to false changing to a [controlled flow](../Update%20flows/Controlled%20flow.md)
- [UpdateText](../../Visual%20rendering/UpdateText.md) is called
- [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) is called
