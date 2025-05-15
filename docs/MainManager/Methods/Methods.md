## Unity events

```cs
private void Start()
```
The first method from the game that gets called by Unity. This is due to MainManager being one of the only component attached to the main scene of the game on the main camera GameObject meaning once the engine starts the game, this will be called first (this is also due to the fact MainManager has high execution priority).

It is possible this method gets called multiple times. The first call is detected by `basicload` being false which will additionally causes InputIO.StartUp to run. On Steam, this method creates the SteamManager and calls Init on Steamworks.NET, but the logic is platform specific.

Overall, Start aims to initialise core fields to sane defaults, set the current culture to `en-US` (enforces save files (de)serialisation) and to start a LoadEverything coroutine which does most of the startup work.

```cs
private void Update()
```
The main global update method of the game. It only does anything if `basicload` is true (LoadEverything completed).

Here is a summary of the tasks it handles in order:

- Handles dialogue mode SetText updates to process textbox inputs as well as prompts, numberpompts and letterprompts inputs
- Handles setting the Time.timeScale when cooking when the player requests to fast forward the cutscene TODO: detail the conditions for this
- Handles every ItemList inputs updates and handles most of their processing
- Calls GetJoystick

```cs
private void LateUpdate()
```
TODO: very complex, might need to be documented separately

```cs
private void FixedUpdate()
```
If `basicload` is true (LoadEverything ran to completion), the following happens:

- RefreshCamera is called
- LoopMusic is called

## Methods

```cs
private void FontSet()
```
A part of LoadEverything.

This method calls SetText in non dialogue mode in such a way that all letters contained in each fonts and whose characters are in any `letterPromptHelp` will be rendered under the game's normal viewport. This is a caching optimisation to force Unity to preload most letters and fonts materials used for future text rendering which avoids the possibility of stutters when loading any of these assets.

The SetText call is done with a position of (0.0, 0.0, -1.0), parent of `MainCamera` and a string of `|single|` followed by a list of font commands with all valid indexes each being superceded with every letters of every `letterPromptHelp`.

```cs
private static IEnumerator LoadEverything()
```
The main startup coroutine of the game, called by Start.

This is the coroutine that dispatches the main bootstrapping tasks such as setting up some fields like `events` and overall maintains consistencies between game resets (such as calling SetRenderTexture(0) to turn it off). The main work of the game's boot is done by the LoadEssentials coroutine and the SetVariables method which are called by LoadEverything. The coroutine contains multiple frame yields to allow the loading icon to somewhat (even if not consistently) animate via its attached SpriteBounce during the loading process.

```cs
public static void LoadLangSpecific()
```
This method applies a German language specific adjustement where some medals UI icons gets their sprites changed such that they display an "A" instead of an "M". It is only called as part of event 22 (loading a save file) and on Update when processing a confirmation in the language selection listtype. This ensures that the adjustement is applied or not depending on the language being selected.

```cs
public static void SetRenderTexture(int downsampleindex)
```
Updates the `MainCamera`.`targetTexture` such that the game's viewport shows a downsampled version of the game by a factor of `downsampleindex`. If `downsampleindex` is 0, the `targetTexture` is set to null which disables this features and shows the game as is without any downsampling.

This is used as part of a now unused setting, but it's still mainly used as part of the Termacade games. On LoadEverything, this is always called with a `downsampleindex` of 0.

```cs
public static void RefreshRenderTex()
```
A part of SetRenderTexture that updates the camera's Renderer. This is only meaningfully called as part of SetRenderTexture, but PauseMenu technically calls is as part of the now unused settings that allowed to change the downsampling factor of the `MainCamera`.`targetTexture`.

```cs
public static bool AllOnGround(EntityControl[] entities)
```
UNUSED: No one calls this, but it remains functional.

If every entity in `entities` is `onground`, this returns true and it returns false otherwise.

```cs
public static string GetDialogueFromMap(Maps mapid, int lineid, int boxid)
```
Gets the entire SetText line whose id is `lineid` from the map `mapid` if `boxid` is -1.

If `boxid` isn't -1, this will get a specific parts of the SetText line using the `boxid` as the index in an array produced by splitting the SetText line by `|next|` (with StringSplitOptions.None meaning empty strings are included in the resulting split array).

This is used as part of the `|getfrommap|` SetText command and some specific events.

NOTE: If `boxid` isn't a valid index of the string after it is split by `|next|`, an exception will be thrown. Also, this doesn't support the typical dialogue line id format, only a map line id is supported and the id must be valid inside the `mapid` map or an exception will be thrown.

```cs
public static Entity_Data[] LoadEntityData()
```
A part of LoadEssentials.

Loads and return the array value that `endata` should be assigned to which contains all data belonging to each specific animid. The data comes from the `Data/EntityValues` TextAsset and the resulting array is indexed using the enum format of animids (meaning the index 0 = None).

```cs
public static void EndMiniGame(bool antialias, int score)
```
Performs the necessary operations to end a Termacade game where the FXAA component's enablement is set to `antialias` where the game ended with a score of `score`. Among the tasks done by this method includes calling SetRenderTexture(0), calling GetRewardTokens(`score`).

The `antialias` parameter is expected to come from the component containing the Termacade game logic and it is expected to contain the initial enablement of the FXAA component when the game starts. This is used to restore the enablement to its original state. The `score` is used to determine how much game tokens should be awarded by GetRewardTokens.

```cs
private IEnumerator LoadEssentials()
```
A major part of LoadEverything which manages the boot process. It is expected that this coroutine is set to the `chaptername` field which can be used to track if it's still in progress (it will be set to null at the end of the coroutine).

This coroutine loads most cached and core assets of the game so they can be referenced easily without load calls later. Most of these assets are stored in fields of MainManager (most of them are static) and they aren't expected to change much after. The coroutine contains multiple frame yields to allow the loading icon to somewhat (even if not consistently) animate via its attached SpriteBounce during the loading process.

Alongside loading assets, the coroutine also does the following:

- Initialises Physics.gravity to (0.0, -40.0, 0.0)
- Calls Input.GetJoyButtons
- Calls Input.SetDefaultKeys

```cs
private static void LoadItemSprites()
```
A part of SetVariables.

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

```cs
private static TextMesh NewLetter()
private static TextMesh NewLetter(string id)
```
Creates a GameObject whose name is `letterX` where `X` is the value of `id` with a TextMesh that is setup as a SetText letter slot and is then returned. The parameterless version of the method is UNUSED, but remains functional and it calls the other overload with an `id` value of empty string.

This is primarily called as part of LoadEssentials when setting up the initial 500 letter slots in the `letterpool`, but it's also possible GetEmptyLetter calls it if it finds out that a letter slot is null.

A letter slot is a TextMesh that is setup with the following characteristics which are optimised for SetText usage:

- Childed to the MainCam (where MainManager resides) with a local z position of 10.0 on layer 5 (`UI`)
- richText is disabled
- fontStyle of Normal
- anchor set to LowerLeft
- MeshRenderer's shadowCastingMode set to Off
- MeshRenderer's allowOcclusionWhenDynamic disabled

```cs
public static Maps CurrentMap()
```
Returns `map`.`mapid`.

```cs
public static IEnumerator LateSound(string sound, float delay)
```
Yields for `delay` amount of seconds before calling PlaySound(`sound`) which plays the sound whose AudioClip is named `sound`.

```cs
public static bool IsPaused()
```
Returns true if any of the following conditions is true (false is returned otherwise):

- The game is in a `minipause`
- The game is in a `pause`
- The game is `inevent`
- A `battle` is in progress and it is `inevent`

```cs
public static SpriteRenderer Dimmer(int fadetype)
public static SpriteRenderer Dimmer(int fadetype, float fadespeed)
public static SpriteRenderer Dimmer(int fadetype, float fadespeed, Color color)
```
Calls PlayTransition(`fadetype`, 0, `fadespeed`, `color`) and then return `transitionobj[0]`'s SpriteRenderer. For the other 2 overloads, `fadespeed` will default to 0.1 and `color` defaults to pure black.

More concretely, this starts the transition TODO: explain when transitions are properly documented

```cs
public static void SetCamera(Transform target, Vector3? targetpos, float speed)
public static void SetCamera(Vector3 targetpos, float speed)
public static void SetCamera(Vector3 targetpos, Vector3 angle, Vector3 offset, float speed)
public static void SetCamera(Vector3 targetpos, Vector3 angle, float speed)
public static void SetCamera(Transform target, Vector3? targetpos, float speed, Vector3 offset)
public static void SetCamera(Transform target, Vector3? targetpos, float speed, Vector3 offset, Vector3 angle)
public static void SetCamera(Vector3 pos)
```
Configures the camera by setting a lot of camera related fields at once. These overloads are documented in detail in the camera system documentation.

```cs
public void SetVariables()
```
A part of LoadEverything which is a part of the boot process, but it is also called when confirming a language during Update (language selection listtype) and on StartMenu's Start.

This method initialises a lot of instance and static fields of MainManager to their initial sensible values. It also is where a lot of TextAsset's data is loaded via Resources.Load calls. This is also where SetUpBadges, ChangeParty({0, 1}, true), LoadItemSprites and InputIO.LoadSettings are called.

```cs
public static bool IsWithin(int value, int min, int max)
```
Returns true if `value` is both greater than or equal `min` and less than or equal `max`, false otherwise.

```cs
public static void ChangeParty(int[] ids, bool fromscratch)
public static void ChangeParty(int[] ids, bool fromscratch, bool destroyoldentity)
```
TODO: document the party system better and explain better how a change party works because it has wide implications on the playerdata addressing

```cs
public static void AddStatBonus(StatBonus type, int ammount, int to)
```
Adds a new element to the `statbonus` list using the `type`, `ammount` and `to` values provided. `statbonus` contains a list of permanent stats bonuses to apply from the base stats when cal;culating the party's stats. They are saved to the save file.

```cs
public static void PushAway(Transform obj, Vector3 otherobj)
public static void PushAway(Transform obj, Vector3 otherobj, float value)
```
Adds a vector to the `obj`'s postion corresponding to `obj`'s position - otherobj all normalised and multiplied by `value` (this essentially moves `obj` to be away from `otherobj` with a distance of `value`). On the overload without a `value` parameter, the value defaults to 0.05.

```cs
private static void ResetStats()
```
Resets every `playerdata`'s `basehp`, `baseatk` and `basedef` to their base value and reset `basetp` to its base value. The base values are the following:

