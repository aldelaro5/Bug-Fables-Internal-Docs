# Boot and reset process
This page aims to detail how the game initialises itself from the first component's Start to the title screen on StartMenu.

## How the main scene is setup
The main scene contains 2 important rooted GameObject:

- `StartMenu` which has a disabled StartMenu component (it gets enabled during the boot process)
- `MainCam` which contains the structure of the [Camere system](../General%20systems/Camera%20system.md), but more importantly, it has a child called `MainCamera` that has the only MainManager component in the game and it is enabled

This effectively means that the earliest Unity component that will kick in is MainManager which has a Start method. This is where the boot process starts.

## MainManager.Start
MainManager.Start can be considered the first declared method to run with the only methods running before it are runtime defined such as static constructors. It is the earlest part of the boot process.

Here's what happens here:

- Application.runInBackground is set to true only on non console platforms (Application.isConsolePlatform is false)
- Thread.CurrentThread.CurrentCulture is set to the `en-US` culture info. This is done to force all data parsing with float to use `.`. NOTE: It would be more appropriate if this was the invariant culture, but `en-US` happens to have the correct parameters for this
- On the first boot (not a reset), InputIO.StartUp runs. This is a platform specific method that may need to run some logic. On Steam, it initialises the SteamManager GameObject which is the only GameObject marked DontDestroyOnLoad so it persists between resets
- Time.timeScale is reset to 1.0 (normal speed)
- `musicresume` is reset to -1.0 (it avoids carrying over the value from a reset to avoid a bad timestamp restore, check the [musicresume](../General%20systems/Music%20playback.md) for more information)
- The `instance` is set to this MainManager which completes the singleton setup
- A LoadEverything coroutine starts which is the next phase of the boot process
- If not running the the Unity editor, the Application.version gets logged silently

## LoadEverything (part 1/2)
LoadEverything is the main startup coroutine of the game. It runs once per each boot or reset as it is called by Start and only Start.

```cs
private static IEnumerator LoadEverything()
```
The reason this is a coroutine is because there is a loading icon childed to StartMenu with a SpriteBounce that continuously run as this boot process is ongoing. If it was a method, the icon wouldn't animate, but being a coroutine, it inserts yields in various places to allow the icon to animate as no loading is done asynchronously. While it does result in the icon animating in a staggered manner, it still means the icon isn't perpetually frozen.

There's 2 parts to LoadEverything: anything before LoadEssentials and anything after it's done. Here's what happens in the first part of that coroutine:

- Cursor.visible is set to false (so no mouse cursor is visible)
- `MainCamera`.depthTextureMode is set to Depth. Check the [camera system](../General%20systems/Camera%20system.md) documentation to learn more about what the `MainCamera` is
- RenderSettings.ambientLight is set to Color.gray
- `basicload` is reset to false (this is so that nothing runs that could interfere with the boot process after a reset)
- A frame is yielded
- `events` is set to a new EventControl component on the `instance`. This should be the only EventControl in the game
- A frame is yielded
- `chaptername` is set to a new LoadEssentials coroutine which is the most intensive part of the boot process that loads the vast majority of the assets in the game into various fields
- Yield all frames until `chaptername` is null (indicating that LoadEssentials completed)

The rest will be mentioned below in the second part. The next phase is LoadEssentials.

## LoadEssentials
This phase is the most intensive part of the boot process:

```cs
private IEnumerator LoadEssentials()
```
It's a coroutine for the same reason that LoadEverything is. It is expected that the coroutine is set to `chaptername` as this coroutine will set it to null upon completion so LoadEverything can yield on it.

This is potentially the most performance intensive method in the whole game because it loads the vast majority of the game's assets into several MainManager fields for usage in the game. Most of these fields are static as they aren't expected to change during the game, but they will be reassigned on reset since this coroutine is also called once on a reset.

Here is what happens in the coroutine:

