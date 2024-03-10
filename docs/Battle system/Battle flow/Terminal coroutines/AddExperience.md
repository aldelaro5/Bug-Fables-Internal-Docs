# AddExperience
AddExperience is a terminal coroutine that is invoked whenever most battles are won with EXP gain. This coroutine will be the step between the battle finishing and [ExitBattle](../Terminal%20wrappers/ExitBattle.md) which is a wrapper around [ReturnToOverworld](ReturnToOverworld.md).

The coroutine yield breaks immediately if `alreadyending` is true (meaning AddExperience or [GameOver](GameOver.md) was already invoked) or `gameover` is in progress.

> NOTE: The vast majority of the logic in this coroutine concerns very verbose animations and UI rendering. For the sake of brevity, they will be paraphrased and details will be kept at a high level. All other logic will still be mentioned in granular details.

## Setup

- `alreadyending` is set to true which prevents double invocations
- `action` and `cancelupdate` are set to true changing to a [terminal flow](../Update%20flows/Terminal%20flow.md)
- `enemy` is set to false, but this doesn't do anything because we just changed to a terminal flow
- If the `HealingBuzz` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped:
    - All player party members with an `hp` above 0 has [Heal](../../Actors%20states/Heal.md) called on them for an ammount of 2 with nosound
    - The `Heal` sound is played (the reason the Heal calls didn't had sound is to avoid playing healing sounds multiple times)
- 0.5 seconds are yielded if any player party members were healed as a result of the `HealingBuzz` logic above
- If the `VictoryBuzz` [medal](../../../Enums%20and%20IDs/Medal.md) is equipped:
    - HealTP is called which plays a `Heal2` sound followed by a heal of instance.`tp` by 4 followed by a [ShowDamageCounter](../../Visual%20rendering/ShowDamageCounter.md) with type 2, the amount being 4, the start position being `playerdata[`[GetRandomAvaliablePlayer()](../../Actors%20states/Targetting/GetRandomAvaliablePlayer.md)`].battleentity` position + (2.0, 2.0, 2.0) and the end position being (5.0, 5.0, 5.0)
    - 0.5 seconds are yielded
- If `expreward` is above 5:
    - `checkingdead` is set to a new [UseCharm](../UseCharm.md) coroutine starting with `ExpUp` as the type
    - All frames are yielded while `checkingdead` is in progress
- The `switchicon` and `fronticon` are destroyed
- If [flags](../../../Flags%20arrays/flags.md) 613 is true (RUIGEE is active), `expreward` is set to 0
- Otherwise, if `expreward` was already 0 while `enemyfled` is false and instance.`partylevel` is less than 27 (the normal max rank), `expreward` is set to 1
- [RefreshEXP](../../Visual%20rendering/RefreshEXP.md) is called
- AddExperience determines if a `BattleWon` sound should be played. it will be if `musicvolume` is above 0.0 and `music[0]`.name is `Battle0` or `Battle6` (meaning that the music we were just playing was one of the 2 regular battle themes). If that is the case:
    - FadeMusic is called with a fadespeed of 0.05
    - The `BattleWon` sound is played and the AudioSource is kept track locally
    - If the `BattleWon` AudioSource indeed exists, its volume is set to `musicvolume` (this is needed because it's a sound so it wouldn't get the music volume by default)
- [flagvar](../../../Flags%20arrays/flagvar.md) 0 is set to `expreward` (this will be used for rendering later)
- instance.`hudcooldown` is set to -1.0
- All frames are yielded while there are still any enemy party members
- 0.45 seconds are yielded
- All player party members has their matching instance.`hud`'s child has its local position set to Vector3.zero
- For every player party members whose `hp` is above 0:
    - If they have the [Freeze](../../Actors%20states/BattleCondition/Freeze.md) condition, [BreakIce](../../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#breakice) is called on their battleentity
    - battleentity.`spin` is set to (0.0, -17.0, 0.0)
    - battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 4 (`ItemGet`)
- instance.`camtargetpos` is set to (-4.5, 0.0, 2.5)

## EXP gain visuals setup
The process to render the expereince gain screen is very verbose and complex so the procedure is paraphrased a lot here:

- A `textholder` GameObject gets created with a `DelAftBtl` tag (destroyed on [ReturnToOverworld](ReturnToOverworld.md)) childed to the GUICamera with a z local position of 5.0
- [SetText](../../../SetText/SetText.md) is called in [non dialogue mode](../../../SetText/Dialogue%20mode.md#non-dialogue-mode) using the following string: `|`[sort](../../../SetText/Individual%20commands/Sort.md)`,10000||`[color](../../../SetText/Individual%20commands/Color.md)`,4||`[center](../../../SetText/Individual%20commands/Center.md)`||`[dropshadow](../../../SetText/Individual%20commands/Dropshadow.md)`,0.05,-0.05||`[backbox](../../../SetText/Individual%20commands/Backbox.md)`,6|` followed by `menutext[117]` (a message informing the player they gained EXP where the number is `|`[var](../../../SetText/Individual%20commands/Var.md)`,0|`). The call has the following properties:
    - [fonttype](../../../SetText/Notable%20states.md#fonttype) of 2 (UNUSED, but gets overriden to `D3Streetism`)
    - No linebreak
    - No tridimensional
    - position of (0.0, 3.0, 5.0)
    - No cameraoffset
    - size of Vector3.one
    - parent being the `textholder`
    - No caller
- A new UI object called `ExpBar` with a `DelAftBtl` tag (destroyed on [ReturnToOverworld](ReturnToOverworld.md)), a DialogueAnim and a pure yellow color using `guisprites[4]` (the HUD background sprite) is created as the background of the EXP text
- `lvicon` is set to a new UI object named `Icon` using `guisprites[27]` (the EXP icon sprite) childed to the `ExpBar`
- A DynamicFont is setup childed to the `ExpBar` with a layer of 5 (`UI`) with a pure white color with the starting text being the following appended together:
    - instance.`partyexp` padded to the left with 3 `0`
    - `/`
    - instance.`neededexp`
- All player party members with an `hp` above 0 has their battleentity.`spin` zeroed out
- `leveled` is set to false
- 0.15 seconds are yielded

## EXP orbs counting and wait
This sub section concerns the counting of the EXP gained and the management of its fast version if requested by the player.

- instance.[skiptext](../../../SetText/Related%20Systems/Text%20advance.md#text-skip) is set to false (this isn't used for its SetText use since it will be managed manually)
- A new GetSkip coroutine is started and stored localled which will yield all frames until input 4 (Confirm) is pressed which wil cause instance.`skiptext` to be set to true
- The EXP orbs are counted starting with `bigexporbs` then moving to `smallexporbs` that exists (there is a frame yield between the 2 types's counting). A big one will have 1 EXP counted 10 times while a small one will only have it counted once. This is how the counting happens each time:
    - The orb gets destroyed
    - AddExp(1) is called which does the following:
        - Incrementes instance.`partyexp` by 1
        - An `Exp2` sound is played on `sounds[4]`, `sounds[5]`,`sounds[6]` or `sounds[7]` (the first one that isn't playing with `sounds[7]` being the falback) with a pitch being 1.5 + a random number between -0.1 and 0.1 inclusive
        - If instance.`partyexp` reached instance.`neededexp` (meaning this is a rank up), instance.`partyexp` is decreased by `neededexp` and the rank up is signaled by returning true
        - Otherwise, no rank up occured so false is returned
    - If the AddExp call resulted in a rank up, Leveled is called which sets `leveled` to true, but if it wasn't true before, a SpinAround component added to `lvicon` with its `itself` set to (0.0, 20.0, 0.0) followed by a `Lazer` sound being played at 1.2 pitch
    - The DynamicFont of the EXP's text is refreshed using the new instance.`partyexp`
    - If instance.`skiptext` is still false, a yield is done. The time of the yield is 0.075 seconds unless `expreward` was at least 10 where it will be 0.045 seconds instead
- At this point, the GetSkip coroutine call earlier is stopped since it no longer has any use
- If instance.`skiptext` was set to true by the end of the counting, the DynamicFont of the EXP's text is refreshed using the new instance.`partyexp` followed by 0.2 seconds being yielded
- A frame is yielded
- instance.`skiptext` is set back to false
- For a maximum of 50.0 frames (tracked with a local counter starting at 0.0 and incremented by 1/50 of the game's frametime until it reaches 1.0), all frames are yielded until any input is pressed which interupts this wait early
- If the `BattleWon` sound was played earlier, all frames are yielded while it's still playing and hasn't reached 2.35 seconds
- A second is yielded if no `BattleWon` sound was played

## Text UI teardown

- A DialogueAnim is added on the `ExpBar` object to move it to the left offscreen
- 0.5 seconds are yielded
- The `textholder` object is destroyed
- A frame is yielded

## Rank up logic
This logic only happens if `leveled` is true and instance.`partylevel` is less than 27 (the normal max rank).

The logic in here is very complex and mostly contains verbose animations so it will be paraphrased a lot.

### Setup

- instance.`partylevel` is incremented
- 0.25 seconds are yielded
- instance.`neededexp` is increased to a certain amount to indicate the new amount required to rank up. The first increase that applies in the following ones is the one that will occur (they are mutually exclusive):
    - +5 If [flags](../../../Flags%20arrays/flags.md) 656 is true (MOREFARM is active)
    - +3 if instance.`partylevel` is at least 24
    - +2 if instance.`partylevel` is at least 15
    - +1 if none of the above applied
- ChangeMusic is called with the `LevelUp` music
- `Prefabs/Particles/Floweretti` particles are instantiated childed to the `battlemap` and played
- All `Sprites/GUI/BattleMessage/rankX` are loaded where `X` is the [languageid](../../../SetText/languageid.md) (it falls back to English if no Sprites exists for it)
- All objects with tag `Enemy` are destroyed
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) is called
- MainManager.Heal is called
- instance.`hudcooldown` is set to -1
- instance.`option` is set to 0 and instance.`maxoptions` to 3

### Rank up animation

- All player party members has their battleentity.`animstate` set to 4 (`ItemGet`) and their `spin` to (0.0, 20.0, 0.0)
- All player party members are move to be centered on the screen using MiddlePos via lerping their battleentity.position for a maximum of 34 frames (tracked via a local framecounter advancing by 0.03 of the game's frametime from 0.0 to 1.0)
- All player party members gets their battleentity.`spin` zeroed out and their battleentity has a [SpecialAnimation](../../../Entities/EntityControl/EntityControl%20Methods.md#specialanimation) started with the `levelup` animation
- The letters of the Rank up messages appears in the following fashion:
    - A `LetterHolder` object is created and childed to GUICamera
    - New UI objects called `letterX` where X is the index of the letter sprite loaded earlier are created all childed to the `LetterHolder` with a FontEffects using just the [rainbow](../../../SetText/Individual%20commands/Rainbow.md) effect
    - The letters appear from offscreen at 99.0 in Y and 5.0 in Z using a lerp off their local position. This is done over a maximum of 200.0 frames and it is porportional to the amount of letters to show
    - 0.65 seconds are yielded
    - All letters have their local scale lerped from Vector3.one to (0.0, 1.25, 1.0) and their angle increased by Vector3.up * 5x the game's frametime continuoudly over the course of maximum 34 frames (tracked via a local framecounter advancing by 0.03 of the game's frametime from 0.0 to 1.0)
    - The `LetterHolder` is destroyed

### Rank up bonus UI setup

- 3 `Prefabs/Objects/Vine` are created using layer 15 (`3DUI`) childed to the `battlemap` with each their own UI icon. The vines are positioned centered to the screen using MiddlePos and they have a DialogueAnim with a tsize being Vector3.one * 2.0. Each has a `guisprites[42]` (a hexagon) on the background of a new `Overicon` object with a SpriteRenderer childed to their vine. The sprites used for the icons are:
    - 0: HP icon
    - 1: TP icon
    - 2: MP icon
- An MP count HUD element is created:
    - New UI object called `medalhud` childed to the `textholder` with a pure yellow color using the HUD background sprite
    - New UI object called `medalicon` chiled to the `medalhud` using the MP icon sprite
    - A DynamicFont is setup on the `medalhud` as parent with `dropshadow` and with the text being the following appended together:
        - instance.`bp` padded to the left by 2 `0`
        - `/`
        - instance.`maxbp` padded to the left by 2 `0`
    - A DialogueAnim is added on the `medalhud` so it appears from the top of the screen
- 0.8 seconds are yielded
- A 9Box is created with a DialogueAnim to make it grow
- A [SetText](../../../SetText/SetText.md) call happens in [non dialogue mode](../../../SetText/Dialogue%20mode.md#non-dialogue-mode) with the text being `|`[single](../../../SetText/Individual%20commands/Single.md)`|` followed by `menutext[122]` (a prompt message to pick a stat boost) and the following properties:
    - [fonttype](../../../SetText/Notable%20states.md#fonttype) of 0 (`BubblegumSans`)
    - No linebreak
    - No tridimensional
    - position of (-7.5, 0.15, 0.0)
    - No cameraoffset
    - size of Vector3.one
    - parent being the 9Box
    - No caller
- A new UI object is created named `t` childed to the GUICamera using the `cursorsprite[0]` with a SpriteBounce
- 5 frames are yielded (counted by MainManager.`framestep`)

### Rank up bonus UI naviguation and confirmation

- All frames are yielded while Input 4 (Confirm) isn't pressed
- A `Confirm` sound is played
- A frame is yielded
- The cursor gets childed to the `battlemap` then moved offscreen and DestroyText is called with it
- instance.`hudcooldown` is set to -1
- A [SetText](../../../SetText/SetText.md) call happens in [non dialogue mode](../../../SetText/Dialogue%20mode.md#non-dialogue-mode) with the text being `|`[single](../../../SetText/Individual%20commands/Single.md)`|` followed by `menutext[119]` (the HP rank up bonus description) and the following properties:
    - [fonttype](../../../SetText/Notable%20states.md#fonttype) of 0 (`BubblegumSans`)
    - No linebreak
    - No tridimensional
    - position of (-7.5, 0.15, 0.0)
    - No cameraoffset
    - size of Vector3.one
    - parent being the 9Box
    - No caller
- 5 frames are yielded (counted by MainManager.`framestep`)
- From here, an infinite loop is entered with a frame yield each iteration that won't exit until one of the 3 rank up bonuses have been chosen. Each iterations features the selected instance.`option`'s vine's sprites being rotated and scaled via a lerp continuously propotional to the games frametime. It also feature another SetText call just like the above when instance.`option` changes, but the text is `menutext[119 + X]` where `X` is instance.`option`:
    - If `caninputcooldown` isn't expired, it is decreased by the MainManager.`framestep` and no input can be taken until it expires
    - Otherwise, the logic depends on the input:
        - 2 / 3 (Left / Right): changes instance.`option` accordingly with wrap around to 2 followed by a scroll sound followed by `caninputcooldown` being set to 3.0
        - 4 (confirm): The UI loop is exited (the option chosen is indicated by instance.`option`)
- The `LevelUp` sound is played
- For a maximum of 40 frames (tracked with a local counter starting at 0.0 and incremented by 1/40 of the game's frametime until it reaches 1.0):
    - All other vines than the instance.`option` one have their local Y position lerped to 20.0 and their Z to 0.0 which moves them offscreen
    - The instance.`option` vine gets their angles reset
- The `CrowdClap` sound is played on `sounds[9]` with 0.5 volume

### Rank Up bonus application
The logic to apply the bonus depends on instance.`option`:

- 0 (HP): All player party members gets their `basehp` and `maxhp` gets incremented and a 1 `HP` bonus to the party is added to instance.`statbonus` via AddStatBonus
- 1 (TP): A 1 `TP` bonus to the party is added to instance.`statbonus` via AddStatBonus
- 2 (MP): instance.`maxbp` and instance.`bp` gets increased by 3 and a 3 `MP` bonus to the party is added to instance.`statbonus` via AddStatBonus. It also causes the `medalhud`'s DynamicFont text to be refreshed with the new instance.`bp` and instance.`maxbp`

After, the following happens:

- [ApplyStatBonus](../../ApplyStatBonus.md) is called
- Each player party member has a unique animation played there that sets their battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) to a value that depends on the `trueid`:
    - `Bee`: 111, also comes with a SlowSpinStop of (0.0, -20.0, 0.0) for 30.0 frametime on the battleentity
    - `Beetle`: 114, also comes with a battleentity.`flip` toggle
    - `Moth`: 115
- A frame is yielded
- [ApplyBadges](../../ApplyBadges.md) is called

### LevelUpMessage and teardown

- instance.`hudcooldown` is set to -1
- `medalhud`'s DialogueAnim's `targetpos` is set to (3.7, 10.0, 5.0) which moves it along with the rest of the HUD
- `medalhud` is destroyed in 0.5 seconds
- 0.5 seconds is yielded
- The 9Box is destroyed
- [flagvar](../../../Flags%20arrays/flagvar.md) 0 is set to 0 (this will be used to track whenever a coroutine gets done)
- A [LevelUpMessage](../LevelUpMessage.md) coroutine is started (this will set flagvar 0 to 1 when it's complete)
- All frames are yielded while flagvar 0 is 0 (meaning the LevelUpMessage isn't done yet)
- [ApplyBadges](../../ApplyBadges.md) is called
- A frame is yielded
- MainManager.Heal is called with noparticle and with nosound
- instance.`hudcooldown` is set to -1

## Special heal
If the rank up logic didn't happen, but instance.`partylevel` is at least 99 (which normally can never happen), Heal is called.

## Last steps

- A frame is yielded
- `sounds[9]` is stopped with a 0.002 delay
- [ExitBattle](../Terminal%20wrappers/ExitBattle.md) is called