- `basehp`: 7 (9 if the `playerdata`'s `trueid` is 1 which is `Beetle`)
- `baseatk`: 2
- `basedef`: 0
- `basetp`: 10

```cs
public static void ApplyStatBonus()
```
Does the following to apply all of the `statbonus` on top of the base party stats:

- Calls ResetStats
- Apply all of the `statbonus`
- If `statbonus` wasn't empty, calls ApplyBadges

```cs
public static bool HasPlayer(int id)
```
Returns true if any `playerdata` has a `trueid` of `id`, false otherwise.

```cs
public static bool PartyLowHP()
```
UNUSED: No one calls this, but it remains functional.

Returns true if any `playerdata` has an `hp` of 4 or below, false otherwise.

```cs
public static void RefreshWind(Renderer t)
```
Set the following float shader properties of the material of `t`:

- `_ShakeDisplacement`: Random between `map`.`windspeed` / 2.0 and `map`.`windspeed`. NOTE: This property is unused in the wind shader so this does nothing, the intended property was likely `_ShakeWindspeed`
- `_ShakeBending`: Random between `map`.`windintensity` / 2.0 and `map`.`windintensity`
- `_ShakeDisplacement`: Random between 0.075 and 0.25

```cs
private static BattleData SetDefaultStats(int id)
```
A part of ChangeParty.

Returns a BattleData with the following fields:

- `animid` and `trueid`: `id`
- `hp`, `maxhp` and `basehp`: 7 (9 if `id` is 1 which is `Beetle`)
- `atk` and `baseatk`: 2
- `skills`: new empty list
- `cursoroffset`: 2.3 in y
- `entityname`: `menutext` at index 46 + `id` (this is basically the language specific party member name of `Bee`, `Beetle` and `Moth`)
- `condition`: new empty list

```cs
public static void HurtParticle(Vector3 pos, bool playsound)
```
Calls HitPart(`pos`) and if `playsound` is true, this if followed by a call to PlaySound(`Hurt`).

```cs
public static BattleData GetPlayerData(int id, bool frombattleentity)
public static BattleData GetPlayerData(int id)
```
Returns the first `playerdata` whose `trueid` is `id` or the first `playerdata` if none exists.

If `frombattleentity` is true however, the method will first search for a `playerdata` whose `battleentity` has a `battleid` of id and it will return it if one is found. Otherwise, the regular search is done.

On the overload that doesn't take a `frombattleentity`, the default value is false.

```cs
public static BattleData? GetPlayerDataNullable(int id)
```
Returns the first `playerdata` whose `trueid` is `id` or null if none exists.

```cs
public static void AddPrizeMedal(int id)
```
Make the prize medal id `id` available to be obtained or purchased. More precisely, the following is done:

- If `DoublePain` medal is equipped or flag 614 is true (HARDEST is enabled):
    - flagvar of `prizeflags[id]` is set to 1 (obtainable by talking to Artis)
    - flag 56 is set to true (puts a ! mark above Artis)
- Otherwise:
    - flagvar of `prizeflags[id]` is set to 2 (obtainable by purchasing the medal at a Caravan shop)
- flagvar 55 is incremented (amount of prize medals avaible or obtained)

```cs
public static int HasBadge(int badgeid)
```
UNUSED: No one calls this, but it remains functional.

Returns the amount of times the medal whose id is `badgeid` is equipped to anyone. More precisely, it checks the amount of `badges` elements whose medal id is `badgeid`.

This is presumably a previous implementation of BadgeIsEquipped.

```cs
public void SetUpBadges()
```
A part of SetVariables.

Set `badgeshops[0]` (Merab) and `badgeshops[1]` (Shades) to their starting list and also set their matching `avaliablebadgepool` to be new lists containing the same elements. Here are the starting list of medals set:

- Merab: `HPPlus`, `TPPlus`, `PoisonResistance`, `SleepyResistance`, `DoublePainReal`, `Ambush`, `Pip`, `LastWind`, `ItemRecycle`, `BadDream`
- Shades: `SuperBlock`, `PoisonAttacker`, `PoisonDefender`, `StatusBoost`, `EXPBoost`

```cs
public static void AddQuest(int id)
```
Calls ChangeBoardQuest(`id`, 0) which essentially "moves" the quest to the open board, but in this case, it more adds it from nowhere to it. This is used in the AddQuest SetText command and it is part of CheckQuests when the quest's requirements are fufilled.

```cs
public static void CheckQuests()
```
Check every `questchecks` elements whose first requirements isn't 0 (no requirements) and whose quest id doesn't appear on any `boardquests`. For each of them, if all of the corresponding requirements are fufilled, AddQuest is called with the matching quest id.

A `questchecks` requirement is defined by the following (the value 0 is valid, but it can't be the first requirement in the list since that will count as no requirements):

- Greater than 0: The flag slot of the requirement number needs to be true
- 0 or below: The area whose id is the absolute value of the requirement number needs to have its corresponding `librarystuff[4]` flag set to true (meaning the area id was added to the world map)

```cs
public static IEnumerator TempIgnoreCollision(Collider a, Collider b, float seconds)
```
Ignores all collisions between `a` and `b` for `seconds` amount of seconds before uniginoring all collisions between `a` and `b`.

```cs
public static void LaunchObject(Transform obj, Vector3 push)
public static void LaunchObject(Rigidbody r, Vector3 push, bool gravity)
```
Set some fields of `r` to launch `r` or `obj`'s RigidBody:

- velocity: `push`
- isKinematic: false
- useGravity: `gravity`

On the overload without a `gravity` parameter, the default value is true.

```cs
public static void UpdateArea(int newarea)
```
Performs the tasks needed to process an area change which happens if needed on a MapControl's Start. More precisely, the following is done:

- All regional flags are reset
- `areaid` is updated to `newarea`
- The matching `librarystuff[4]` area flag of `newarea` is set to true which marks it as visible on the world map
- TODO: recheck setting flag 597 to true

```cs
public static void UpdateShops()
```
Set all `avaliablebadgepool` to each be a RandomSort version of their matching `badgeshops`. This happens on MapControl's Start, NPCControl's SetBadgePool(true) for a medals shop and during event 22 (loading a save file) when upgrading the save file to 1.2.0.

Additionally, for `avaliablebadgepool[0]` (Merab) specifically, if flag 587 is true (bought all Merab's medals), but `badgeshops[0]` isn't empty, flag 587 is reset to false. This corrects an inconsistency which can happen on a freshly upgraded 1.2.0 save file where the flag saying all medals were bought is true, but there's still some to be bought after the upgrade.

```cs
public static bool ObjectsAreActive(int[] objects, bool any)
public static bool ObjectsAreActive(NPCControl[] objects, bool any)
```
If `any` is true, returns true if any `objects` have a `hit` value of true, returns false otherwise.

If `any` is false, returns true only if all `objects` have a `hit` value of true, returns false otherwise.

In the `int[]` overload, the integers reprents the map entity ids of the NPCControl to use so they must be valid map entity for the current map or an exception will be thrown.

```cs
public static void RandomSort(ref List<int> array)
public static void RandomSort(ref int[] array)
```
Changes `array` such that it is randomly shuffled using the Fisher–Yates algorithm. On the `List<int>` overload, `array` is set to a newly created list contained the shuffled elements.

```cs
public static IEnumerator FlipSpriteBool(SpriteRenderer sprite, bool x, bool y, float everyxframes, float duringxframes)
```
A coroutine that will causes `sprite` to have their flipX toggled if `x` is true and their flipY toggled if `y` is true every `everyxframes` amount of frames for a total duration of `duringxframes`. If `duringxframes` is -1, the duration is infinite and the coroutine runs indefintely until it is manually stopped.

```cs
public void DoClock()
```
Perform tasks once an in game timer clock tick has passed which is normally every second (the method is designed to be called using InvokeRepeating every 1.0 second which is typically setup by EventControl).

TODO: possibly document this system separately, there's a lot of important stuff going on here notably, the Resources.UnloadUnusedAssets calls spams

```cs
public void RefreshPlayer(bool onlycollider = false)
```
Reset various physical related state to the `player` and `playerdata`:

- `player`.`ceiling` is set to false
- If `onlycollider` is false, every `playerdata` whose `entity`.`noclock` is false gets their entity rooted to the scene and their `entity`.`onground` set to false
- If the game isn't in a `battle` and not `inevent`, ForceHitWall is called on `player`.`entity`

```cs
public static bool IsParty(Transform obj)
```
Returns true if `obj` is a `playerdata`'s `entity`'s transform, false otherwise.

```cs
public static bool FreePlayer(bool getfly)
public static bool FreePlayer()
```
Only returns true if all of the following conditions are fufilled (returns false otherwise):

- `player` exists (meaning a map isn't being loaded)
- The game isn't in a `minipause`
- The game isn't in a `pause`
- The game isn't `inevent`
- The SetText `message` lock isn't active
- The `player` isn't `digging` or using the submarine's sonar
- If `getfly` is true, the `player` also needs to not be `flying` (otherwise, this condition doesn't apply)

On the overload without `getfly`, the default value is true.

```cs
public void CheckAchievement()
```
Verify and perform the necessary steps on a bunch of hardcoded conditions that may require an UpdateJournal call, a flag slot update or a call to InputIO.Achivement.

This method does nothing if the first LateUpdate of MainManager hasn't ran yet.

TODO: document the specific list of conditions and tasks in a separate page

```cs
public static bool SamiraGotAll()
```
Returns true if PurchasedMusicAmmount's return value is the amount of Musics enum values - 8, false otherwise. 8 is a hardcoded value that represents the amount of Musics enum values whose music aren't purchasable from Samira.

```cs
public static int PurchasedMusicAmmount()
```
A part of SamiraGotAll.

Returns the amount of `samiramusics` elements which aren't marked as "not bought" (assuming it implies being bought).

```cs
public static bool[] GetLibraryBools(int page)
```
Returns a new array whose elements is `librarystuff[page]` meaning the 256 flags array matching the `page`.

```cs
public static int HowManyTrue(bool[] array)
```
Returns the amount of elements in `array` that have a value of true.

```cs
public static bool InCameraRange(Transform obj)
public static bool InCameraRange(Vector3 t)
```
Returns true if `t` is a position that is considered inside the camera's range by considering `entityactive`, false otherwise. More precisely, the method only returns true if all of the following conditions are true:

- `t`.x is between -`entityactive`.x and `entityactive`.x + 1.0
- `t`.y is between -`entityactive`.y and `entityactive`.y + 1.0
- `t`.z is greater than `entityactive`.z TODO: more precisely explains this, but it likely means that `t` isn't too forward in the scene

The overload that takes a Transform is UNUSED as no one calls it, but it remains functional where `t` becomes `MainCamera`.WorldToViewportPoint(`obj`'s position).

```cs
public static AudioSource PlaySound(string soundclip)
public static AudioSource PlaySound(string soundclip, int id)
public static AudioSource PlaySound(string soundclip, int id, float pitch, float volume)
public static AudioSource PlaySound(string soundclip, int id, float pitch, float volume, bool loop)
public static AudioSource PlaySound(string clip, float volume)
public static AudioSource PlaySound(string clip, float pitch, float volume)
public static AudioSource PlaySound(AudioClip soundclip)
public static AudioSource PlaySound(AudioClip soundclip, int id)
public static AudioSource PlaySound(AudioClip soundclip, int id, float pitch)
public static AudioSource PlaySound(AudioClip soundclip, int id, float pitch, float volume)
public static AudioSource PlaySound(AudioClip soundclip, int id, float pitch, float volume, bool loop)
```
Plays the `soundclip` AudioClip using the `sounds[id]` AudioSource with a pitch of `pitch`, volume of `soundvolume` * `volume` and a loop property of `loop` then returns the AudioSource where the AudioClip started playing as well as setting `lastsoundid` to `id`. In practice, `id` can only be from 0 to 14 as `sounds` has a fixed size of 15 enforced by LoadEssentials.

If SoundVolume returns false (meaning sounds are muted), the method returns null instead. NOTE: It does NOT return an empty AudioSource, this means the return value needs to be null checked to avoid an exception being thrown.

If `id` is -1, the value to use as the id is determined automatically using the GetFreeSound method.

For the overloads taking a string instead of an AudioClip, `soundclip` becomes the result of loading the AudioClip at `Audio/Sounds/X` where `X` is the string provided.

For overloads not taking every parameters, here are their default values:

- `id`: -1
- `pitch`: 1.0 (normal)
- `volume`: 1.0 (normal)
- `loop`: false

```cs
private static int GetFreeSound()
```
Returns the first `sounds` index whose AudioSource's `isPlaying` is false. If every `sounds` elements are playing something, the last `sounds` element is returned even if it is playing an AudioClip.

```cs
public static void StopSound(AudioClip clip, float delay)
public static void StopSound(AudioClip clip)
public static void StopSound(string clipname)
public static void StopSound(string clipname, float delay)
```
Calls Stop on every `sounds` whose clip is `clip` to stop playing the AudioClip on any of these AudioSource. If `delay` is above 0.0, a FadeSound coroutine is started instead with the matching `sounds` element and the `delay` value to smoothly fade the clip.

If SoundVolume returns false (meaning sounds are muted), this method does nothing.

On overload that takes a string instead of an AudioClip, the AudioClip is obtained by loading the asset at `Audio/Sounds/X` where `X` is the string provided.

On overloads without a `delay` parameter, the default value is 0.0.

```cs
public static void StopSound(int clip)
public static void StopSound(int clip, float delay)
```
Calls Stop `sounds[clip]` to stop playing the AudioClip on the AudioSource. If `delay` is above 0.0, a FadeSound coroutine is started instead with `sounds[clip]` and the `delay` value to smoothly fade the clip.

If SoundVolume returns false (meaning sounds are muted), this method does nothing.

On the overload without a `delay` parameter, the default value is 0.1.

```cs
public static bool SoundVolume()
```
Returns true if sounds aren't muted, false otherwise. More precisely, the method returns true if any of the following conditions is true:

- `soundvolume` is above 0.0 and there's no `pausemenu` displayed
- `soundvolume` is above 0.0 and there is a `pausemenu` displayed, but its `windowid` isn't 4 (not on the settings window)
- There is a `pausemenu` displayed with a `windowid` is 4 (settings window) and `pausemenu`.`svolume` is above 0.0. This essentially means that if the game is on the PauseMenu's settings window, the working value of the sound volume setting must not be muted and it takes precedence over `soundvolume` in these conditions

```cs
public static Vector3 ClampVectorBox(Vector3 input, Vector3 limitspos, Vector3 limitsneg)
public static Vector3 ClampVectorBox(Vector3 input, Vector3 limits)
```
Returns a vector where each component of `input` got clamped from the matching component of `limitneg` to the matching component of `limitspos`.

On the overload taking a `limits` parameter, `limitspos` becomes `limits` and `limitsneg` becomes -`limits`.

```cs
public static Vector3 ClampMagnitude(Vector3 v, float max, float min)
```
Returns a vector that matches `v` in direction, but its magnitude got clamped from `min` to `max`. More precisely, the vector that gets returned depends on the following conditions:

- If `v`.sqrMagnitude is higher than `max` * `max`, `v`.normalized * `max` is returned
- If `v`.sqrMagnitude is lower than `min` * `min`, `v`.normalized * `min` is returned
- Otherwise, `v` is returned unchanged

```cs
public static IEnumerator LerpObject(Transform obj, Vector3 position, float speed, bool destroyonend)
```
Moves `obj` via a Lerp from its current position to `position` with a factor that grows from 0.0 to 1.0 at a rate of TieFramerate(`speed`) each frame. Once the Lerp completed, `obj` gets destroyed if `destroyend` is true.

```cs
public static void ChangeMusicVolume(float frametime)
public static void ChangeMusicVolume(float targetvolume, float frametime)
public static void ChangeMusicVolume(int clipid, float targetvolume, float frametime)
```
Stops `musiccoroutine` if it was in progress and set it to a new ChangeMVolume(`clipid`, `targetvolume`, `frametime`) coroutine. This coroutine will smoothly change `music[clipid]`.volume towards `targetvolume` over the course of `frametime` amount of frames. NOTE: In practice, `clipid` should always be 0 since `music` only contains 1 AudioSource.

The overloads that doesn't take every parameters are UNUSED, but remains functionals where `clipid` defaults to `musicchannel` and `targetvolume` defaults to `musicvolume`.

```cs
private static IEnumerator ChangeMVolume(int clipid, float volume, float frametime)
```
Smoothly changes `music[clipid]`.volume via a Lerp from the current volume to `volume` over the course of `frametime` amount of frames + 1 frame. Once the Lerp completes and the additional frame yield, the volume is set to `volume`. NOTE: In practice, `clipid` should always be 0 since `music` only contains 1 AudioSource.

```cs
private static IEnumerator SwitchMusic(AudioClip musicclip, float fadespeed, int id)
private static IEnumerator SwitchMusic(AudioClip musicclip, float fadespeed, int id, bool seamless)
```
TODO: Detail the music system separately since this is complex

The overload without a `seamless` parameter is UNUSED since the coroutine is never called, but it remains functional and the value of the parameter will default to false with an extra frame yield after calling the other coroutine.

```cs
public static void CheckSamira(AudioClip music)
public static void CheckSamira(string music)
```
TODO: Detail the samira portion of musics since this is complex

```cs
public static void ChangeMusic(AudioClip musicclip, float fadespeed, int id)
public static void ChangeMusic(string musicclip, float fadespeed, int id, bool seamless)
public static void ChangeMusic(AudioClip musicclip, float fadespeed, int id, bool seamless)
public static void ChangeMusic(string musicclip, float fadespeed)
public static void ChangeMusic(AudioClip musicclip, float fadespeed)
public static void ChangeMusic(string musicclip, float fadespeed, int id)
public static void ChangeMusic()
public static void ChangeMusic(int mapid)
public static void ChangeMusic(string musicclip)
public static void ChangeMusic(AudioClip musicclip)
```
TODO: Detail the music system separately since this is complex

The overload with parameters (string, float, int) is UNUSED since the coroutine is never called, but it remains functional.

```cs
public static int SoundIsPlaying(string name)
```
Returns the index of the first `sounds` that `isPlaying` and whose clip.name is `name`. If no `sounds` is found with these conditions, -1 is returned.

```cs
private static bool AnyJoyKey(bool hold)
```
TODO: Involves InputIO logic that's not documented yet

```cs
private static bool AnyJoyStick()
```
TODO: Involves InputIO logic that's not documented yet


```cs
private void GetJoystick()
```
TODO: Involves InputIO logic that's not documented yet


```cs
private static void ShowBackDialogue()
```
Destroys all the child of `textbox` and calls SetText in non dialogue mode to show `|color,7|` followed by `diagstring[currentdialogue]` (where all double and triple spaces are removes as well as removing the leading space if one is present). This method is used when a backtracking is requested on Update during the course of a SetText call in dialogue mode.

```cs
private static bool HasSkillCost(int tpcost, int playerid)
```
Returns true if `tp` is at least `tpcost`, false otherwise.

However, if `playerdata[playerid]` has the `HPFunnel` medal equipped or if `tpcost` is negative, this instead returns true when `playerdata[playerid]`.`hp` - 1 is at least the absolute value of `tpcost`, false otherwise.

This method is intended to check if the player is able to pay the cost of a skill whether it is in TP (`tpcost` isn't negative) or HP (`tpcost` is negative or `playerdata[playerid]` has the `HPFunnel` medal equipped). In the case the cost is in HP, the HP cost is the absolute value of `tpcost` - 1.

```cs
public static IEnumerator Spin(Transform obj, Vector3 target, float frametime, bool smooth)
```
Changes `obj` angles towards `target` over the course of `frametime` + 1.0 amount of frames via a Lerp. If `smooth` is true, a SmoothLerp is used instead.

```cs
public static bool KeyHold(Directions direction)
```
TODO: Involves InputIO logic that's not documented yet

```cs
public static void ResetKeyHold()
```
TODO: Involves InputIO logic that's not documented yet

```cs
private static bool MultiItem()
```
Returns true if the current ItemList resulted in multiple items being selected in `multiselect`, false otherwise. More precisely, it returns true if `multiselect` isn't empty and at least one of the following condition is true:

- listtype is 2 (storage items)
- listtype is 0 (standard items) and flag 349 is true (currently using the Ant Storage Service)
- listsell is true (current ItemList is an items shop ItemList)

```cs
private static bool IsInMultiList()
```
Returns true if the current ItemList supports multiselect, false otherwise. More precisely, it returns true if at least one of the following condition is true:

- listtype is 2 (storage items)
- listtype is 0 (standard items) and flag 349 is true (currently using the Ant Storage Service)
- listsell is true (current ItemList is an items shop ItemList)

```cs
public static bool IsPlayerInPos(int pos, Transform entity)
```
Returns true if `playerdata[battle.partypointer[pos]]`.`battleentity` is `entity`, false otherwise.

```cs
public static IEnumerator TempColor(Color color, float frametime, SpriteRenderer render)
```
Smoothly changes the material.color of `render` towards `color` over the course of `frametime` + 1.0 amount of frames, yields an additional frame and then set the `render`'s color back to where it was before this coroutine.

This coroutine is meant to be stored in `templetter` because it is set to null at the end of this coroutine so this can be used to track its progress.

```cs
public static void SetFont(TextMesh letter, int fontid)
```
Configure `letter` such that it uses the font specified by `fontid` including setting the proper material (stored in `fontmat`) and the font shader (`fontmat[0]/shader`) on the MeshRenderer TODO: why the first fontmat's shader specifically ???. More precisely, the font used will be `fonts[x]` where `x` is FontID(`fontid`) and it will also dictate the `fontmat` to use in the same manner. FontID will map `fontid` such that:

- Any negative numbers maps to their absolute value + 1
- 2 (UNUSED) maps to 1 (`D3Streetism`)
- Any other numbers of 0 or above are unchanged

```cs
public static IEnumerator TempLetters(string text, int[] spacebreaks, bool rainbow, Vector3 posLeave0ToCenter, float showtime)
```
This is UNUSED and left in a broken state, but it can technically be called. It is NOT recommended to call this coroutine.

Temporarily renders `text` using newly allocated letter slots for `showtime` amount of seconds with a rainbow font effect if `rainbow` is true. The `spacebreaks` and `posLeave0ToCenter` parameters don't do anything and their values should always be null and Vector3.zero respectively.

This coroutine is broken because the letter slots used aren't configured properly (wrong positioning, wrong font and wrong font sizes). This results in the text not rendering properly with the default Unity font.

```cs
public static IEnumerator GradualColor(Renderer obj, Color target, float frametime)
public static IEnumerator GradualColor(SpriteRenderer obj, Color target, float frametime, bool sprite)
```
Changes `obj`.material.color (or `obj`.color for a SpriteRenderer) towards `target` via a Lerp over the course of `frametime` + 1.0 amount of frames. The `sprite` parameter is unused.

```cs
private static void UpdateQuestBoard()
```
TODO: Document the QuestBoard handling separately as it seems complex

```cs
private void CloseQuestBoard()
```
TODO: Document the QuestBoard handling separately as it seems complex

```cs
private void DestroyList()
```
Destroys the current ItemList to stop its rendering with a shrink animation done over 0.5 seconds.

This coroutine also does the following additionally:

- Set `itemList` to null
- Destroys `cursor`
- Reset `skiptext` to false
- `listoption` is reset to `option`
- `option` is reset to -1
- `listY` is reset to -1

NOTE: Calling this doesn't necessarily mean the ItemList is done being processed. It only means that the UI will no longer show, but it's possible the game needs to process the outcome of the ItemList after. This happens for example with SetText and a PickItem command where SetText needs to process the outcome after `itemList` has been set to null. This is an important distinction for the `inlist` issue.

```cs
private void RefreshNumberPrompt()
```
Destroys and recreate the text rendered as the working output of a numberprompt or letterprompt. More precisely, this calls DestroyText(`npromptholder`) followed by calling SetText in non dialogue mode with `|center|` + the current output (stored in a flagstring) padded with `_` to its max length (stored in flagvar 10) with `npromptholder` as the parent.

This needs to be called whenever a change in the output text needs to be reflected in the rendering.

```cs
public static float ClampedAngle(float input, float max)
```
This is UNUSED and is considered broken. If called, it performs the following:

- If `input` is negative, `max` is added to it
- As long as `input` is higher than `max`, `input` is set to `max` - `input`
- `input` is returned

```cs
public static BattleData GetEnemyData(int id)
public static BattleData GetEnemyData(int id, bool createentity)
public static BattleData GetEnemyData(int id, bool createentity, bool noexp)
```
Returns a BattleData whose data matches the ones of a new enemy with id `id`.

`noexp` being true will result in the BattleData's `exp` to be 0 even if it otherwise would not be.

`createentity` being true means `battleentity` will be created and its fields will be configured accordingly to the enemy data.

The overload without a `noexp` parameter will have its value default to false.

The overload with just the `id` parameter is UNUSED, but remains functional and the `createentity` parameter will default to to false.

```cs
public static float GetSoundDistance(Vector3 position)
public static float GetSoundDistance(Vector3 position, float maxdistance)
```
Returns a number between 0.0 and 1.0 where 0.0 means `position` is `maxdistance` or more units away from `MainCamera` position and 1.0 means `position` is the same as `MainCamera` position. It's a method that's meant to be used to compute the volume a sound should have relative to the `MainCamera` so closer sounds louder than sounds located further away.

The overload without a `maxdistance` parameter has its value default to 25.0.

```cs
public static float GetSoundDistance(float distance, float maxdistance)
public static float GetSoundDistance(float distance)
```
Returns a number between 0.0 and 1.0 where 0.0 means `distance` is `maxdistance` or more and 1.0 means `distance` is 0.0 or lower. It's a method that's meant to be used to compute the volume an entity should have when comparing its `camdistance` to `sounddistance` so entities located closer sounds louder than entities located further away.

The overload without a `maxdistance` parameter is UNUSED, but rmains functional and it has `maxdistance` default to 25.0.

```cs
public static void PlayMoveSound(EntityControl entity)
public static void PlayMoveSound(int animid, int soundid, bool fly)
```
Either calls PlaySound using `id` as the id with loop or does nothing depending on the `animid` / `entity.originalid` and the value of `fly`. If PlaySound is called, the soundclip, pitch and volume are determined depending on the `animid` / `entity.originalid` where the sound represents the appropriate movement sound to play whenever any entity whose animid is `animid` moves.

The overload with just an `entity` parameter has the `soundid` value default to 9 and `fly` value default to whether or not `entity`.`height` is higher than 0.1 (meaning they have a visual offset in y).

Here are the specific properties for each supported value of `animid` (if the value isn't in this list, PlaySound is not called and the method does nothing):

- `Scorpion`: `Step` with 1.0 pitch and volume
- `DivingSpider`: `WetStep` with 1.0 pitch and volume
- `FlyTrap` and `Krawler`: `Scuttle3` with 1.0 pitch and volume
- `Armorpillar`: `FunnyStep` with 0.9 pitch and 0.3 volume
- `Mushroom`: If `fly` is true, `Fly` with 0.5 pitch and 1.0 volume, no PlaySound call otherwise
- `Seedling` and `Flowering`: If `fly` is true, `Toss2` with 1.1 pitch and 0.5 volume, no PlaySound call otherwise
- `Thief`, `Midge` and `Mantidfly`: If `fly` is true, `BugWing` with 1.0 pitch and volume, no PlaySound call otherwise
- `Spuder`: `Scuttle2` with 0.9 pitch and 0.8 volume
- `MimicSpider`: `Scuttle2` with 1.1 pitch and 0.8 volume
- `Plumpling`: `ThumpSoft` with 0.65 pitch and 0.65 volume
- `CordycepsAnt`: `Scuttle` with 0.5 pitch and 0.8 volume
- `Abomihoney`: `PuddleMove` with 1.15 pitch and 1.0 volume

```cs
public static IEnumerator ForceFailsafe(Transform obj, Vector3 target, float cooldown)
```
This is UNUSED, but remains functional. If after waiting for `cooldown` amount of frames, the GetDistance between `obj` position and `target` is greater than 0.45000002, the following happens (the coroutine does nothing useful otherwise):

- DeathSmoke(`obj`.position) called
- `obj`.position set to `target`
- DeathSmoke(`target`) called

```cs
public static int GetEXP(int input)
public static int GetEXP(int input, int level)
public static int GetEXP(int input, Enemies enemy)
public static int GetEXP(int input, int lv, Enemies? enemy)
```
Calculates and returns the scaled EXP that `enemy` would give when the player is at rank `lv` and the unscaled EXP value is `input`. Check the GetEXP documentation for the details on how this method calculates the scaled EXP. The enemy scaling part doesn't apply if `enemy` is null, but this never happens under normal gameplay.

The default `lv` value for the third overload is `partylevel`.

The first 2 overloads are UNUSED, but remains functional and works like this:

- First overload: `lv` is `partylevel` and `enemy` is null
- Second overload: `lv` is `level` and `enemy` is null

```cs
public static void PlayBuzzer()
```
Calls PlaySound(`Buzzer`, 10).

```cs
public static void Heal()
public static void Heal(bool noparticle, bool nosound)
public static void Heal(Healing[] parameters, int[] partyids, bool noparticle, bool nosound)
```
Heals the `playerdata` whose index is in `partyids` where the healing is determined by `parameters` and perform the associated visual / audio effects for the healing. If `nosound` is true, PlaySound(`Heal`) will not be called. If `noparticle` is true, HealParticle will never be called even if it would have been otherwise. This method also sets `hudcooldown` to 100.0 to show the party HUD temporarilly. If HealParticle gets called, the parent is the `playerdata`'s `entity` or `battleentity` depending if the game is in a `battle`.

For the first 2 overloads, the `parameters` value defaults to a single element being `Full`, `partyids` defaults to `partyorder` (which is every `playerdata`), `noparticle` defaults to false and `nosound` defaults to false.

Here are the healing effects of the different `parameters` value:

- `Full`: The `playerdata`'s `hp` is set their `maxhp` and the party's `tp` is set to `maxtp`. Particles are included if `noparticle` is false
- `TPOnly`: The party `tp` is set to `maxtp`. Particles are always excluded regardless of `noparticle`
- `FullHPOnly`: The `playerdata`'s `hp` is set their `maxhp`. Particles are included if `noparticle` is false

```cs
public static int ConditionTurns(int playerid, int turns)
```
Simply returns `turns`. `playerid` is UNUSED.

```cs
public static bool HasWeakness(BattleControl.AttackProperty property, BattleData entity)
```
Returns true if `entity`.`weakness` contains `property`, false otherwise.

```cs
public static void SetCondition(BattleCondition condition, ref BattleData entity, int turns, int fromplayer)
public static void SetCondition(BattleCondition condition, ref BattleData entity, int turns)
```
Adds or amend a condition to an actor. See the SetCondition documentation to learn more. The `fromplayer` parameter is UNUSED and the overload passes a value of -1.

```cs
private static void FixCondition(BattleData entity)
```
A part of SetCondtion. See the SetCondition documentation to learn more.

```cs
public static bool ConditionChance(BattleCondition condition, BattleData entity)
```
Determines via RNG if a condition should be inflicted on an actor. See the ConditionChance documentation to learn more. NOTE: This method results in infliction odds that are always off by 1.

```cs
public static int HasCondition(BattleData entity)
public static int HasCondition(BattleCondition condition, BattleData entity)
```
Returns the amount of main turns left on a condition if it is present on an actor. See the HasCondition documentation to learn more.

```cs
public static void RemoveCondition(BattleCondition condition, BattleData entity)
```
Removes a condition on an actor if it is present. See the RemoveCondition documentation to learn more.

```cs
public static SpriteRenderer GetTransitionSprite()
public static SpriteRenderer GetTransitionSprite(int id)
```
Returns the SpriteRenderer of `transitionobj[id]`. For the overload, `id` defaults to 0.

```cs
public static void PlayScrollSound()
```
If `sounds[10]` isn't playing anything, PlaySound is called to play the `Audio/Sounds/Scroll` AudioClip on `sounds[10]` with a pitch of 1.0 and a volume of 0.65. If this is called while in the `pausemenu` with a `windowid` of 4 or higher TODO: why "or higher" ??? , `sounds[10]` volume gets set to `svolume` so it follows the working value in the settings.

If `sounds[10]` was playing something when this is called, this method does nothing.

```cs
public void UpdateList(bool up)
public void UpdateList(bool up, int skip)
public void UpdateList(bool up, int skip, bool nosound)
public void UpdateList(Directions dir)
public void UpdateList(Directions dir, bool nosound)
```
Performs the necessary tasks and visual updates to reflect a change in the current `itemList` when the cursor moves. TODO: Document this in ItemList documentation if not done already

```cs
public static void ReadSettings(string[] c)
```
Set all fields backed by a setting value where `c` is the string array read by reading the `config.dat` file. TODO: Document config.dat format!

```cs
public static string SaveSettings()
```
Returns a `config.dat` compliant text from the settings fields. TODO: Document config.dat format!

```cs
public static string SaveFile(Vector3? savepos)
```
Returns a save file compliant text from the current state of MainManager. Check the save file format documentation to learn more.

```cs
public static bool Interval(float value, float min, float max, bool inclusive)
```
This is UNUSED, but remains functional.

Returns true if `value` is between `min` and `max`, false otherwise. If `inclusive` is true, a `value` of exactly `min` or `max` will count as being inside the intervale while if `inclusive` is false, it will not count as being inside.

```cs
private void RefreshDiscovery()
```
Updates the local position of the `discoverymessage` UI element according to `discoveryhud`. If `discoveryhud` is higher than 0.0, the local position is lerped from the existing one to (-8.0, -3.0, 0.0) with a factor of TieFramerate(0.1) which will smoothly reveal `discoverymessage`. Otherwise, the same lerp happens, but towards (-8.0, -6.0, 0.0) instead which will hide `discoverymessage`.

On the first time this is called, `discoverymessage` will be null which will cause this method to create it from scratch (any subsequent calls reuses the same object). It involves the `Prefabs/Objects/logbookicon` prefab and 3 SetText calls in non dialogue mode. After the creation, `discoverymessage` will be the parent of the prefab.

```cs
public static void UpdateOutlines()
```
Calls Shader.SetGlobalFloat to set `GlobalOutline` to a value that depends on `map`.`areaid` and the `enableoutline` setting. Here's the exact logic used:

- If `enableoutline` is 0 (OFF), the value is set to 0.0
- If `enableoutline` is 1 (LOW), the value is set to 0.1
- If `enableoutline` is 2 (HIGH), it depends if `map`.`areaid` is `MetalLake` or not:
    - If it is, the value is set to 0.1
    - If it is not, the value is set to 0.3

```cs
public static int MixIngredients(int item1, int item2)
```
Search `recipedata` for a compatible recipe where `item1` and `item2` matches the ingredients and returns the result of the matching recipe if one is found or 8 (`Mistake`) if no such recipe exists. To search of a single item recipe, `item2` needs to be -1. The method supports having `item1` and `item2` being reversed compared to the recipe for a double item recipe since it will normalise the order before searching.

If `item1` or `item2` is 156 (`Guarana`), no search is performed and instead, the result is a random result determined among the following:

- If `item2` isn't 156 (`Guarana`):
    - 1/13 to be `RoastBerry`
    - 2/13 to be `BigMistake`
    - 1/13 to be `BerryJuice`
    - 2/13 to be `BerrySmoothie`
    - 1/13 to be `Mistake`
    - 3/13 to be `CookedJellyBean`
    - 1/13 to be `TangyJam`
    - 1/13 to be `SpicyBomb`
    - 1/13 to be `BurlyBomb`
- Otherwise (`item1` is 156, `Guarana`):
    - 2/11 to be `RoastBerry`
    - 4/11 to be `Mistake`
    - 2/11 to be `BigMistake`
    - 1/11 to be `BerryJuice`
    - 1/11 to be `CookedJellyBean`
    - 1/11 to be `BerrySmoothie`

```cs
public static bool HasRecipe(int id)
```
This is UNUSED, but remains functional. Returns the flag value matching the recipe entry in `librarystuff` where `id` is the resulting item in the recipe. If no such recipe exists, false is always returned.

```cs
public static int GetRecipeID(int id)
```
Returns the recipe id from `libraryorder` where the resulting item id is `id` or -1 if no such recipe exists.

```cs
public static void ResetCamera()
public static void ResetCamera(bool instant)
```
Resets many camera related fields. For more information, check the camera system documentation.

```cs
private void ResetCamSpeed()
```
A part of ResetCamera(bool instant) where `camspeed` and `camanglespeed` are reset to 0.1.

```cs
public static bool PartyIsNotMoving()
```
Returns true if no `playerdata`'s `entity` are in a `forcemove`, false otherwise.

```cs
public static bool GetKey(int id)
public static bool GetKey(int id, bool hold)
```
TODO: Involves InputIO logic that's not documented yet

```cs
public static Vector2 GetPressedDirection(bool hold)
```
This is UNUSED, but remains functional. Returns a normalized vector that represents the accumulation of all 4 cardinal directions that are being pressed with up/down and left/right separation. This results in the following logic to determine the components of the vector tested in order (the vector gets normalised after all of this):

- x: -1.0 for left, 1.0 for right, 0.0 for neither
- y: 1.0 for up, -1.0 for down, 0.0 for neither

```cs
public static float DimishingReturns(float input, float decay)
```
This is UNUSED, but remains functionsl. Returns (`input` - 1.0) * `decay`.

```cs
public static float[][] LoopPoint()
```
Returns the data from `Data/LoopPoints` which is meant to be set to the value of `musicloop`. Check the music loop data documentation to learn more about the data format.

```cs
private void LoopMusic()
```
Checks the current `music` playing and if its end loop point in `musicloop` isn't 0.0 and its playback is past that end point, `music[0]`.time is set to the start loop point in `musicloop`.

The first time this is called, `musicloop` will be null so this method will initialise the value to the return of LoopPoint if that is the case.

```cs
public static int GetRandomMedal()
public static int GetRandomMedal(bool dontremove, bool random)
```
TODO: This seems simple, but annoying to parse, come back to this later to explain how this works

```cs
public static GameObject NewUIObject(string objname, Transform parent, Vector3 pos)
public static GameObject NewUIObject(string objname, Transform parent, Vector3 pos, Vector3 size, Sprite sprite)
public static GameObject NewUIObject(string objname, Transform parent, Vector3 pos, Vector3 size, Sprite sprite, int sortorder)
```
Creates and return a new GameObject that has the following properties:

- Childed to `parent` or `GUICamera` if `parent` is null
- Layer 5 (`UI`)
- No angles
- Local position of `pos`
- Scale of `size`
- If `sprite` isn't null, the GameObject will have a SpriteRenderer with a sprite of `sprite` and a sortingOrder of `sortorder`

On the first 2 overloads, here are what the parameters defaults to:

- `size`: Vector3.one
- `sprite`: null
- `sortorder`: 0

```cs
public static bool CheckIfCanExist(int[] requires, int[] limit, int regionalflag)
```
Returns false if all of the following conditions are satisfied which indicates that the conditions sent are satisfied (true otherwise indicating the conditions are NOT satisifed):

- Any flag slots that matches the absolute value of any `limit` elements of -2 or lower must be false
- If `requires` is considered not empty, all flags slots specified in `requires` must be true. The array is considered not empty if it is not null, not empty and its first element isn't negative
- If `limit` is considered not empty, all flags slots specified in `limit` must be false. The array is considered not empty if it is not null, not empty and its first element isn't negative
- If `regionalflag` applies, the regionalflag slot `regionalflag` must be false. The condition applies only if `regionalflag` isn't negative and `map` isn't null (meaning the map isn't being loaded)

This method implements the backing logic of a ConditionChecker and NPCControl's existence logic. NOTE: The return value is inverted: false if conditions are satisifed, true if conditions are NOT satisfied.

```cs
private static Vector3 CamLimiter(Vector3 pos)
```
Returns `pos` where each of its component was clamped with the matching components from `camlimitneg` to `camlimitpos`

```cs
public static Vector3 LimitRadius(Vector3 pos, Vector3 origin, float radius)
public static Vector3 LimitRadius(Vector3 pos, Vector3 origin, float radius, bool ignoreY)
```
Returns a vector that is `origin` + a vector whose y component is `pos`.y and the x/z components are the ones obtained from a ClampMagnitude vector of `pos` - `origin` with a maxLength of `radius`. If `ignoreY` is true, the resulting vector's y component is always `pos`.y instead of being `origin`.y + `pos`.y.

The first overload doesn't implement the `ignoreY` logic so it's as if its value was false.

The intuitive way to see this method is that the return is `pos`, but limited by magnitude to be within a circle with an origin of `origin` and a radius of `radius`. Having `ignoreY` as true means the y componet of `pos` won't apply to this radius limit.

```cs
private void RefreshCamera()
```
The main camera update method. Check the camera system documentation to learn more.

```cs
public static void SaveCameraPosition()
public static void SaveCameraPosition(bool set)
```
Saves or loads the camera state. Check the camera system documentation to learn more.

```cs
public static void LoadCameraPosition()
```
Loads the camera state that was saved previously with SaveCameraPosition(true). Check the camera system documentation to learn more.

```cs
public static Vector3 LerpVectorAngle(Vector3 input, Vector3 target, float ammount)
```
Returns a vector where each of its component is the result of a LerpAngle from `input` to `target` with a factor of `ammount` using matching components for all vectors.

```cs
public static Vector3 LerpVectorAngleSmooth(Vector3 input, Vector3 target, Vector3 current, float ammount)
```
This is UNUSED, but remains functional. Returns a vector where each of its component is the currentVelocity output of a SmoothDampAngle from `input` to `target` with a smoothTime of `ammount` using matching components for all vectors.

```cs
public static void RebuildHUD()
```
Does the following which destroys the existing HUD:

- Calls ApplyBadges
- Calls ApplyStatBonus
- Destroys `hud[0]`'s parent if `hud[0]` exists
- Reset `hud` to null

```cs
private static void CreateHUD()
```
A part of RefreshHUD (when `hud` is null or empty). Calls ApplyBadges and ApplyStatBonus followed by the initialisation of various HUD related fields. These fields includes:

- `hud`
- `hudsprites`
- `hudfont`
- `hudvalue`

```cs
private static void NewHUDElement(int hudid, Vector2 position, Transform parent)
```
Part of CreateHUD. Initialises `hud[hudid]` with a local position of (`position`.x, `position`.y, 10.0) childed to `parent`. TODO: Detail what each hudid represents here

```cs
public static float QuadraticY(float x1, float x2, float ymax, float currentx)
public static float QuadraticY(Vector2 x1, Vector2 x2, float ymax, float currentx)
```
Returns the y component of the vector given by BeizierCurve(`x1`, `x2`, `ymax`, GetPercentage(`x1`.x, `x2`.x, `currentx`)).

The first overload is UNUSED, but remains functional where `x1` and `x2` becomes vector where their x component is the matching float sent and their y/z components are 0.0.

```cs
public static string GetItemString(bool emptyinv)
```
Returns a string that represents a `,` separated list of all item id in `items[0]` (standard items inventory). If `emptyinv` is true, `items[0]` is reset to a new empty list before returning the string which empty empties the entire standard items inventory.

```cs
public static Vector2 BeizierCurve(Vector2 start, Vector2 end, float ymax, float t)
```
TODO: explain the math here, it's complex

```cs
public static bool EntitiesAreNotMoving(EntityControl[] entities)
```
Returns true if no entity in `entities` are in a `forcemove`, false otherwise.

```cs
public static string GetBleep(EntityControl entity)
public static string GetBleep(EntityControl entity, float volume)
```
Returns a string that forms a valid bleep SetText command with the following parameters:

- sound: `entity`.`dialoguebleepid`
- pitch: `entity`.`bleeppitch`
- volume: `volume`

```cs
public static Vector3 BeizierCurve3(Vector3 start, Vector3 end, float ymax, float t)
public static Vector3 BeizierCurve3(Vector3 start, Vector3 end, Vector3 mid, float t)
```
TODO: explain the math here, it's complex

```cs
public static bool AsianLang()
```
Returns true if `languageid` is 3 (Japanese), false otherwise.

```cs
public static float BeizierFloat(float height, float t)
```
TODO: explain the math here, it's complex

```cs
public static float QuadraticYSemiClamped(float currentx, float startx, float endx, float modifier)
```
This is UNUSED, but remains functional. TODO: explain the math here, it's complex

```cs
public static float QuadraticYUnclamped(float currentx, float startx, float endx, float modifier)
```
TODO: explain the math here, it's complex

```cs
public static Vector3 RandomVector(Vector3 input)
public static Vector3 RandomVector(float randomx, float randomy, float randomz)
public static Vector3 RandomVector(float input)
public static Vector3 RandomVector(Vector2 input)
public static Vector3 RandomVector(float randomx, float randomy)
```
Returns a vector where each components is determined randomly between -`input` and `input` with matching components for all vectors involved.

For the second overload, it's as if the `input` vector was (`randomx`, `randomy`, `randomz`).

For the third overload, it's as if each components of `input` was the float value sent.

The fourth overload is UNUSED, but remains functional and it simply doesn't generate a vector with a z component, but otherwise works the same way.

The fifth overload behaves similarily to the second overload, but it doesn't generate a vector with a z component.

```cs
public static float ColorDifference(Color a, Color b)
```
Returns the magnitude of a vector whose components is the RGB difference between `a` and `b` (x is the red difference, y is the green difference and z is the blue difference).

```cs
public static void GlobalCommand(ref string text)
```
Replaces the global commands placeholder text in `text` to their actual commands. This is an unused system, check the global commands system documentation to learn more.

```cs
private static void ForceHUD()
```
Sets all `tphp`, `ptmhp`, `tmtp` values and `tempmoneh` to -1.

This essentially forces a call to RefreshHUD on the next LateUpdate because it will find that everything the HUD should display has changed.

```cs
private static void RefreshHUD()
```
TODO: possibly document the HUD system separately as this gets complex

```cs
public static void ShowHUD(bool show)
```
TODO: possibly document the HUD system separately as this gets complex

```cs
public static Vector3 MiddlePoint(Vector3[] inputs)
```
This is UNUSED, but remains functional. Returns a vector that is all `input` added together / the length of `inputs`.

```cs
public static int CheckIfMore(int value, int[] values)
```
This is UNUSED, but remains functional. Returns the amount of `values` element that is less than `value`.

```cs
public static Vector3[] GetEntitiesPos(EntityControl[] e)
```
Returns an array where each element is the position of the matching `e`'s entity.

```cs
public static void RefreshSkills()
```
Refresh each `playerdata`'s `skills` list which represents all the skills that are available to each player party member. Check the skills data documentation for more details.

```cs
public static int[] PartyArray()
```
This is UNUSED. Returns an array of a single element being `playerdata[0]`.`trueid`.

```cs
public static void InventoryFromString(string inp, int itemtype, bool addd)
```
Replace `items[itemtype]` with the list of item ids obtained by splitting `inp` by `,`. If `addd` is true, the items will be added to the existing list instead.

NOTE: `inp` must be a a `,` separated list of valid item ids and `itemtype` must be 0, 1 or 2 or an exception will be thrown.

```cs
public static IEnumerator DelayedObj(float delay, string path, Vector3 position, string sound, float destroy)
```
After waiting for `delay` amount of seconds, load a GameObject using `path` ressources path placing it at `position`. If `destroy` is above 0.0, the GameObject will be destroyed in `destroy` amount of seconds.

```cs
public static bool HasFollower(AnimIDs followerid)
```
Returns true if any `tempfollowers` has an `originalid` of `followerid` - 1, false otherwise.

```cs
public static bool ForceMoving(EntityControl[] e)
```
Returns true if any entity in `e` is in a `forcemove`, false otherwise.

```cs
public static void PlayTransition(int id, int data, float speed, Color color)
```
Stops `transition` if it was ongoing and set it to a new Transition coroutine call with the following parameters:

- id: `id`
- data: `data`
- speed: `speed`
- color: `color`, but if each RGB component was higher than 0.95, the color value will be (0.95, 0.95, 0.95, `color`.a) which is almost pure white

Additionally, if `transitionobj` isn't empty and `id` is 0, 2, 4 or 5 TODO: what's the pattern ??? , all non null `transitionobj` elements gets destroyed before the Transition call.

```cs
public static void DestroyAllChildren(Transform parent)
```
Destroys all children GameObject of `parent`.

```cs
public static void DestroyText(Transform parent)
public static void DestroyText(Transform parent, bool destroy)
```
Calls DisableLetter on every child of `parent` with a tag of `Text` where DisableLetter has a destroy parameter of `destroy` if a value is sent (otherwise, the overload used is DisableLetter(Transform) which behaves as if true was sent).

```cs
private static void DisableLetter(Transform input)
private static void DisableLetter(Transform input, bool destroy)
```
Performs the following on `input`:

- Destroys all children GameObject with a ButtonSprite component
- Call DisableLetter on all children's TextMesh
- If `destroy` is true, `input` gets destroyed

The overload without a `destroy` parameter has its value default to true.

```cs
private static void DisableLetter(TextMesh input)
```
Either destroys `input` if it doesn't have a `Letter` tag (meaning it's not an active letter slot) or disable the letter slot if it does have a `Letter` tag such that it is considered free again. The disablement process goes as follow:

- Set `input`.text to empty string
- If `input` has a FontEffects, it is destroyed
- `input` gets childed to the MainManager instance
- `input`.tag is reset to `Untagged`

```cs
public static SpriteRenderer NewSolidColor(string name, Color color)
public static SpriteRenderer NewSolidColor(string name, Color color, float pixelsperunit, Vector3 position, Vector2 pivot)
```
Returns the SpriteRenderer of a newly created GameObject named `name` positioned at `position` where the SpriteRenderer was created by applying a 1x1 Texture2D on the entire sprite whose sole pixel color is `color`, the pivot of the sprite is `pivot` and the pixelsPerUnit of the sprite is `pixelsperunit`.

On the first overload, the `pixelsperunit` defaults to 100.0, the `position` defaults to Vector3.zero and the `pivot` defaults to (0.5, 0.5).

```cs
public static void FadeOut()
public static void FadeOut(float speed)
```
Calls PlayTransition(1, 0, `speed`, Color.black).

On the overload without a `speed` parameter, the value defaults to 0.1.

```cs
public static void FadeIn()
public static void FadeIn(float speed)
public static void FadeIn(float speed, Color color)
```
Calls PlayTransition(0, 0, `speed`, `color`).

On the overload without a `speed` parameter, the value defaults to 0.1.

On the overloads without a `color` parameter, the value defaults to Color.black.

```cs
public static void SetCameraInstant(Vector3 pos)
```
Calls SetCamera(`pos`, 1.0) which will instantly reposition the camera without doing it over a period of time.

```cs
private static IEnumerator Transition(int id, int data, float speed, Color color)
```
TODO: document the transition system separately since it gets complex

```cs
public static void ForceAnim(EntityControl entity)
```
Calls Play on the `anim` of `entity` with a clip that matches its current `animstate`.

NOTE: This only supports standard animations, extra animations and the `f` argument if the `height` is above 0.1.

```cs
public static void RefreshEntities()
public static void RefreshEntities(bool onlyplayer)
public static void RefreshEntities(bool forceanim, bool refreshmap)
public static void RefreshEntities(bool forceanim, bool refreshmap, bool onlyplayer)
```
TODO: document this separately since it's complex

```cs
public static void SetMonitor()
```
This is an empty method, it does nothing, but remains used by FlappyBee under normal gameplay.

```cs
public static void SetEntityLastPos(bool setit)
```
If `setit` is true, all EntityControl present in the scene will have their `lastpo` set to their position. If `setit` is false, all EntityControl present in the scene will have their position set to their `lastpos` and if their animid is -1 (None), their `rigid` gets their gravity disabled in kinematic mode.

```cs
public static float GetSqrDistance(Vector3 a, Vector3 b)
public static float GetSqrDistance(Vector3 a, Vector3 b, bool ignorey)
```
Returns (a - b).sqrMagnitude. If `ignoreY` is true, `a` and `b` will have their y components set to 0 before doing the substraction.

On the overload without a `ignoreY` parameter, it acts as if the value was false.

```cs
public static float GetDistance(Vector3 a, Vector3 b)
public static float GetDistance(Vector2 a, Vector2 b)
public static float GetDistance(Vector3 a, Vector3 b, bool ignoreY)
```
Returns (a - b).magnitude. If `ignoreY` is true, `a` and `b` will have their y components set to 0 before doing the substraction.

On the first overload, the method behaves as if `ignoreY` was false.

The second overload is UNUSED, but remains functional. It does the same operations as the first overload, but there's no z component.

```cs
public static float GetDistance(float a, float b)
```
Returns the absolute value of a - b.

```cs
public static float LoopValue(float value, float min, float max)
public static float LoopValue(float value, float min, float max, bool floor)
```
This is UNUSED. Returns a value derived from `value` by doing the following on it:

- Add `max` as long as `value` is lower than `min`
- Set value to %= `max` as long as `value` is higher than `max`
- If `floor` is true, `value` gets floored
- `value` is returned

On the overload without a `floor` parameter, the value defaults to false.

```cs
public static GameObject PlayParticle(string name, Vector3 position)
public static GameObject PlayParticle(string name, Vector3 position, float time)
public static GameObject PlayParticle(string name, string sound, Vector3 position)
public static GameObject PlayParticle(string name, string sound, Vector3 position, Vector3 angle, float alivetime)
public static GameObject PlayParticle(string name, string sound, Vector3 position, Vector3 angle, float alivetime, int rendersort)
```
Returns a newly created GameObject that is instantiated from the prefab at `Prefabs/Particles/X` where `X` is `name` at position `position` with angles of `angle` and whose Renderer has a material.renderQueue of `rendersort`. If `sound` isn't null, PlaySound(`sound`) is called before creating the GameObject. If `alivetime` is higher than 0.0, the GameObject will get destroyed in `alivetime` amount of seconds.

For the overloads without all parameters, their values defaults to the following:

- `sound`: null
- `angle`: (-90.0, 0.0, 0.0)
- `alivetime`: 5.0 (or the `time` value for the second overload)
- `rendersort`: 3000

```cs
public static Color RainbowColor(int variant)
public static Color RainbowColor(int variant, float colorspeed)
public static Color RainbowColor(int variant, float colorspeed, float min, float limit, float alpha)
```
TODO: complex sin math

```cs
public static float ColorMagnitude(Color color)
```
Returns (`color`.r, `color`.g, `color`.b).magnitude.

```cs
public static float Snap(in float value, in float step)
```
If `step` is 0.0, `value` is returned, otherwise, the return value is `value` / `step` + 0.5 floored * `step`. The intuitive way to think about this method is it returns the nearest number from `value` that is divisible by `step`.

```cs
public static Vector3 Snap(in Vector3 value, in Vector3 step)
```
Returns a vector where each component is Snap(`value`.x/y/z, `step`.x/y/z) with matching components for both vectors.

```cs
public static IEnumerator LevelUpMessage()
```
Applies any rank up bonuses if any exists and recalculate the party stats. Check the LevelUpMessage documentation to learn more.

```cs
public static void SetPlayers()
public static void SetPlayers(Vector3[] newentitypos)
```
A part of the map loading process. Recreate all `playerdata` elements. If `newentitypos` isn't null, the existing `playerdata` gets destroyed before the recreation and after, their position are set to matching `newentitypos` elements (indexed by `playerdata` index).

The parameterless overload acts as if `newentitypos` was null.

The recreation process is documented in the map loading documentation, but the method is also used in certain events that needs to recreate the party.

```cs
public static EntityControl[] GetPartyEntities()
public static EntityControl[] GetPartyEntities(bool idorder)
```
Returns an array of all `playerdata`'s `entity`. If `idorder` is false, the array is ordered by `playerdata` index. If `idorder` is true, the order is done by animid of `Bee`, `Beetle` and `Leif` (more precisely, it's GetEntity(-4), GetEntity(-5) and GetEntity(-6)).

```cs
private static IEnumerator LowerVolume(int musicid, float volume, float frametime, bool percent)
```
Changes `music[musicid]`.volume over `frametime` + 1.0 amount of frames via a lerp to `volume` (or the existing volume * `volume` if `percent` is true).

```cs
public static IEnumerator ChapterName(int chapterid)
```
Plays the animations and effects for a chapter start cutscene. TODO: document what value chapterid should have

```cs
public static float TieFramerate(float value)
```
Returns `value` * Time.smoothDeltaTime * 60.0. This is intended to be used to count game frames or to scale events happening over time. A `value` of 1.0 returns the actual smoothed out frametime while a `value` lower than 1.0 returns a fraction of it.

NOTE: The multiplication by 60.0 implies that a return value of 1.0 means that exactly 1/60th of a second passed since the last couple of frames on average. It strongly assumes that the target FPS is 60 so anything higher will result in the value being lower. The game was NOT designed to handle this and will result in a lot of events that happens over time being much slower than normal due to this strong FPS dependency.

```cs
public static IEnumerator ShakeObject(Transform obj, Vector3 shake, float frametime, bool returntostart)
```
For `frametime` + 1.0 amount of frames, `obj` position will be offset by RandomVector(`shake`) each frames. Once this is done, if `returntostart` is true, `obj` position is reset to its value at the start of this coroutine.

```cs
public static void LoadMap()
public static void LoadMap(int id)
public static void LoadMap(int id, bool recreateplayers)
```
Decomission and destroy the current `map` before loading a new one with a Maps id of `id`. If `recreateplayers` is true, all `playerdata` gets destroyed followed by setting `player` to null before this process.

This method is documented in detail in the map loading documentation.

```cs
public static void FixSamira()
```
If `samiramusics` isn't null or empty, this method will remove the following elements from `samiramusics` as these are intentionally excluded from the Samira purchase list:

- `Title`
- `Wind`
- `Water`
- `MachineHum`
- `Breathing`

```cs
public static Transform Create9Box(Vector3 position, Vector2 size, int type, int sortorder, Color color, bool grow)
```
Returns the transform of a newly created GameObject that is configured as a UI object using a 9 box configured sprites. More precisely, it's configured like the following:

- Name of `9Box`
- Childed to the `GUICamera`
- Layer 5 (`UI`)
- Local position of `position`
- No Angles
- If `grow` is false, the scale is Vector3.one. If it's true, it's Vector3.zero, but with a DialogueAnim component added which will bring the scale to Vector3.one over some frames
- A SpriteRenderer is added with the following properties:
    - sprite: Loaded from `Sprites/GUI/9Box/boxX` where `X` is `type`
    - color: `color`
    - drawMode: Tiled
    - tileMode: Adaptive

```cs
public static IEnumerator LateAngle(Transform obj, Vector3 angle, bool local, WaitForSeconds time)
```
Yield for `time` then change the angles of `obj` to `angle`. If `local` is true, the field of `obj` set is localEulerAngles instead of eulerAngles.

```cs
public static bool CheckAllBool(bool[] array, int[] values, bool state)
public static bool CheckAllBool(bool[] array, bool state)
```
Returns true if all `values` elements contains an `array` index whose matching value is `state`, false otherwise. All `values` elements must contain valid `array` indexes or an exception will likely be thrown.

For the overload without a `values` parameter, the method checks all of `array` elements to be `state` instead. It behaves the same then if `values` contained all the valid `array` indexes.

```cs
private static float GetLetterOffset(TextMesh letter, float size)
public static float GetLetterOffset(char letter, int fontid, float size)
```
Returns (`letter`.bounds.extents.x * 2.0 - 2.0) * `size`. If `letter` is null, 0.3 * `size` is returned instead.

This method is a part of SetText's single letter rendering used to calculate the position and scale of the backbox command when applicable.

```cs
public static float GetLetterOffset(char letter, int fontid, float size)
```
Returns the amount of space `letter` takes horizontally when rendered by SetText in normal letter rendering using a font with id `fontid` and a size of `size`. More precisely, this is calculated the following way:

- If `letter` is ` `, the result is 0.3 * `size`
- If `letter` is not contained in FontID(`fontid`), the result is 0.3 * `size`
- If nether of the above cases applies, the result is (the CharacterFont's advance * 0.75 - 2.0) * (`size` / the fontSize of the Font)

```cs
public static int FontID(int id)
```
Returns `id` mapped to a font id with the following logic:

- Any negative numbers maps to their absolute value + 1
- 2 (UNUSED) maps to 1 (`D3Streetism`)
- Any other numbers of 0 or above are unchanged

```cs
public static GameObject WaterSplash(Vector3 pos, Vector3 size)
```
Instantiates and returns a GameObject using the `Prefabs/Objects/WaterSplash` prefab positioned at `pos` with a scale of `size`. The GameObject will be destroyed in 0.75 seconds.

```cs
public static void SystemText(string text, Transform parent, Vector3 pos)
```
Calls SetText with the following parameters:

- text: `text`
- fonttype: 0
- linebreak: null
- dialogue: false
- tridimensional: false
- position: `pos`
- cameraoffset: Vector3.zero
- size: Vector3.one
- parent: `parent`
- caller: null

```cs
public static Vector3 VectorFromString(string[] inputs)
```
Returns a vector where the x component is `inputs[0]`, the y component is `inputs[1]` and the z component is `inputs[2]` after converting each strings to float.

```cs
public static IEnumerator LightingBolt(Vector3 a, Vector3 b, int segments, float variant, Color color, float frametime, float lineduration)
```
Instantiates a `Prefabs/Particles/LightingBolt` prefab initialy positioned at `a` with some configuration of its TrailRenderer:

- material.color: `color`
- colorGradient: A gradient that has 2 keyframes each with a color of `color`
- time: `lineduration`

The GameObject will progressively move towards `b` over the course of around `frametime` amount of frames in `segments` amount of movement segments (clamped from 2 so there's always at least 2 segments). These movement segments are evenly distributed in time and positioning over the course of moving from `a` to `b` via a Lerp. On any movement sections except the first and last one, there will be a random offset applied to the target of the movement determined by the return of RandomVector(`variant`). The intuitive way to think about this is that this will result in the trail looking jagged as a random offset gets applied to each target of each movement, but the starting position is always correct so it will result in a random zig zag pattern.

Once the whole movement is completed, the GameObject will be destroyed in 1.0 seconds and a frame is yielded.

```cs
private static bool ContainsLine(string command)
```
A part of OrganizeLines that determines if `command` is a SetText command whose name contains `line` and that its condition to cause a line break are satisfied. The only 2 line commands where it might not apply are the following:

- shopline: Doesn't apply if `player`.`npc[0]`.`interacttype` isn't Shop
- unpauseline: Doesn't apply if the game is in a `pausemenu`

The method doesn't support any other line commands which are expected to be hangled in OrganizeLines. Check the OrganizeLines documentation to learn more.

```cs
private static string ReplaceFunctions(string text)
```
A part of OrganizeLines that acts as the main commands processor of OrganiseLines for commands that supports this. Check the OrganizeLines documentation to learn more.

```cs
private static string OrganizeLines(string text, float maxoffset, float size, int fontid)
```
The SetText automatic line breaker system that will automatically insert line breaks when appropriate and process some specific commands on its own. Check the OrganizeLines documentation to learn more.

```cs
public static float GetDistance01(float start, float end, float multiplier)
```
This is UNUSED, but remains functional. Returns 1.0 - GetDistance(`start`, `end`) * `multiplier` clamped from 0.0 to 1.0.

```cs
public static float GetPercentage(float start, float end, float currentvalue)
```
Returns 1.0 - (`end` - `currentvalue`) / (`end` - `start` clamped from 0.001 to infiniy). The intuitive way to think about this is it returns the progression ratio of `currentvalue` assuming a starting point of `start` and a goal of `end`. It is assumed that `start` <= `currentvalue` <= `end`.

```cs
public static void ScreamWaves(Vector3 pos)
```
Starts a Waves coroutine with the following parameters:

- pos: `pos`
- ammount: 5
- frametime: 20.0
- delay: 0.25 seconds
- invert: false
- tridimentional: false
- color: null

```cs
public static IEnumerator Waves(Vector3 pos, int ammount, float frametime, WaitForSeconds delay, bool invert, bool tridimentional, Color? color)
```
Instantiates the `Prefabs/Objects/SphereGlowEffect` prefab `ammount` times all positioned at `pos` each with a Renderer's material.color of `color` if `color` isn't null with a wait of `delay` between each instantiation. The scale of each prefabs is done via a GradualScale with a frametime of `frametime` and a destroy of true so each instances gets destroyed after the scaling has completed. As for the initial and target scale:

- If `invert` is false, the initial scale is Vector3.zero and the target is (50.0, 50.0, 1.0), but the z component is 50.0 instead if `tridimentional` is true
- If `invert` is true, the initial scale and target are reversed from the above

```cs
public static IEnumerator GradualScale(Transform obj, Vector3 target, float frametime, bool destroy)
```
Changes `obj` scale towards `target` over `frametime` + 1.0 amount of frames. If `destroy` is true, `obj` gets destroyed once this is over.

```cs
public static IEnumerator SetText(string text, Vector3 position, Transform parent)
public static IEnumerator SetText(string text, Transform parent, NPCControl caller)
public static IEnumerator SetText(string text, bool dialogue, Vector3 position, Transform parent, NPCControl caller)
public static IEnumerator SetText(string text, int fonttype, float? linebreak, bool dialogue, bool tridimensional, Vector3 position, Vector3 cameraoffset, Vector2 size, Transform parent, NPCControl caller)
```
A combination between a text rendering system, a scripting system and a dialogue management system used by the game for various purposes from presenting dialogues to UI text rendering. Check the SetText documentation to learn more.

```cs
public static IEnumerator InnSleep(NPCControl caller, Vector3? position, bool changemusic, bool nofadeout)
public static IEnumerator InnSleep(NPCControl caller, Vector3? position, bool changemusic, bool nofadeout, Vector3? camn, Vector3? camp)
```
Performs the cutscene animations of the party sleeping and getting healed after. Here are what the parameters do:

- `caller`: If the value is not null, their `entity`.`talking` will be set to false before the animation and their `entity`.`emoticoncooldown` will be reset to 0.0 after the animation
- `position`: If the value is not null, the `player` position to set during the cutscene while the screen is all black
- `changemusic`: If true, a ChangeMusic call will happen at the end of the cutscene where the musicclip is `music[0].clip.name` or null if `music[0]`.clip is null
- `nofadeout`: If true, there will not be a fadeout transition at the end of the cutscene
- `camn`: If `position` and this value are not null, `map`.`camlimitneg` will be set to this value during the cutscene
- `camp`: If `position` and this value are not null, `map`.`camlimitpos` will be set to this value during the cutscene

For the second overload, `camn` and `camp` values defaults to null. All the overload does is start a coroutine of the other overload followed by a frame yield.

This coroutine is meant to be stored in `chaptername` as it is set to null at the end of the coroutine so this field can be used to track its progress. The cutscene involves these major steps:

- PlaySound(`Inn`, 5) called
- FadeMusic(0.1) called
- PlayTransition(4, 9999, 0.01, Color.black) called
- All frames will be yielded for up to 700.0 frames until the transition is over
- If `position` is not null, the `player` is repositioned and the camera limits adjusted if needed with a yield of 0.1 seconds and a TeleportFollowers(true) call
- Yield all frames for up to 60.0 frames as long as `sounds[5]` is still playing
- If `nofadeout` is false: PlayTransition(1, 0, 0.15, Color.clear). 0.5 seconds will be yielded after the call
- Heal called

```cs
public static bool FrameDifference(int frames)
```
Returns true if Time.frameCount % the game's framerate / 60.0 ceiled is 0. The `frames` parameter does nothing.

In practice, this will always return true with vsync set to OFF. If vsync is set to ON, the amount of time true is returned varies and it depends on the monitor's refresh rate:

- 60hz and lower: true is always returned
- Between 60hz and 120hz exclusive: true is returned every 2 frames, false otherwise
- Between 120hz and 180hz exclusive: true is returned every 3 frames, false otherwise
- etc... (keep adding 60hz to the bounds to decrease the frequency at which true is returned)

```cs
public static string GetDialogueText(int id)
```
Resolves a dialogue line id of `id`. Check the dialogue line id documentation to learn more.

```cs
public static SpriteRenderer NewSpriteObject(Vector3 position, Transform parent, Sprite sprite)
public static SpriteRenderer NewSpriteObject(string name, Vector3 position, Vector3 rotation, Transform parent, Sprite sprite, Material mat)
```
Returns the SpriteRenderer of a newly created GameObject with the following properties:

- name of `name`
- Childed to `parent`
- Layer 14 (`Sprite`)
- Local position of `position`
- Angles of `rotation`
- sprite of `sprite`
- material of `mat`

For the first overload, the `name` defaults to `tempsprite`, `rotation` defaults to Vector3.zero and `mat` defaults to the `spritemat`.

```cs
public static Sprite GetItemSprite(bool badge, int id)
```
Returns `itemsprites[0, id]` if `badge` is false or `itemsprites[1, id]` if `badge` is true.

```cs
public static int CrystalBerryAmmount()
```
Returns the amount of `crystalbflags` that are true.

```cs
public static IEnumerator FadeSound(AudioSource sound, float speed)
```
Decreases the volume of `sound` by TieFramerate(`speed`) every frame until it reaches 0.0 or `sound` doesn't play anymore. Stop is called on `sound` once this is over.

```cs
public static void TeleportFollowers()
public static void TeleportFollowers(bool all)
```
Does the following to bring all the followers close to the `player`:

- All `playerdata` except the first one has their `entity`.`rigid`.velocity reset to Vector3.zero and their `entity` poisition set to the `player` position + `globalcamdir`.forward.normalized * 0.025
- If `all` is true, all `map`.`tempfollowers` has their position set to the `player` position
- If `map`.`chompy` is present, DelayedPosition is called on them with a pos of the `player` position + `globalcamdir`.forward.normalized * 0.3

The parameterless overload has the `all` value default to false.

```cs
public static void TeleportFollowers(float distance, TPDir dir, Transform caller)
public static void TeleportFollowers(float distance, TPDir dir, Vector3 caller)
public static void TeleportFollowers(float distance, float offset, TPDir dir, Vector3 caller, bool alsoTPextras)
public static void TeleportFollowers(float distance, float offset, TPDir dir, Vector3 caller)
public static void TeleportFollowers(TPDir direction, Vector3 from)
```
TODO: This is complex, document and organise later

```cs
public static GameObject Create6Wheel(Sprite[] sprites, Vector3 spritescale)
public static GameObject Create6Wheel(Sprite[] sprites, Vector3 spritescale, bool gui)
public static GameObject Create6Wheel(Color[] colors, Sprite[] sprites, Vector3 spritescale, bool gui)
```
Returns a newly created GameObject named `Wheel` showing a 6 segments wheel each with a sprite rendered over a colored `guisprites[122]` (???). Here are what the parameters configures:

- `colors`: The material.color of each segment's `guisprites[122]` background sprite
- `sprites`: The foreground sprite rendered in each segment over their background sprite
- `spritescale`: The scale of of all foreground sprites
- `gui`: If this is true, all sprites renders on layer 5 (`UI`) and the GameObject gets childed to the `GUICamera` via SetParenting instead of being rooted to the scene

The first 2 overloads are UNUSED, but remains functional and will have the `colors` value default to 6 elements all being Color.white and the `gui` value default to false.

```cs
public static void SetParenting(Transform t, Transform parent)
```
Child `t` to `parent` and does the following on `t` after:

- Reset local position to Vector3.zero
- Reset scale to Vector3.one
- Reset angles to Vector3.zero

```cs
public static void Reset()
```
Reloads the main scene. This is equivalent to resetting the entire game which happens normally when exiting to the main menu.

```cs
private static void BreakLine(ref float x, ref float y, float max, Vector2 size)
```
If `x` is at least `max`, `x` is set to 0.0 and `y` is decreased by 0.7 * `size`.y. The method does nothing if `x` isn't at least `max`.

This is only used during SetText when processing an icon command if linebreak isn't null where `x` is currentoffset, `y` is currentline, `max` is linebreak and `size` is the SetText's size.

```cs
private static EntityControl GetLastFollowing()
```
Returns the last entity in the `player` follow chain or `player`.`entity` if the `player`.`entity` has no `followedby`. This will prioritise the last `playerdata`'s `entity`, but if that one has a `followedby`, the last `map`.`tempfollowers` is returned instead.

```cs
public static void StopEntitiesMove()
```
Calls StopForceMove on all EntityControl present in the scene.

```cs
public static EntityControl AddFollower(EntityControl caller, int id)
```
Adds a follower to the follow chain. Check the follower system documentation to learn more.

```cs
public static EntityControl GetExtraFollower(int id)
```
Returns the first follower entity in `map`.`tempfollowers` whose animid is `id` or null if no such follower exists.

```cs
public static IEnumerator LateParent(Transform a, Transform b, float time)
```
This is UNUSED, but remains functional. Waits `time` amount of seconds before childing `a` to `b`.

```cs
private static void ResetDiag()
private static void ResetDiag(bool soft)
```
Sets `tempdiag` to `|size,` + `fontdsize` + `|`. This is the starting value of `tempdiag` where no backtracking is in progress. If `soft` is false, `diagstring` is reset to a new empty list and `currentdialogue` is reset to 0.

The parameterless overload behaves as if `soft` was true.

```cs
public static void StopDig()
public static void StopDig(bool delayed)
```
Does the following on each `playerdata`'s `entity` which ends any digging animations that were ongoing:

- Call LockRigid(false) on the entity
- Reset `spin` to Vector3.zero
- Reset `startscale` to Vector3.one
- Reset the `sprite`'s scale to Vector3.one
- Reset `overrideanim` to false
- Enables the `ccol`

The overload with a `delayed` parameter is UNUSED, but remains functional and it will start a DigStop coroutine instead, but all it does is yield for 0.1 seconds before calling the other overload so it's effectively doing the same after waiting for 0.1 seconds.

```cs
private static IEnumerator DigStop()
```
Part of the UNUSED StopDig overload where it will yield for 0.1 seconds before calling the parameterless overload of StopDig.

```cs
public static int GetValueFromString(string input)
```
A SetText command utility used to obtain a value from a string `input` that can be formatted in multiple ways. Here are the format it supports (checked and done in order):

- `money`: Returns instance.`money`
- `vX` or `varX`: Returns the value of the flagvar slot `X`
- Anything else: Returns `input` converted to int

```cs
public static IEnumerator Shrink(Transform obj, float frametime, bool deleteonend)
```
Changes `obj`'s scale towards Vector3.zero over `frametime` + 1.0 amount of frames. If `deleteonend` is true, `obj` is destroyed once this is over.

```cs
public static void FaceTowardsY(Transform t, Vector3 pos)
```
Make `t` LookAt `pos` whitout changing the x and z angles.

```cs
public static void Insert(int value, ref int[] array)
```
Sets `array` to a new array with one more element at the end with a value of `value`.

```cs
private static int IntFromString(string input)
```
A SetText command utility used to obtain a value from a string `input` that can be formatted in multiple ways. Here are the format it supports (checked and done in order):

- `vX` or `varX`: Returns the value of the flagvar slot `X`
- Anything else: Returns `input` converted to int

NOTE: It's essentially the same as GetValueFromString, but without support for the `money` value.

```cs
public static void EndOfMessage()
```
A method that performs many checks and procedures to be done at the end of any SetText call in dialogue mode when not `inevent` and not in a `battle`. Check the SetText life cycle documentation to learn more.

```cs
public static bool AllActive(EntityControl[] e)
```
Returns true if all entities in `e` either doesn't have an `npcdata` or it has one with a `hit` of true, returns false otherwise.

```cs
public static void SetLetter(ref TextMesh letter, int fontid, string text, Transform parent, Vector3 pos, Color color, int sortorder, Vector3 size)
```
Configures the letter slot `letter` with the following:

- SetFont(`letter`, `fontid`) called
- Childed to `parent`
- Local position of `pos`
- text of `text`
- anchor of LowerLeft
- color and MeshRenderer's material.color of `color`
- angles of Vector3.zero
- scale of `size`
- MeshRenderer's sortingOrder of `sortorder`

```cs
public static Vector3 GetMinibubblePos(Transform target)
public static Vector3 GetMinibubblePos(Transform target, float y)
public static Vector3 GetMinibubblePos(float x, float y)
```
Returns a vector with the following components which allows to determine the local position a MiniBubble should use assuming it is childed to `GUICamera`:

- x: `x` * 33.0 clamped from -5.0 to 5.0
- y: `y`
- z: 10.0

The second overload has the `x` value be `MainCamera`.WorldToViewportPoint(`target`.position).x - 0.5.

The first overload is UNUSED, but remains functional and it does the same as the second overload, but with a `y` value of 0.85.

```cs
public static void ResetEntitySpeed()
```
Sets the `anim`.speed of all the following entities to 1.0:

- All `map`.`entities` that aren't null and have their `anim` not null
- All `playerdata`'s `entity`

```cs
public static void ResetEntitySpeed(bool all)
```
If `all` is false, calls the parameterless overload of this method.

If `all` is true, sets the `anim`.speed of all EntityControl present in the scene that satisfies all of the following conditions:

- Not `iskill`
- `originalid` that isn't negative (defined animid)
- `anim` not null

```cs
public static EntityControl FindEntity(int id)
```
Returns the first EntityControl present in the scene whose animid is `id` or null if no such entity exists.

```cs
public static void SortLights()
public static void SortLights(MeshRenderer[] r)
```
For every `r` element that isn't null and has a material.shader set to `fakelight`, their material.renderQueue is set to 2500 + `MainCamera`.WorldToViewportPoint(the `r` element).z * 100.0 (or * 10.0 if in a `battle`). If `r` is null, this process will be done on all MeshRenderer present on the scene that satisfies the conditions.

The parameterless overload has a `r` value of null.

Intuitively, this method allows to have all `fakelight` materials in `r` (or the whole scene) be visually sorted due to the renderQueue.

```cs
public static int MultipleKeys(bool hold)
```
TODO: Involves InputIO logic that's not documented yet

```cs
public static void RefreshHUDValues()
```
Does the following to update the displayed HUD values to their backing value:

- Set all `playerdata`'s `hpt` to their `hp`
- Set `tpt` to `tp`
- Set `moneyt` to `money`

```cs
public static void FixEntities()
```
Does the following on all EntityControl present on the scene except those whose `npcdata` is an NPC or Enemy with an `item`:

- Reset `overrideflip` to false
- Reset `overrideanim` to false
- Set `rigid` to have gravity without kinematic
- Set `onground` to true
- Reset `rigid`.velocity to Vector3.zero
- If `anim` isn't null, `anim`.`speed` is reset to 1.0

```cs
private static char GetKoreanChar(int[] ids)
```
Returns the Korean characters obtained by composing all three segments of its id that are contained in `ids`. Check the Korean letterprompt documentation to learn more.

```cs
private void UpdateKoreanPrompt(int type)
```
Updates the colors of the different sections of the Korean letterprompt when `type` is the current section needing to be selected. The colors are Color.red for selected, Color.black for not selected, but is selectable in the current section and (0.0, 0.0, 0.0, 0.5) for other sections than the current one. Check the Korean letterprompt documentation to learn more.

```cs
public static void ChangeLayer(Transform obj, int layer)
```
Changes the layer of all children of `obj` to `layer`.

```cs
private void ChangeLetterPrompt(int specific = -1)
```
TODO: recheck how letterprompt works to explain this one

```cs
private static void CreateLetterPrompt(int id, Color color)
```
Renders a letterprompt. Check the letterprompt documentation to learn more.

```cs
public static TextMesh GetEmptyLetter()
```
Finds and allocate if needed a free letter slot from `letterpool` and return it or return null if no letter slots is available. If the letter slot is null, it is allocated using NewLetter with its `letterpool` index ToString as the id before being returned. For any non null letter slots, only those with an text of empty string are considered free by this method.

```cs
public static bool AddExp(int ammount)
```
Increases `partyexp` by `ammount`. If this resulted in a rank up, true is returned, false otherwise. A rank up is detected if after the increase, `partyexp` becomes at least `neededexp`. When that happens, `partyexp` is decreased by `neededexp`.

Before returning, a PlaySound call happens to play the the `Exp2` sound with a volume of 1.0 and a pitch of 1.5 + a random number between -0.1 and 0.1. The `sounds` element to use is the first among `sounds[4]`, `sounds[5]`, `sounds[6]` and `sounds[7]` in that order that isn't playing (always falsback to `sounds[7]` even if it is playing).

```cs
public static void DeathSmoke(Vector3 pos)
public static void DeathSmoke(Vector3 pos, Vector3 size)
public static void DeathSmoke(Vector3 pos, Vector3 size, int render)
```
Does the following to emit some `deathpart`:

- Set `deathpart` scale to `size`
- Set `deathpart` position to `pos`
- Emit(20) called on `deathpart`
- Set `deathpart`'s Renderer's material.renderQueue to `render`

For the first 2 overloads, the `size` value defaults to Vector3.one and the `render` value defaults to 3000.

```cs
public static void HitPart(Vector3 pos)
```
Set `hitpart` position to `pos` and call Play on it to play its particles.

```cs
public static void PlayBleep(AudioClip bleep, float bleeppitch, float bleepvol, int i)
```
Play the `bleep` clip on `beeps` if i is an even number and `beeps` wasn't playing already. The pitch used will be `bleeppitch` + a random number between -0.05 and 0.05 and the volume will be `bleepvol` * `bleepvolume` (or `pausemenu`.`dvolume` if the game is in a `pausemenu` so it uses the working value in the settings).

The `i` parameter is meant to be a number that changes each calls to this. This is more suited for SetText where it sends the current character index being processed in the char loop so the playback happens around half the time a character is rendered.

```cs
public static bool ArrayIsEmpty(object[] input)
public static bool ArrayIsEmpty(object[] input, bool stringcheck)
```
Returns true if every elements in `input` is either null or its ToString returns `null`, false otherwise. If `stringcheck` is false, only the null value check is done, not the ToString.

The overload without the `stringcheck` parameter behaves as if its value was false.

```cs
public static void UpdateJounal()
public static void UpdateJounal(Library type, int variable)
```
If `type` is Bestiary and the game is `inbattle`, `librarystuff[1, variable]` is set to true which mark the bestiary entry as obtained.

Otherwise, the following happens to mark a `librarystuff` flag as obtained if applicable as well as revealing in the HUD the obtention of a flag:

- If `variable` isn't negative, `librarystuff[type, variable]` is set to true which marks the entry as obtained
- `discoveryhud` is set to 350.0 which reveals the HUD element
- The `Disc` animation clip plays on the child of `discoverymessage` or the `Arch` clip if `type` is Logbook
- The 2nd, 3rd and 4th child of `discoverymessage` have their enablement changed depending on `type`:
    - Logbook: Only the 4th child is enabled, 2nd and 3rd are disabled
    - Other `type` values: Only the 3rd child is enabled, 2nd and 4th are disabled

The parameterless overload has the `type` value default to Discovery and `variable` to -1. This has the effect of not doing any changes to `librarystuff`, but still reveal `discoveryhud`.

```cs
public static EntityControl GetEntity(int id)
public static EntityControl GetEntity(string id, EntityControl caller)
public static EntityControl GetEntity(string id)
private static EntityControl GetEntity(NPCControl caller, string args)
```
This returns the entity matching a certain id `id` with support for various format. The format of the id is documented in the entity id documentation. The `caller` parameter is returned if `id` is `caller` and the game isn't `inbattle`.

The overloads offer various levels of format support:

- First overload only supports numeric id format to address `map`.`tempfollowers`, `playerdata`, `map`.`entities` and `battle`.`extraentities`
- Second overload acts as the first after converting `id` to int when `inbattle`. Otherwise, it adds supports for an `id` of `this` and `caller` as well as supporting `defines` resolution where if resolved, the `id` is converted to int and used as the first overload. If no string `id` is provided and no `defines` exists for `id`, the overload acts as the first after converting `id` to int
- Third overload is the same as the second, but without support for `caller` (most likely, an exception will be thrown if `id` is `caller` in such case)
- Fourth overload acts as the first if `caller` is null. If it's not, it acts as the second overload

```cs
private static void SetTalk(bool dialogue, bool talkstate)
```
Changes the `tailtarget`'s EntityControl's `talking` to `talkstate`. If `talkstate` is true, their `backsprite` is also set to false.

This method does nothing if `dialogue` is false or if `tailtarget` is null.

This method is meant to be used in a SetText context where SetText can periodically control visually whether the `tailtarget` should be in their `t` animation or not.

```cs
public static float GetTextLenght(string text, int fontid)
```
Returns the sum of all GetLetterOffset values of each char in `text` using a fontid of `fontid` and a size of 1.0. Each ` ` in `text` is counted as 0.3 without calling GetLetterOffset.

```cs
public static void HealParticle(Transform parent, Vector3 size, Vector3 offset)
public static void HealParticle(Transform parent, Vector3 size, Vector3 offset, bool UI)
```
Instantiates `Prefabs/Particles/Heal` in a new GameObject with the following properties which will be destroyed in 3.0 seconds:

- Childed to `parent`
- Scale of `size`
- Local position of `offset`
- If `UI` is true, the layer is set to 5 (`UI`)

The overload without an `UI` parameter will have a default value of false.

```cs
public static void CreateCursor(Transform parent)
```
Set `cursor` to a newly created GameObject with the following properties:

- Name of `Cursor`
- Childed to `parent`
- Layer is 5 (`UI`)
- Scale of Vector3.one
- Local position of Vector3.zero
- Has a SpriteRenderer with a sprite of `cursorsprite[0]` and a sortingOrder of 1
- Has a SpriteBounce. MessageBounce is called on it after adding it

```cs
public static Vector3 RandomItemBounce(float range, float height)
```
Returns a vector where the y component is `height` and the x/z components comes the vector returned by ClampMagnitude(RandomVector(`range` / 2, `range` / 2), `range` / 2, `range` / 2). Intuitively, it's a vector with a random x/z direction (each random between -`range` / 2 and `range` / 2) whose magnitude is `range` / 2 and whose y component is `height`.

```cs
public static void DestroyTemp(GameObject obj, float time)
public static void DestroyTemp(ref GameObject obj, float time)
```
Set `obj` position to (0.0, -9999.0, 0.0) and destroy it in `time` amount of seconds. If `obj` is passed by ref, it is set to null after the Destroy call.

The method does nothing if `obj` is null.

```cs
private static int[] GetLibraryIDs(int index)
```
Get the array of effectively used `libraryorder[index]` (meaning the length is `librarylimit[index]`). Intuitively, it returns an array of all valid `librarystuff[index]` id so it acts as a Library enum value.

```cs
public static IEnumerator DelayedPosition(Transform obj, Vector3 pos, float delay, bool local)
```
Waits `delay` amount of seconds (or a frame if it's 0.0 or below) then set `obj` position to `pos` (or local position instead if `local` is true).

```cs
public static void DisableRender(Renderer render, float tolerance, Vector3 offset)
```
Changes `render`.shadowCastingMode depending on a CheckIfCamera test on `render` position using a tolerance of `tolerance` and an offset of `offset`. If the test returns true, the new shadowCastingMode is On, ShadowsOnly otherwise.

```cs
public static void DisableRender(GameObject render, float tolerance, Vector3 offset)
```
This is UNUSED, but remains functional. Set the enablement of `render` depending on the return of CheckIfCamera on `render` position using a tolerance of `tolerance` and an offset of `offset`.

```cs
public static void ShakeScreen()
public static void ShakeScreen(float time)
public static void ShakeScreen(float ammount, float time)
public static void ShakeScreen(float ammount, float time, bool dontreset)
public static void ShakeScreen(Vector3 ammount)
public static void ShakeScreen(Vector3 ammount, float time)
public static void ShakeScreen(Vector3 ammount, float time, bool dontreset)
```
Set `camposshake` to `camoffset` and `screenshake` to (`ammount`.x, `ammount`.y / 2.0, 0.0). If `time` is above 0.0, a StopScreenShakeReturn call is invoked in `time` amount of seconds (if `dontreset` is true, the coroutine is StopScreenShake instead). For more information on the screen shaking system, check the camera system documentation.

On all overload not taking a `dontreset` parameter, the value defaults to false.

On all overload not taking a `time` parameter, the value defaults to -1.0 (so no stop coroutine invoked in the future).

On the overloads not taking an `ammount` parameter, the value defaults to (0.1, 0.1, 0.1).

On the overloads taking an `ammount` parameter as float, the value defaults to (`ammount`, `ammount`, `ammount`).

```cs
public void StopScreenShakeReturn()
```
Part of ShakeScreen when dontreset is false. Reset `screenshake` to Vector3.zero and set the `MainCamera` local position to `camposshake`.

```cs
public void StopScreenShake()
```
Part of ShakeScreen when dontreset is true. Reset `screenshake` to Vector3.zero.

```cs
public static void FadeMusic(float fadespeed)
```
Calls ChangeMusic(null, `fadespeed`) to smoothly change the current music to silence.

```cs
private static bool CheckIfCamera(Vector3 campos, float tolerance, Vector3 offset)
```
Performs a test to see if `campos` + `offset` is a point that's rendered enough from `MainCamera`'s WorldToViewportPoint. More precisely, the method only returns true if the following conditions are satisfied from the return vector of `MainCamera`.WorldToViewportPoint(`campos` + `offset`):

- x component must be between -0.5 and 1.5 exclusive
- y component must be between -0.5 and 1.5 exclusive
- z component must be above `tolerance`

```cs
public static int GetTPCost(int player, int id)
public static int GetTPCost(int player, int id, bool matchid)
```
TODO: this should be documented in BattleControl

```cs
public static int HowManyItem(int type, int id)
```
Returns the number of occurences of the item whose id is `id` are in `items[type]`.

```cs
public static IEnumerator ArcMovement(GameObject obj, Vector3 startpos, Vector3 targetpos, Vector3 spin, float height, float frametime, bool destroyonend)
public static IEnumerator ArcMovement(GameObject obj, Vector3 targetpos, float height, float frametime)
```
TODO: complex math stuff

```cs
public static string GetBadgeName(int id)
```
Returns a formatted string using `menutext[268]` to indicate a medal's name whose id is `id` followed by the language specific of the "medal" term.

The way this works is `menutext[268]` only contains the letters `i` and `m`. The `m` part gets replaced by `menutext[159]` which is the language specific "medal" term. The `i` part gets replaced by `badgedata[id, 0]` which is the medal's name whose id is `id`. 

```cs
public static void RefreshBadgeOrder()
```
Reorder `badges` by creating a new list containing the same elements, but ordered in such a way that each medal id appears in the same order as `badgeorder` medal ids.

```cs
public static int[] OrganizeArrayInt(int[] inputarray, int[] order)
```
Returns a new array where every elements is `order` in the same order they appear, but every elements that doesn't also exists in `inputarray` are removed.

```cs
private static int[] GetBadgeIDs()
```
This is UNUSED, but remains functional. Returns an array containing the medal ids in `badges`. NOTE: If the array is empty, but the game isn't in a `pausemenu`, an array of element being -1 is returned instead of an empty array.

```cs
public static int[] GetEquippedBadgeIDs()
```
Returns an array containing the medal ids in `badges` that are equipped on any player party member or the party. NOTE: If the array is empty, but the game isn't in a `pausemenu`, an array of element being -1 is returned instead of an empty array.

```cs
private static int[] GetLibraryOrder(int id)
```
Returns all effectively used (meaning length is `librarylimit[id]`) entry ids in `libraryorder` where the first dimension index is `id` (so it acts as a Library enum value).

```cs
private static int[] GetOverralQuests()
```
Return an array cointaining non zero elements in `boardquests[1]` (taken quests) as they appear followed by all non zero elements in `boardquests[2]` (completed quests), but each elements is a negative number of the value.

```cs
public static Vector3[] GetPartyPos(bool inorder)
```
Return the `playerdata` positions in an array. If `inorder` is false, they are ordered by `playerdata` index. If `inorder` is true, they are in the following order: GetEntity(-4), GetEntity(-5), GetEntity(-6).

```cs
public static void DestroyPlayers(bool remake, bool inorder)
```
This is UNUSED, but remains functional. Destroys all `playerdata`'s `entity`.gameObject. If `remake` is true, SetPlayers(GetPartyPos(`inorder`)) is called after.

```cs
public static Vector3 MultiplyVector(Vector3 a, Vector3 b)
```
Returns a vector where each components is calculated by multiplying the matching components of `a` and `b`.

```cs
private static int[] GetQuestsBoard(int type)
```
TODO: document the questboard stuff separately as it gets complex

```cs
public static int[] GetBosses()
```
Returns the value `listvar` should have in the B.O.S.S battles list type. Check the B.O.S.S list type documentation to learn more.

```cs
public static void PlaySoundAt(string sound, float volume, Vector3 position)
```
Calls AudioSource.PlayClipAtPoint with the following parameters:

- clip: obtained by loading the `Audio/Sounds/X` AudioClip where `X` is `sound`
- position: `position`
- volume: GetSoundDistance(`position`) * `volume` * `soundvolume`

```cs
public static int[] GradualFill(int ammount)
public static int[] GradualFill(int startat, int ammount)
```
Returns an array with length `ammount` where the first element is `startat` and each subsequent elements increases by 1.

The overload without a `startat` parameter has its value default to 0.

```cs
public static int[] GetSettings()
```
Returns the value `listvar` should have in the settings list type. Check the settings list type documentation to learn more.

```cs
public static string[] Controllers()
```
Return Input.GetJoystickNames(), but each element that has a length of 1 or lower are removed.

```cs
public static int[] SamiraMissing()
```
Return a newly created array that contains all `samiramusics` elements that haven't been bought yet.

```cs
public static void ShowItemList(int type, Vector2 position, bool showdescription, bool sell)
```
The method used to present an ItemList. Check the ItemList documentation to learn more.

```cs
public static Vector3 ChildScale(Vector3 scale, Transform parent, bool swapZY)
```
Returns a vector where each components is calculated by dividing the matching components of `scale` over `parent`. If `swapZY` is true, only the x component is calculated with matching components (the y component is `scale`.y / `parent`.z and the z component is `scale`.z / `parent`.y).

```cs
public static Transform NewChild(string name, Transform parent)
```
Create and return the transform of a newly created GameObject childed to `parent` named `name` with angles and local position of Vector3.zero and a scale of Vector3.one.

```cs
public static bool HasQuest(int questid)
```
Returns true if `questid` appears in any `boardquests`.

```cs
public static int GetFreePlayerAmmount()
public static int GetFreePlayerAmmount(bool hponly)
```
Returns the amount of player party members that are considered free in battle. Check the GetFreePlayerAmmount documentation to learn more.

```cs
public static Vector3 CardinalSnap(Vector3 angle)
public static Vector3 CardinalSnap(Vector3 angle, bool cameratied)
```
Returns a normalized vector with a direction that is the closest of the 4 cardinal directions of `angle`.y. If `cameratied` is true, `globalcamdir`.forward is added to the result and then normalized. More precisely, here's the logic to determine the base direction of the vector based on `angle`.y:

- Between 45.0 and 135.0 exclusive: Vector3.left
- At least 135.0 and strictly lower than 225.0: Vector3.forward
- At least 225.0 and strictly lower than 315.0: Vector3.right
- At least 315.0 or strictly lower than 45.0: Vector3.back

For the overload without a `cameratied` parameter, the value defaults to false.

```cs
public static Vector3 CardinalSnap(Transform obj)
public static Vector3 CardinalSnap(Transform obj, int directions)
public static Vector3 CardinalSnap(Vector3 obj, int directions)
```
Return `obj` where the y component of the vector is Mathf.RoundToInt(`obj`.y / (360.0 / `directions`)) * (360.0 / `directions`). Intuitively, this y component will be the closest delimiter number among the ones that are obtained by splitting 360.0 in `directions` amount of equal length sections. It means it's meant to work with euler angles with an arbitrary amount of cardinal directions (a cardinal direction here is one evenly split segments of 360.0).

The first 2 overloads are UNUSED, but remains functional.

For the overloads taking `obj` as a transform instead of a vector, the value becomes `obj`.tansform.eulerAngles.

For the overload not taking a `directions` parameter, the value defaults to 4.

```cs
public static float CardinalSnap(float angle)
public static float CardinalSnap(float angle, int directions)
```
Return Mathf.RoundToInt(`angle` / (360 / `directions`)) * (360 / `directions`). Intuitively, this returns the closest delimiter number among the ones that are obtained by splitting 360 in `directions` amount of equal length sections. It means it's meant to work with euler angles with an arbitrary amount of cardinal directions (a cardinal direction here is one evenly split segments of 360). NOTE: The 360 / `direction` division is incorrect because it's an integer division, NOT a float one so there can be loss of precisions if `directions` isn't a whole divisor of 360 resulting in uneven segments splits.

For the overload without a `directions` parameter, the value defaults to 4.

```cs
public static Vector3 CardinalSnap8(Vector3 angle, bool cameratied)
```
Returns a normalized vector with a direction that is the closest of the 8 cardinal directions of `angle`.y. If `cameratied` is true, `globalcamdir`.forward is added to the result and then normalized. More precisely, here's the logic to determine the base direction of the vector based on `angle`.y (this is the value before the vector gets normalized):

- Between 22.5 and 67.5 exclusive: (-1.0, 0.0, -1.0)
- At least 67.5 and strictly lower than 112.5: (-1.0, 0.0, 0.0)
- At least 112.5 and strictly lower than 157.5: (-1.0, 0.0, 1.0)
- At least 157.5 and strictly lower than 202.5: (0.0, 0.0, 1.0)
- At least 202.5 and strictly lower than 247.5: (1.0, 0.0, 1.0)
- At least 247.5 and strictly lower than 292.5: (1.0, 0.0, 0.0)
- At least 292.5 and strictly lower than 337.5: (1.0, 0.0, -1.0)
- At least 337.5 or strictly lower than 22.5: (0.0, 0.0, -1.0)

```cs
public static float ClampToMinMax(float v, float min, float max)
public static float ClampToMinMax(float v, float min, float max, bool lowest)
```
Returns `min` or `max` depending on `v` and Lerp(`min`, `max`, 0.5) which is the midpoint of the range. If `v` is lower than the midpoint, `max` is returned, `min` otherwise. If `lowest` is true, this logic is inverted so `min` is returned if `v` is lower than the midpoint, `max` otherwise.

On the overload without a `lowest` parameter, the value defaults to false.

```cs
public static bool InBetween(float v, float a, float b)
```
Return true if `v` is higher or equal than `a` and strictly lower than `b`, false otherwise.

```cs
public static void LookAt(Transform obj, Vector3 targetp)
public static void LookAt(Transform obj, Vector3 targetp, bool keepangle)
```
Have `obj` LookAt `targetp` and set `obj` x and z angles to 0.0 after. This effectively changes `obj` y angles to the one aligned with `targetp`. If `keepangle` is true, the x and z angles of `obj` won't change instead of being set to 0.0.

On the overload without a `keepangle` parameter, it behaves as if the value was false.

```cs
public static int GetAlivePlayerAmmount()
```
Returns the amount of `playerdata` whose `hp` are above 0 and aren't `eatenby`.

```cs
public static void ApplySettings()
```
TODO: add the config.dat documentation that heavily involves this

```cs
public static bool AllPartyFree()
```
Returns true if no `playerdata` has a `cantmove` of 1 or higher (no actor turns available on the current main turn), false otherwise.

```cs
public static int BadgeHowManyEquipped(int id)
public static int BadgeHowManyEquipped(int id, int playerid)
```
Returns the amount of medals in `badges` whose medal id is `id` and is equipped to the party or to the player party member with a `trueid` of `playerid`.

```cs
public static bool BadgeIsEquipped(int id)
public static bool BadgeIsEquipped(int id, int playerid)
```
Returns true if there's at least 1 medal in `badges` whose medal id is `id` and is equipped to the party or to the player party member with a `trueid` of `playerid`. If `playerid` is -1, it includes every equipped medals in the search.

```cs
public static bool CheckActiveEntities(int[] ids)
```
Returns true if all `map`.`entities` whose index is in `ids` have an `npcdata`.`hit` of true, false otherwise. It is possible for an `ids` element to have this check inverted where an `npcdata`.`hit` of false is considered active by specifying the negative version of the index (this implies this can't be done with id 0).

```cs
public static void ReloadSave()
```
This method is a helper to setup an event 22 start (loading a save file). Here are the steps performed:

- StopSound(`CrowdChatter`) called
- FadeMusic(0.1) called
- Stop all coroutines of `battle` and destroy it if it exists
- Stop all coroutines of `events`
- Destroy all `playerdata` entities's GameObject
- Set RenderSettings.skybox to null
- Destroy the `map`
- `events`.StartEvent called to start event 22 without a caller

```cs
public static void ApplyBadges()
```
Recalculate the stats effects of all medals to the party. Check the ApplyBadges documentation to learn more.

```cs
public static GameObject CreateRock(Vector3 pos, Vector3 size, Vector3 rotation)
```
Returns a newly instantiated GameObject using the `Prefabs/Objects/CrackedRock` prefab with the following properties:

- Position: `pos`
- Angles: `rotation`
- Scale: `size`
- The MeshCollider is destroyed
- The Fader is destroyed

```cs
public static void CrackRock(Transform parent, bool destroyparent)
```
Instantiates `Prefabs/Objects/CrackRockBreak` with the same position, angles and scale of `parent`. If `destroyparent` is true, `parent` gets destroyed after the instantiation.

```cs
public static void AddBadge(int id)
```
Add a `badges` element where the medal id is `id` and the equip state is -2 (unequipped).

```cs
public static void SetUpList(int type, bool showdescription, bool sell)
```
This is a helper to preconfigure an ItemList in a standard way before calling ShowItemList. It shouldn't be called during an ItemList processing, only before starting one. It sets some ItemList related fields:

- `storeid`: 0
- `listtype`: `type`
- `listammount`: 6
- `listdesc`: `showdescription`
- `listsell`: `sell`

The method also sets `inputcooldown` to 5.0 so most inputs processing are disabled for 5.0 frames.

```cs
private static void ResetPlayerStat(int id)
```
A part of ApplyBadges where `playerdata[id]` has several fields reset to their base value. Check the ApplyBadges documentation to learn more.

```cs
public static void ResetList()
```
Sets some ItemList related fields to their starting values. This is meant to be a helper to ShowItemList where calling this can help prepare the setup of the ItemList before calling ShowItemList. It shouldn't be called during an ItemList processing, only before starting one. Here are the fields and their values:

- `listcursor`: 0
- instance.`option`: 0
- `listlow`: 0
- `listmax`: `listammount`
- `listY`: -1

```cs
public static int[] SaveList()
```
Returns the array value meant to be set to `overridedlist` which is meant to save some of the state of the ItemList. The array returned is 5 elements with the following fields values in order:

- instance.`option`
- `listcursor`
- `listlow`
- `listmax`
- `listoption`

The values can be restored by passing the returned array to LoadList.

```cs
public static void LoadList(int[] v)
```
Restore some ItemList fields using the values in `v`. The format has to match the one specified in SaveList where `v` is 5 elements containing these values in order:

- instance.`option`
- `listcursor`
- `listlow`
- `listmax`
- `listoption`

```cs
public static LoadData? Load(int file, bool lite)
```
Completely load the save file with filename `saveX.dat` where `X` is `file` from the game's root directory with various error handling. In practice, only the values 0, 1 and 2 are valid `file` values. For information on the file format itself, check the save file format documentation. The return value is a struct that contains some metadata about the save used to present them in the StartMenu or null if the loading failed. If `lite` is true, the save will be loaded to gather these metadata, but it won't result in any changes to any MainManager fields.

This method handles a lot of potential errors, here is a more detailed list of their logic:

- If the save file doesn't exist, null is returned
- Reading the save file is enclosed in a try catch block that will return null if the reading of the file failed
- The entire method is enclosed in a try catch block that will handle any exceptions by a Debug.Log call and returning null
- In `lite` loading, the reading of the save file's filename is enclosed in a try catch block that will handle exceptions by setting the LoadData's filename to `|color,1|SLOT X - NO FILE NAME` where `X` is `file` + 1

Additionally, in non `lite` loading, the following additional steps are done in this loading process:

- ChangeParty without fromscratch is called with the party composition in the save file once their information have been read sucessfully
- ApplyBadges is called right before returning on a sucessful load.

```cs
public static float[] Divisions(int divisions)
```
Return an array of length `divisions` where the first element is 1.0 / (`divisions` + 1) and each subsequent elements increases by 1.0 / (`divisions` + 1). Intuitively, it creates a sequence of increasing fractions where the numerator goes from 1.0 to `divisions` - 1 and the denominator is `divisions` + 1.

```cs
public static IEnumerator LatePos(Transform obj, Vector3 pos, float delay, bool keeptrying)
```
Wait `delay` amount of seconds before setting `obj` position to `pos` if `obj` isn't null by then. If `keeptrying` is true, it changes the behavior where the coroutine will set `obj` position to `pos` if it isn't null on each frames for the next `delay` amount of frames.

```cs
public static int SaveProgressIcons()
```
Returns the amount of progress icons that should be rendered on the StartMenu. Each count if a specific flag slot is true, here's the list:

- 41 (end of chapter 1)
- 88 (end of chapter 2)
- 299 (end of chapter 3)
- 345 (end of chapter 4)
- 347 (end of chapter 6)
- 346 (got the flame brooch)
- 555 (post game)

```cs
public static bool Save(Vector3? savepos)
```
Calls InputIO.Save(`savepos`) which will persist the game state to the current save file. Check the save file format documentation to learn more.

```cs
public static string PromptYesNo()
public static string PromptYesNo(int yes, int no)
```
Returns a SetText prompt command string to present a yes/no prompt. The string returned is the following parts concatenated together:

- `|prompt,map,0.5,2,`
- The value `yes`
- `,`
- The value `no`
- `,@`
- `menutext[5]` (yes text)
- `,@`
- `menutext[6]` (no text)
- `|`

The parameterless overload is UNUSED, but remains functional and has both `yes` and `no` default to -11 (it's expected to contain `|boxstyle,-1||end|`).

```cs
private static void GetRewardTokens(int score)
```
Changes the value of flagvar 1 (amount of game tokens) depending on `score`, `battleresult` (which indicate if the minigame was beaten) and flagvar 6 (indicates the minigame played, 0 is FlappyBee and 1 is MazeGame). The value is always clamped from 2 to 999999 at the end.

More specifically, this is the logic used to determine the value (before the clamping):

- If `battleresult` is false (the minigame was failed), the value is `score` / 200.0 floored
- Otherwise, if flagvar 6 is 0 (minigame was FlappyBee), the value is `score` / 95.0 floored
- Otherwise (minigame is assumed to be MazeGame), the value is `score` / 140.0 floored

```cs
public static bool AnyKeyButThis(int id, bool hold)
```
TODO: Involves InputIO logic that's not documented yet

```cs
public static IEnumerator TransferMap(int targetmap, Vector3 targetpos)
public static IEnumerator TransferMap(int targetmap, Vector3 moveto, Vector3 tppos, Vector3 othermovepos)
public static IEnumerator TransferMap(int targetmap, Vector3 moveto, Vector3 tppos, Vector3 othermovepos, NPCControl caller)
```
Changes the current map with a loading transition. Check the map loading documentation to learn more.

The first overload is UNUSED, but remains functional.

```cs
public void ForceLoadSprites()
```
Start an FLS coroutine.

```cs
private static IEnumerator FLS()
```
Forces a reference to be held for one frame of every `map`.`entities`'s `sprite`.`sprite` by creating new GameObject with SpriteRenderer childed to the `map` at the `player` position. Once all the GameObject are created, a frame is yielded. After, all the GameObjects that were created are immediately destroyed.

This method is meant to be a loading optimisation done during the map loading process.

```cs
private static string GetText(bool menu, int id)
```
Return GetDialogueText(`id`) if `menu` is false or `menutext[id]` if `menu` is true.

```cs
public static int GetEnemyPortrait(int id)
```
Obtains the `librarysprites` index of the portrait sprite associated with the enemy whose id is `id`. There's some special cases logic to this which is documented in the GetEnemyPortrait documentation.

```cs
public static Vector2 GetDirection(in Vector2 a, in Vector2 b)
public static Vector3 GetDirection(in Vector3 a, in Vector3 b)
public static Vector3 GetDirection(in Vector3 a, in Vector3 b, bool ignoreY)
```
Returns (`a` - `b`).normalized. If `ignoreY` is true, the vectors used to do the subtraction are `a` and `b` where their y component is 0.0.

For the overloads without the `ignoreY` parameter, the value defaults to false.

```cs
public static Vector3 GetDirection4(Vector3 a, Vector3 b, bool ignoreY)
```
Returns the same value than GetDirection(`a`, `b`, true). The `ignoreY` parameter doesn't semantically change anything.

```cs
public static IEnumerator MoveTowards(Transform obj, Vector3 target, float frametime, bool smooth, bool local)
```
Changes `obj` position (or local position if `local` is true) towards `target` via a Lerp (or SmoothLerp if `smooth` is true) over the course of `frametime` amount of frames.

```cs
public static IEnumerator MoveTowards(Transform obj, Vector3 start, Vector3 end, float frametime, bool smooth, Action<bool> caller)
public static IEnumerator MoveTowards(Transform obj, Vector3 start, Vector3 end, float frametime, float startdelay, float shrink, float destroytime, bool smooth, Action<bool> caller)
public static IEnumerator MoveTowards(Transform obj, Vector3 start, Vector3 end, float frametime, float startdelay, float shrink, float destroytime, bool smooth, Action<bool> caller, string soundatend)
```
The following happens to move `obj` from `start` to `end`:

- If `startdelay` is above 0.0, wait for `startdelay` amount of seconds
- Changes `obj` position from `start` to `end` via a Lerp (or SmoothLerp if `smooth` is true) over the course of `frametime` + 1.0 amount of frames
- `obj` position is set to `end`
- If `soundatend` isn't null, PlaySound(`soundatend`) is called
- A frame is yielded
- If `shrink` is above 0.0, a DialogueAnim is added to `obj` with a `shrink` of true and a `shrinkspeed` of `shrink`
- If `destroytime` isn't negative, `obj` is destroyed in `destroytime` amount of seconds (or immediately if the value is 0.0)
- `caller` is invoked with a parameter of false

The first 2 overloads will start a third overload coroutine and yield a frame.

For the overloads without a `soundatend` parameter, the value defaults to null.

The second overload is UNUSED, but remains functional.

For the first overload, here are the default values for the parameters not sent:

- `startdelay`: 0.0
- `shrink`: 0.0
- `destroytime`: -1.0

```cs
public static Vector3 GlobalScale(Vector3 desiredscale, Vector3 parentscale, bool flipZY)
```
This is UNUSED, but remails functional. Returns a vector where each components is calculated by dividing the matching components of `desiredscale` over `parentscale`. If `flipZY` is true, only the x component is calculated with matching components (the y component is `desiredscale`.z / `parentscale`.z and the z component is `desiredscale`.y / `parentscale`.y).

```cs
public static void DialogueText(string text, Transform tailtarget, NPCControl caller)
```
Calls SetText with the following parameters:

- text: `text`
- fonttype: 0
- linebreak: `messagebreak`
- dialogue: true
- tridimensional: false
- position: Vector3.zero
- cameraoffset: Vector3.zero
- size: Vector3.one
- parent: `tailtarget`
- caller: `caller`

```cs
public static IEnumerator OpenQuestBoard(EntityControl caretaker, NPCControl caller)
```
TODO: document quest board separately since this is complex

```cs
public static void ChangeBoardQuest(int id)
public static void ChangeBoardQuest(int id, int type)
```
If `boardquests[type]` doesn't contain the quest id `id`, the following happens:

- 0 (no quest) is removed from `boardquests[type]`
- `id` is added to `boardquests[type]`
- If `type` is 0 (open quests) and `id` is above 0 (not a no quest), flag slot 2 is set to false (indicates that new unchecked open quests are available)

For the overload without a `type` parameter, the value defaults to 0 (open quests).

```cs
private static string BoardString()
```
Returns a `,` seprated list of all `instance.boardquests[0]` quest ids in a string.

```cs
public static void CompleteQuest(int id)
```
TODO: document quest board separately since this is complex

```cs
public static ItemUse GetItemUse(int id)
public static ItemUse GetItemUse(int id, int itemtype)
```
Returns the ItemUse of an item with id `id`. Check the ItemUse documentation for more details. NOTE: `itemtype` must be 0 or an exception will be thrown.

The overload without an `itemtype` parameter has its value default to 0.

```cs
public static int? GetItemHeal(int id)
```
This is UNUSED. it involved the unused BattleControl.TryHealEnemyItem coroutine.

```cs
public static int DoItemEffect(ItemUsage type, int value, int? characterid)
```
Process an ItemUsage with value `value` on `playerdata[characterid]` if `characterid` isn't null. Check the DoItemEffect documentation to learn more.

```cs
public static void IgnoreRadius(Collider col, bool ignore)
```
This is UNUSED, but remains functional. If `ignore` is true, causes all NPCControl present in the scene to have their `scol` ignore `col`. If `ignore` is false, they are unignored instead.

```cs
private static Sprite GetButtonSpriteD(int id)
```
Returns null. The `id` value does nothing. TODO: huh???

```cs
public static void SwitchParty(bool battle)
```
TODO: this is sort of documented, but not organised right so this needs to be organised

```cs
public static int[] PrizeBadges(bool caravan)
```
TODO: document the prize medals stuff separately so it's clearer how this is organised

```cs
public static int GetEnemyPrizeID(int flagvalue)
```
TODO: document the prize medals stuff separately so it's clearer how this is organised

```cs
public static EntityControl[] GetEntities(int[] ids)
public static EntityControl[] GetEntities(int[] ids, EntityControl[] ads)
```
Return the array of entities where each element is a resolved entity id from `ids` using GetEntity(int). If `ads` isn't null, the `ads` entity are appended to the return array.

For the overload not taking a `ads` parameter, the value defaults to null.

```cs
public static Vector3 SmoothLerp(Vector3 a, Vector3 b, float t)
public static Vector3 SmoothLerp(Vector3 a, Vector3 b, float t, float onlythrough, float onlyafter)
```
Return a vector obtained by performing a Mathf.SmoothLerp on each matching components of `a` and `b` using `t` as the factor. If `onlythrough` is above 0.0 and `t` is less than it, a Vector3.Lerp(`a`, `b`, `t`) is returned instead. If `onlyafter` is above 0.0 and `t` is greater than it, a Vector3.Lerp(`a`, `b`, `t`) is returned instead. 

For the first overload, the `onlythrough` and `onlyafter` parameters defaults to 0.0.