- The `letterpool` is initialised to have 500 [letter slots](../SetText/Letter%20slots.md) each with NewLetter called on them to initialise all slots
- `screenshake` is reset to Vector3.zero. Check the [camera system](../General%20systems/Camera%20system.md) documentation to learn more
- Yield for a frame
- The following fields are set to the return of a Resources.LoadAll call each followed by a frame yield done in order:
    - `textboxsprites`: Sprite from `Sprites/GUI/textbox`
    - `partfab`: GameObject from `Prefabs/Particles`
    - `parttex`: Texture from `Sprites/Particles`
    - `spritepart`: Sprite from `Sprites/Particles`
    - `asounds`: AudioClip from `Audio/Sounds`
- On console platforms, `msounds` is set to a new array each containing the return of Resource.Load of the AudioClip from `Audio/Music/X` where `X` a [Musics](../Enums%20and%20IDs/Musics.md) enum value as string for all Musics eneum values (this is done to cache them for faster loads later on console)
- `dsounds` is set to the return of Resource.LoadAll AudioClip from `Audio/Sounds/Dialogue`
- A frame is yielded
- A silent log line is printed indicating the amount of `partfab` loaded, the amount of `parttex` + `spritepart` loaded and the amount of `asounds` and `dsounds` loaded
- `GUICamera` is set to the first child of `MainCamera`'s Camera component. Check the camera system documentation to learn more about this structure
- The following fields are set to the return of a Resources.Load call done in order (no yielding between them):
    - `defaultpmat`: The PhysicMaterial from `Materials/DefaultPhysis`
    - `spritemat`: The Material from `Materials/SpriteMat`
    - `holosprite`: The Material from `Materials/SpriteHologram`
    - `emptymat`: The Material from `Materials/Empty`
    - `spritematlit`: The Material from `Materials/SpriteLit`
    - `outlinemain`: The Material from `Materials/OutlineMain`
    - `spritedefaultunity`: The Material from `Materials/SpriteDefault`
    - `windShader`: The Material from `Materials/WindShader`
    - `Main3D`: The Material from `Materials/3DMain`
    - `Fade3D`: The Material from `Materials/3DFade`
    - `grayscale`: The Material from `Materials/Grayscale`
    - `letters`: The TextAsset from `Data/Letters`.ToString().ToCharArray() (it is UNUSED in practice)
- The following fields are set to the return of a Resources.LoadAll call each followed by a frame yield done in order:
    - `grasssprite`: Sprite from `Sprites/Objects/grass`
    - `leafsprites`: Sprite from `Sprites/GUI/battleleaves`
- The following fields are set to the return of a Resources.Load call done in order (no yielding between them):
    - `fakelight`: The Material from `Materials/FakeLight`.shader
    - `shadowsprite`: The Sprite from `Sprites/Misc/shadow`
    - `mainPlane`: The Material from `Materials/MainPlane`
    - `fadePlane`: The Material from `Materials/FadePlane`
    - `languagehelp`: The TextAsset from `Data/LanguageHelp`.ToString().Split(`\n`)
- `endata` is loaded, check the [entity data](../TextAsset%20Data/Entity%20data.md) documentation to learn more
- A frame is yielded
- `hitpart` is set to a the ParticleSystem component of a new instance of the `Prefabs/Particles/HitPart` prefab positioned offscreen at -999.0 in y
- `deathpart` is set to a the ParticleSystem component of a new instance of the `Prefabs/Particles/deathsmoke` prefab positioned offscreen at -999.0 in y and -90.0 x rotation
- `globalcamdir` is configured as a new GameObject named `CamDir` childed to `MainCamera` with no local positioning. Check the camera system documentation to learn more
- `librarysprites` is set to the return of Resource.LoadAll Sprite from `Sprites/Items/EnemyPortraits`
- A frame is yielded
- The following fields are initialised:
    - `extrafollowers` (new list)
    - `sounds` (15 new AudioSources, check the [sounds playback](../General%20systems/Sounds%20playback.md) documentation to learn more)
    - `music` (1 new AudioSource, check the [music playback](../General%20systems/Music%20playback.md) documentation to learn more)
- `leafpos` is loaded, check the [leafpos](../TextAsset%20Data/Miscellaneous%20global%20data.md#leaves-destination-positions-during-battle-transition) documentation to learn more
- `questchecks` is loaded, check the [questcheck data](../TextAsset%20Data/BoardQuests%20data.md#questchecks-data) documentation to learn more
- A frame is yielded
- `termacadeprize` is loaded, check the [termacade prize data](../TextAsset%20Data/Termacade%20Prizes%20data.md) documentation to learn more
- `enemydata` is loaded, check the [enemydata](../TextAsset%20Data/Enemies%20data.md#enemydata-data) documentation to learn more
- A frame is yielded
- `fonts` is initialised to a new array containing all the Font from `Fonts/X` where `X` is a font element (check the [fonttype](../SetText/Notable%20states.md#fonttype) documentation to learn more)
- `fontmat` is initialised to a new array containing all the Material from `Fonts/X` where `X` is a font element (check the [fonttype](../SetText/Notable%20states.md#fonttype) documentation to learn more)
- `bleeps` is initialised to a new AudioSource added to the MainManager.instace, check the [bleep](../SetText/bleep.md) documentation to learn more
- All `sounds` element are set to AudioSource each added to the MainManager.instance with a velocityUpdateMode of Fixed and a volume of `soundvolume`, check the [sounds playback](../General%20systems/Sounds%20playback.md) documentation to learn more
- A frame is yielded
- `music[0]` is set to an AudioSource added to the MainManager.instance set to play on loop, check the [music playback](../General%20systems/Music%20playback.md) documentation to learn more
- A frame is yielded
- `guisprites` is set to all Sprite loaded from `Sprites/GUI/gui` followed by all the ones loaded from `Sprites/GUI/gui2`
- `cursorsprite` is set to an array with a single element being `guisprites[145]`
- If Application.version contains `Beta`, NewUIObject is called to create an object called `betawatermark` childed to the `GUICamera` locally positioned at (7.8, -4.4, 1.0) using the sprite `guisprites[169]` with a sortOrder of 9999 which has a color of pure black at 0.1 alpha
- A frame is yielded
- `letterbox` is initialised to an array of 2 elements each the following configuration of a NewSolidColor of Color.black each childed to the `GUICamera`:
    - Name: `letterboxX` where `X` is the `letterbox` index
    - Pivot: (0.5, 0.5, 0.5)
    - Color: Color.clear
    - Layer: 5 (`UI`)
    - Scale: (5.0, 0.5, 1.0)
    - sortingOrder: -40
- `letterbox[0]` is locally positioned at (0.0, 5.75, 10.0)
- `letterbox[1]` is locally positioned at (0.0, -5.75, 10.0)
- A frame is yielded
- `recipedata` is loaded, check the [recipe data](../TextAsset%20Data/Recipes%20data.md) documentation to learn more
- A frame is yielded
- instance.`projectilepsrites` is set to Resource.LoadAll Sprite from `Sprites/Misc/projectiles`
- A frame is yielded
- Physics.gravity is set to -40.0 in y
- InputIO.GetJoyButtons is called TODO: Document InputIO
- A frame is yielded
- InputIO.SetDefaultKeys is called TODO: Document InputIO
- A frame is yielded
- `chaptername` is set to null to mark this coroutine as completed

## LoadEverything (part 2/2)
After LoadEssentials, the boot process continues in LoadEverything:

- An frame is yielded after LoadEverything is done
- If `languageid` isn't defined (it's negative meaning this is the first boot), InputIO.LoadSettings is called without overwrite TODO: Document InputIO and config.dat. Otherwise (`languageid` is defined after a reset), SetVariables is called, but this call isn't useful since it will get called later by StartMenu, check the SetVariables section below to learn more
- A frame is yielded
- [SetRenderTexture](../General%20systems/Camera%20system.md#rendertexture) 0 is called to turn the feature off
- A frame is yielded
- instance.[FontSet](../SetText/Letter%20slots.md#fontset) is called
- A frame is yielded
- `basicload` is set to true which enables most of the logic of MainManager's Unity events (Update, LateUpdate and FixedUpdate)
- The StartMenu in the scene is finally enabled and it is where the boot process will resume

From there, LoadEverything is done and won't be called until a reset happens. This means MainManager starts its update process, but the boot process isn't completely done. It resume on StartMenu.Start which has been newly enabled so it will kick in on the next frame.

## StartMenu.Start
The boot process continues in StartMenu.Start which setup the last phase of the boot process.

Here's what happens here:

- RenderSettings.skybox is set to Resource.Load Material from `Materials/Skybox/Grass1` with its `_Tint` color set to Color.gray
- The StartMenu gets childed to the `GUICamera` locally positioned at (0.0, 4.5, -17.0)
- StartMenu's `sprites` gets set to all SpriteRenderer in the StartMenu's children
- Any invoke of [DoClock](DoClock.md) on MainManager.instance is cancelled
- StartMenu's `copycursor` is initialised for the save file selection
- If this Start isn't running after selecting the language select option on the title screen (StartMenu's `noload` is false), all the save data get preloaded for selection later. Otherwise (this is a reset that happened due to selecting the language select option on the title screen), Resource.UnloadUnusedAssets is called
- StartMenu.LoadModel is called which setups some title screen entities depending on the furthest progression of any save file
- What happens here depends if `languageid` is defined (meaning not negative) if it's not defined, it means that no language was loaded from config.dat so the game will go to the language selection screen with by doing the following:
    - SetUpList is called to setup a [languages listtype](../ItemList/List%20Types%20Group%20Details/Languages%20list%20Type.md) without description and sell
    - [ShowItemList](../ItemList/ShowItemList.md) is called to setup the same listtype with a position of (-1.0, -0.35) without description and sell
    - `sprites[3]` (the load icon) gets disabled
- Otherwise (`languageid` is defined from config.dat):
    - MainManager.instance.SetVariables is called which is the last step of the boot process
    - The Intro coroutine starts which starts the actual title screen UI
- `noload` is reset to false (since the reset completed)

So from there, there's 2 possibilities: either the game proceeds to the language selection screen or the game proceeds to the last stage of the boot process which is SetVariables.

### Language selection
Essentially, most of the handling here is done entirely by MainManager.Update, specifically handling the languages listtype. Only confirmations are accepted since a language must be selected before proceeding. Eventually, when a language is selected, the [languages listtype confirmation handling](../ItemList/List%20Types%20Group%20Details/Languages%20list%20Type.md#confirmation-handling) kicks in which will notably call SetVariables and reach the last stage of the boot process.

It's possible however after boot to get back to the language selection from the title screen. Doing this will cause a full reset, but the `languageid` becomes undefined and then saved in the settings so the next time StartMenu.Start kicks in, it will detect this case (since this also sets `noload` to true) and immediately go to the language selection.

There's also a specific method involved when confirming the language: LoadLangSpecific.

```cs
public static void LoadLangSpecific()
```
This method applies a German language specific adjustement where some medals UI icons gets their sprites changed such that they display an "A" instead of an "M". It is only called as part of event 22 (loading a save file) and on Update when processing a confirmation in the language selection listtype. This ensures that the adjustement is applied or not depending on the language being selected.

## SetVariables
This is the last stage of the boot process where lots of TextAsset are parsed and loaded into fields as well as several fields getting initialised.

```cs
public void SetVariables()
```
It's important to mention that it's possible this is the second time SetVariables gets called: it might have been called earlier during LoadEverything as mentioned above. However, the only time this happens is under circumstances where a second call here ALWAYS happen meaning the first call simply redo everything the first one did rendering the first call useless, but doesn't have adverse effects in practice. Nonetheless, by the time StartMenu.Start is done or the language selection is done, SetVariables is guaranteed to be called once which is required for the game's boot.

Here is what happens here:

- The following fields are set to the return of a Resources.Load call that loads language specific data done in order where `X` in the asset path is the `languageid` (check the linked documentation to learn more):
    - `battlemessage`: All Sprite from `Sprites/GUI/BattleMessage/battlemX` (if this returns null or empty array, the path fallsback to `Sprites/GUI/BattleMessage/battlem0` which are the Sprite in English)
    - `menutext`: TextAsset from `Data/DialoguesX/MenuText` ([dialogue data](../TextAsset%20Data/Dialogue%20data.md) documentation)
    - `commondialogue`: TextAsset from `Data/DialoguesX/CommonDialogue` ([dialogue data](../TextAsset%20Data/Dialogue%20data.md) documentation)
    - `musicnames`: TextAsset from `Data/DialoguesX/MusicList` ([musics data](../TextAsset%20Data/Musics%20data.md) documentation)
    - `commandhelptext`: TextAsset from `Data/DialoguesX/ActionCommands` ([other language specific data](../TextAsset%20Data/Miscellaneous%20language%20specific%20data.md) documentation)
    - `areanames`: TextAsset from `Data/DialoguesX/AreaNames` ([other language specific data](../TextAsset%20Data/Miscellaneous%20language%20specific%20data.md) documentation)
- The following fields gets initialised:
    - `samiramusics` (empty list). Check the [music playback](../General%20systems/Music%20playback.md) documentation to learn more
    - The hardcoded values of `prizeflags`, `prizeids` and `prizeenemyids`, check the [prize medals](../General%20systems/Medals.md#prize-medals) documentation to learn more
    - Both elements of `badgeshops` and `avaliablebadgepool` (each being empty lists)
    - All 3 `items` elements (each being empty list)
- The following data gets loaded (check the linked documentation to learn more about the formats and TextAsset used, some are language specific):
    - `itemdata`: [items data](../TextAsset%20Data/Items%20data.md) documentation
    - `skilldata`: [skills data](../TextAsset%20Data/Skills%20data.md) documentation
    - `badgedata` and `badgeorder`: [medals data](../TextAsset%20Data/Medals%20data.md) documentation
    - `boardquestdata`: [boardquests data](../TextAsset%20Data/BoardQuests%20data.md) documentation
    - `librarydata`, `libraryorder`, `achiveicons`, `discoveryicons`: Several library data documentations:
        - [discoveries data](../TextAsset%20Data/Discoveries%20data.md) documentation
        - [enemies data](../TextAsset%20Data/Enemies%20data.md) documentation
        - [recipes data](../TextAsset%20Data/Recipes%20data.md) documentation
        - [records data](../TextAsset%20Data/Records%20data.md) documentation (logic documented in the [achievements](../General%20systems/Achievements.md) documentation)
- All 3 `boardquests` elements gets initialised (all with a list containing 0 which is the dummy `None` element). Check the [QuestBoards system](../General%20systems/Quest%20boards.md) documentation to learn more
- `enemynames` gets loaded, check the [enemies data](../TextAsset%20Data/Enemies%20data.md) documentation to learn more
- `badges` gets initialed to a new list followed by SetUpBadges being called. Check the [medals](../General%20systems/Medals.md) documentation to learn more
- `statbonus` gets initialised to a new list followed by a ChangeParty fromscratch with a party of {`Bee`, `Beetle`}. Check the [player party](../General%20systems/Player%20party.md) documentation to learn more
- All of the following fields gets initialised to a value:
    - `enemyencounter`: new int[256, 2], check the [enemy ecounter save data](../External%20data%20format/Save%20File.md#line-18-array-line-enemy-encounters) documentation to learn more
    - `insideid`: -1 (not in an inside), check the [insides](../MapControl/Insides.md) documentation to learn more
    - `tp`, `maxtp` and `basetp`: 10
    - `bp` and `maxbp` (MP fields): 5
    - `partylevel`: 1
    - `maxitems`: 10
    - `maxstorage`: 35
    - Several [camera system](../General%20systems/Camera%20system.md) related fields:
        - `camspeed`: 0.1
        - `camanglespeed`: 0.1
        - `camoffset`: `defaultcamoffset` which is normally (0.0, 2.25, -0.25)
        - `camoffset2`: Vector3.zero
        - `camangleoffset`: `defaultcamangle` which is normally (10.0, 0.0, 0.0)
    - [flags](../Flags%20arrays/flags.md) array: new array of 750 elements
    - [flagvar](../Flags%20arrays/flagvar.md) array: new array of 70 elements with these initial values (rest are left to 0):
        - `flagvar[28]`: 9500
        - `flagvar[29]`: 4500
    - [flagstring](../Flags%20arrays/flagstring.md) array: new array of 15 elements
    - [regionalflags](../Flags%20arrays/Regionalflags.md) array: new array of 100 elements
    - [crystalbflags](../Enums%20and%20IDs/crystalbfflags.md) array: new array of 50 elements
    - `vectorflags` array: new array of 10 elements (this array is UNUSED and it contains Vector3 elements)
    - `librarystuff` array: new bool[`librarydata`.GetLength(0), `librarydata`.GetLength(1)]. Each first dimension's subarrays is docuimented in the following pages:
        - 0: [discoveries entry](../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md)
        - 1: [bestiary entry](../Enums%20and%20IDs/librarystuff/Bestiary%20entry.md)
        - 2: [Recipe entry](../Enums%20and%20IDs/librarystuff/Recipes%20entry.md)
        - 3: [Records entry](../TextAsset%20Data/Records%20data.md)
        - 4: [Areas](../Enums%20and%20IDs/librarystuff/Areas.md)
    - `musicvolume`: 1.0 (the [music playback](../General%20systems/Music%20playback.md) base volume scaler)
    - `soundvolume`: 1.0 (the [sound playback](../General%20systems/Sounds%20playback.md) base volume scaler)
    - `halt`: false (a field only used in [event](../Enums%20and%20IDs/Events.md) 1 which is after hitting the bridge at Snakemouth Den)
    - instance.`switchicon`: The SpriteRenderer of a NewUIObject with name `SwitchIcon` childed to the `GUICamera` with a pos of (-8.0, 0.0, 10.0) scaled by 0.75x uniformely
    - `promptpick`: -1 (A [prompt](../SetText/Individual%20commands/Prompt.md) or [numberprompt](../SetText/Individual%20commands/NumberPrompt.md) related field)
- LoadItemSprites gets called to initialise `itemsprites`, check the section below for details
- InputIO.LoadSettings without overwrite is called to reload config.dat

Once this is done, the boot process is complete and the game is ready to proceed to the title screen.

### LoadItemSprites and GetMoreItem
These 2 methods involve complex logic to initialise `itemsprites`.

```cs
private static void LoadItemSprites()
```
This is a part of SetVariables.

This method intialises `itemsprites` as a [2, 256] array of sprites where the [0, `i`] array contains the Items sprites (each have an item id of `i`) and the [1, `m`] array contains the Medals sprites (each have a medal id of `m`). The sprites comes from the `Sprites/Items/items0` and `Sprites/Items/items1` sprites arrays.

The sprite array used and the index used in that sprite array for each items or medals are determined with a very specific logic:

- For items and where the item id is below 176, the sprite is pulled from `Sprites/Items/items0` where the sprite array index is the same as the item id
- For items and where the item id is 176 or above, the sprite is pulled from `Sprites/Items/items1` where the sprite index is 176 - the item id
- For medals and where the matching `badgedata`'s field 8 is negative, the sprite is pulled from `Sprites/Items/items0` where the sprite index is 176 + the medal id
- For medals and where the matching `badgedata`'s field 8 isn't negative, the sprite is pulled from `Sprites/Items/items1` where the sprite index is the matching `badgedata`'s field 8

```cs
private static Sprite GetMoreItem(int i, int offset = 176)
```
This is a part of LoadItemSprites that loads a sprite from `Sprites/Items/items1` using the index `offset` - `i`.

## Reset
As mentioned multiple times, the game supports soft resetting itself through the Reset method:

```cs
public static void Reset()
```
All this method does is call SceneManager.LoadScene(0) which reloads the main scene. Since the entire game is built with a single scene being the main scene, reloading it is equivalent to a full soft reset: everything is lost except the SteamManager (if applicable) and any values of static fields.

When a reset happens, most of the boot process is redone from scratch.
