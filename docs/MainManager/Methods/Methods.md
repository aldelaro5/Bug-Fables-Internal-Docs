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


```cs
public static void UpdateOutlines()
```


```cs
public static int MixIngredients(int item1, int item2)
```


```cs
public static bool HasRecipe(int id)
```


```cs
public static int GetRecipeID(int id)
```


```cs
public static void ResetCamera()
public static void ResetCamera(bool instant)
```


```cs
private void ResetCamSpeed()
```


```cs
public static bool PartyIsNotMoving()
```


```cs
public static bool GetKey(int id)
public static bool GetKey(int id, bool hold)
```


```cs
public static Vector2 GetPressedDirection(bool hold)
```


```cs
private void FixedUpdate()
```


```cs
public static float DimishingReturns(float input, float decay)
```


```cs
public static float[][] LoopPoint()
```


```cs
private void LoopMusic()
```


```cs
public static int GetRandomMedal()
public static int GetRandomMedal(bool dontremove, bool random)
```


```cs
public static GameObject NewUIObject(string objname, Transform parent, Vector3 pos)
public static GameObject NewUIObject(string objname, Transform parent, Vector3 pos, Vector3 size, Sprite sprite)
public static GameObject NewUIObject(string objname, Transform parent, Vector3 pos, Vector3 size, Sprite sprite, int sortorder)
```


```cs
public static bool CheckIfCanExist(int[] requires, int[] limit, int regionalflag)
```


```cs
private static Vector3 CamLimiter(Vector3 pos)
```


```cs
public static Vector3 LimitRadius(Vector3 pos, Vector3 origin, float radius)
public static Vector3 LimitRadius(Vector3 pos, Vector3 origin, float radius, bool ignoreY)
```


```cs
private void RefreshCamera()
```


```cs
public static void SaveCameraPosition()
public static void SaveCameraPosition(bool set)
```


```cs
public static void LoadCameraPosition()
```


```cs
public static Vector3 LerpVectorAngle(Vector3 input, Vector3 target, float ammount)
```


```cs
public static Vector3 LerpVectorAngleSmooth(Vector3 input, Vector3 target, Vector3 current, float ammount)
```


```cs
public static void RebuildHUD()
```


```cs
private static void CreateHUD()
```


```cs
private static void NewHUDElement(int hudid, Vector2 position, Transform parent)
```


```cs
public static float QuadraticY(float x1, float x2, float ymax, float currentx)
public static float QuadraticY(Vector2 x1, Vector2 x2, float ymax, float currentx)
```


```cs
public static string GetItemString(bool emptyinv)
```


```cs
public static Vector2 BeizierCurve(Vector2 start, Vector2 end, float ymax, float t)
```


```cs
public static bool EntitiesAreNotMoving(EntityControl[] entities)
```


```cs
public static string GetBleep(EntityControl entity)
public static string GetBleep(EntityControl entity, float volume)
```


```cs
public static Vector3 BeizierCurve3(Vector3 start, Vector3 end, float ymax, float t)
public static Vector3 BeizierCurve3(Vector3 start, Vector3 end, Vector3 mid, float t)
```


```cs
public static bool AsianLang()
```


```cs
public static float BeizierFloat(float height, float t)
```


```cs
public static float QuadraticYSemiClamped(float currentx, float startx, float endx, float modifier)
```


```cs
public static float QuadraticYUnclamped(float currentx, float startx, float endx, float modifier)
```


```cs
public static Vector3 RandomVector(Vector3 input)
public static Vector3 RandomVector(float randomx, float randomy, float randomz)
public static Vector3 RandomVector(float input)
public static Vector3 RandomVector(Vector2 input)
public static Vector3 RandomVector(float randomx, float randomy)
```


```cs
public static float ColorDifference(Color a, Color b)
```


```cs
public static void GlobalCommand(ref string text)
```


```cs
private static void ForceHUD()
```


```cs
private static void RefreshHUD()
```


```cs
public static void ShowHUD(bool show)
```


```cs
public static Vector3 MiddlePoint(Vector3[] inputs)
```


```cs
public static int CheckIfMore(int value, int[] values)
```


```cs
public static Vector3[] GetEntitiesPos(EntityControl[] e)
```


```cs
public static void RefreshSkills()
```


```cs
public static int[] PartyArray()
```


```cs
public static void InventoryFromString(string inp, int itemtype, bool addd)
```


```cs
public static IEnumerator DelayedObj(float delay, string path, Vector3 position, string sound, float destroy)
```


```cs
public static bool HasFollower(AnimIDs followerid)
```


```cs
public static bool ForceMoving(EntityControl[] e)
```


```cs
public static void PlayTransition(int id, int data, float speed, Color color)
```


```cs
public static void DestroyAllChildren(Transform parent)
```


```cs
public static void DestroyText(Transform parent)
public static void DestroyText(Transform parent, bool destroy)
```


```cs
private static void DisableLetter(Transform input)
private static void DisableLetter(Transform input, bool destroy)
private static void DisableLetter(TextMesh input)
```


```cs
public static SpriteRenderer NewSolidColor(string name, Color color)
public static SpriteRenderer NewSolidColor(string name, Color color, float pixelsperunit, Vector3 position, Vector2 pivot)
```


```cs
public static void FadeOut()
public static void FadeOut(float speed)
```


```cs
public static void FadeIn()
public static void FadeIn(float speed)
public static void FadeIn(float speed, Color color)
```


```cs
public static void SetCameraInstant(Vector3 pos)
```


```cs
private static IEnumerator Transition(int id, int data, float speed, Color color)
```


```cs
public static void ForceAnim(EntityControl entity)
```


```cs
public static void RefreshEntities(bool onlyplayer)
public static void RefreshEntities()
public static void RefreshEntities(bool forceanim, bool refreshmap)
public static void RefreshEntities(bool forceanim, bool refreshmap, bool onlyplayer)
```


```cs
public static void SetMonitor()
```


```cs
public static void SetEntityLastPos(bool setit)
```


```cs
public static float GetSqrDistance(Vector3 a, Vector3 b)
public static float GetSqrDistance(Vector3 a, Vector3 b, bool ignorey)
```


```cs
public static float GetDistance(Vector3 a, Vector3 b)
public static float GetDistance(Vector2 a, Vector2 b)
public static float GetDistance(Vector3 a, Vector3 b, bool ignoreY)
public static float GetDistance(float a, float b)
```


```cs
public static float LoopValue(float value, float min, float max)
public static float LoopValue(float value, float min, float max, bool floor)
```


```cs
public static GameObject PlayParticle(string name, string sound, Vector3 position)
public static GameObject PlayParticle(string name, Vector3 position, float time)
public static GameObject PlayParticle(string name, Vector3 position)
public static GameObject PlayParticle(string name, string sound, Vector3 position, Vector3 angle, float alivetime)
public static GameObject PlayParticle(string name, string sound, Vector3 position, Vector3 angle, float alivetime, int rendersort)
```


```cs
public static Color RainbowColor(int variant)
public static Color RainbowColor(int variant, float colorspeed)
public static Color RainbowColor(int variant, float colorspeed, float min, float limit, float alpha)
```


```cs
public static float ColorMagnitude(Color color)
```


```cs
public static float Snap(in float value, in float step)
```


```cs
public static Vector3 Snap(in Vector3 value, in Vector3 step)
```


```cs
public static IEnumerator LevelUpMessage()
```


```cs
public static void SetPlayers(Vector3[] newentitypos)
public static void SetPlayers()
```


```cs
public static EntityControl[] GetPartyEntities()
public static EntityControl[] GetPartyEntities(bool idorder)
```


```cs
private static IEnumerator LowerVolume(int musicid, float volume, float frametime, bool percent)
```


```cs
public static IEnumerator ChapterName(int chapterid)
```


```cs
public static float TieFramerate(float value)
```


```cs
public static IEnumerator ShakeObject(Transform obj, Vector3 shake, float frametime, bool returntostart)
```


```cs
public static void LoadMap(int id, bool recreateplayers)
public static void LoadMap()
public static void LoadMap(int id)
```


```cs
public static void FixSamira()
```


```cs
public static Transform Create9Box(Vector3 position, Vector2 size, int type, int sortorder, Color color, bool grow)
```


```cs
public static IEnumerator LateAngle(Transform obj, Vector3 angle, bool local, WaitForSeconds time)
```


```cs
public static bool CheckAllBool(bool[] array, int[] values, bool state)
public static bool CheckAllBool(bool[] array, bool state)
```


```cs
private static float GetLetterOffset(TextMesh letter, float size)
public static float GetLetterOffset(char letter, int fontid, float size)
```


```cs
public static int FontID(int id)
```


```cs
public static GameObject WaterSplash(Vector3 pos, Vector3 size)
```


```cs
public static void SystemText(string text, Transform parent, Vector3 pos)
```


```cs
public static Vector3 VectorFromString(string[] inputs)
```


```cs
public static IEnumerator LightingBolt(Vector3 a, Vector3 b, int segments, float variant, Color color, float frametime, float lineduration)
```


```cs
private static bool ContainsLine(string command)
```


```cs
private static string ReplaceFunctions(string text)
```


```cs
private static string OrganizeLines(string text, float maxoffset, float size, int fontid)
```


```cs
public static float GetDistance01(float start, float end, float multiplier)
```


```cs
public static float GetPercentage(float start, float end, float currentvalue)
```


```cs
public static void ScreamWaves(Vector3 pos)
```


```cs
public static IEnumerator Waves(Vector3 pos, int ammount, float frametime, WaitForSeconds delay, bool invert, bool tridimentional, Color? color)
```


```cs
public static IEnumerator GradualScale(Transform obj, Vector3 target, float frametime, bool destroy)
```


```cs
public static IEnumerator SetText(string text, Vector3 position, Transform parent)
public static IEnumerator SetText(string text, Transform parent, NPCControl caller)
public static IEnumerator SetText(string text, bool dialogue, Vector3 position, Transform parent, NPCControl caller)
public static IEnumerator SetText(string text, int fonttype, float? linebreak, bool dialogue, bool tridimensional, Vector3 position, Vector3 cameraoffset, Vector2 size, Transform parent, NPCControl caller)
```


```cs
public static IEnumerator InnSleep(NPCControl caller, Vector3? position, bool changemusic, bool nofadeout)
public static IEnumerator InnSleep(NPCControl caller, Vector3? position, bool changemusic, bool nofadeout, Vector3? camn, Vector3? camp)
```


```cs
public static bool FrameDifference(int frames)
```


```cs
public static string GetDialogueText(int id)
```


```cs
public static SpriteRenderer NewSpriteObject(Vector3 position, Transform parent, Sprite sprite)
public static SpriteRenderer NewSpriteObject(string name, Vector3 position, Vector3 rotation, Transform parent, Sprite sprite, Material mat)
```


```cs
public static Sprite GetItemSprite(bool badge, int id)
```


```cs
public static int CrystalBerryAmmount()
```


```cs
public static IEnumerator FadeSound(AudioSource sound, float speed)
```


```cs
public static void TeleportFollowers()
public static void TeleportFollowers(bool all)
public static void TeleportFollowers(float distance, TPDir dir, Transform caller)
public static void TeleportFollowers(float distance, TPDir dir, Vector3 caller)
public static void TeleportFollowers(float distance, float offset, TPDir dir, Vector3 caller, bool alsoTPextras)
public static void TeleportFollowers(float distance, float offset, TPDir dir, Vector3 caller)
public static void TeleportFollowers(TPDir direction, Vector3 from)
```


```cs
public static GameObject Create6Wheel(Sprite[] sprites, Vector3 spritescale, bool gui)
public static GameObject Create6Wheel(Sprite[] sprites, Vector3 spritescale)
public static GameObject Create6Wheel(Color[] colors, Sprite[] sprites, Vector3 spritescale, bool gui)
```


```cs
public static void SetParenting(Transform t, Transform parent)
```


```cs
public static void Reset()
```


```cs
private static void BreakLine(ref float x, ref float y, float max, Vector2 size)
```


```cs
private static EntityControl GetLastFollowing()
```


```cs
public static void StopEntitiesMove()
```


```cs
public static EntityControl AddFollower(EntityControl caller, int id)
```


```cs
public static EntityControl GetExtraFollower(int id)
```


```cs
public static IEnumerator LateParent(Transform a, Transform b, float time)
```


```cs
private static void ResetDiag()
private static void ResetDiag(bool soft)
```


```cs
public static void StopDig(bool delayed)
public static void StopDig()
```


```cs
private static IEnumerator DigStop()
```


```cs
public static int GetValueFromString(string input)
```


```cs
public static IEnumerator Shrink(Transform obj, float frametime, bool deleteonend)
```


```cs
public static void FaceTowardsY(Transform t, Vector3 pos)
```


```cs
public static void Insert(int value, ref int[] array)
```


```cs
private static int IntFromString(string input)
```


```cs
public static void EndOfMessage()
```


```cs
public static bool AllActive(EntityControl[] e)
```


```cs
public static void SetLetter(ref TextMesh letter, int fontid, string text, Transform parent, Vector3 pos, Color color, int sortorder, Vector3 size)
```


```cs
public static Vector3 GetMinibubblePos(Transform target, float y)
public static Vector3 GetMinibubblePos(Transform target)
public static Vector3 GetMinibubblePos(float x, float y)
```


```cs
public static void ResetEntitySpeed()
public static void ResetEntitySpeed(bool all)
```


```cs
public static EntityControl FindEntity(int id)
```


```cs
public static void SortLights()
public static void SortLights(MeshRenderer[] r)
```


```cs
public static int MultipleKeys(bool hold)
```


```cs
public static void RefreshHUDValues()
```


```cs
public static void FixEntities()
```


```cs
private static char GetKoreanChar(int[] ids)
```


```cs
private void UpdateKoreanPrompt(int type)
```


```cs
public static void ChangeLayer(Transform obj, int layer)
```


```cs
private void ChangeLetterPrompt(int specific = -1)
```


```cs
private static void CreateLetterPrompt(int id, Color color)
```


```cs
public static TextMesh GetEmptyLetter()
```


```cs
public static bool AddExp(int ammount)
```


```cs
public static void DeathSmoke(Vector3 pos, Vector3 size, int render)
public static void DeathSmoke(Vector3 pos, Vector3 size)
public static void DeathSmoke(Vector3 pos)
```


```cs
public static void HitPart(Vector3 pos)
```


```cs
public static void PlayBleep(AudioClip bleep, float bleeppitch, float bleepvol, int i)
```


```cs
public static bool ArrayIsEmpty(object[] input, bool stringcheck)
public static bool ArrayIsEmpty(object[] input)
```


```cs
public static void UpdateJounal()
public static void UpdateJounal(Library type, int variable)
```


```cs
private static EntityControl GetEntity(NPCControl caller, string args)
public static EntityControl GetEntity(string id)
public static EntityControl GetEntity(string id, EntityControl caller)
public static EntityControl GetEntity(int id)
```


```cs
private static void SetTalk(bool dialogue, bool talkstate)
```


```cs
public static float GetTextLenght(string text, int fontid)
```


```cs
public static void HealParticle(Transform parent, Vector3 size, Vector3 offset)
public static void HealParticle(Transform parent, Vector3 size, Vector3 offset, bool UI)
```


```cs
public static void CreateCursor(Transform parent)
```


```cs
public static Vector3 RandomItemBounce(float range, float height)
```


```cs
public static void DestroyTemp(ref GameObject obj, float time)
public static void DestroyTemp(GameObject obj, float time)
```


```cs
private static int[] GetLibraryIDs(int index)
```


```cs
public static IEnumerator DelayedPosition(Transform obj, Vector3 pos, float delay, bool local)
```


```cs
public static void DisableRender(Renderer render, float tolerance, Vector3 offset)
public static void DisableRender(GameObject render, float tolerance, Vector3 offset)
```


```cs
public static void ShakeScreen(Vector3 ammount, float time)
public static void ShakeScreen(float ammount, float time, bool dontreset)
public static void ShakeScreen(Vector3 ammount, float time, bool dontreset)
public static void ShakeScreen(float ammount, float time)
public static void ShakeScreen(float time)
public static void ShakeScreen()
public static void ShakeScreen(Vector3 ammount)
```


```cs
public void StopScreenShakeReturn()
```


```cs
public void StopScreenShake()
```


```cs
public static void FadeMusic(float fadespeed)
```


```cs
private static bool CheckIfCamera(Vector3 campos, float tolerance, Vector3 offset)
```


```cs
public static int GetTPCost(int player, int id)
public static int GetTPCost(int player, int id, bool matchid)
```


```cs
public static int HowManyItem(int type, int id)
```


```cs
public static IEnumerator ArcMovement(GameObject obj, Vector3 startpos, Vector3 targetpos, Vector3 spin, float height, float frametime, bool destroyonend)
public static IEnumerator ArcMovement(GameObject obj, Vector3 targetpos, float height, float frametime)
```

```cs
public static string GetBadgeName(int id)
```


```cs
public static void RefreshBadgeOrder()
```


```cs
public static int[] OrganizeArrayInt(int[] inputarray, int[] order)
```


```cs
private static int[] GetBadgeIDs()
```


```cs
public static int[] GetEquippedBadgeIDs()
```


```cs
private static int[] GetLibraryOrder(int id)
```


```cs
private static int[] GetOverralQuests()
```


```cs
public static Vector3[] GetPartyPos(bool inorder)
```


```cs
public static void DestroyPlayers(bool remake, bool inorder)
```


```cs
public static Vector3 MultiplyVector(Vector3 a, Vector3 b)
```


```cs
private static int[] GetQuestsBoard(int type)
```


```cs
public static int[] GetBosses()
```


```cs
public static void PlaySoundAt(string sound, float volume, Vector3 position)
```


```cs
public static int[] GradualFill(int startat, int ammount)
public static int[] GradualFill(int ammount)
```


```cs
public static int[] GetSettings()
```


```cs
public static string[] Controllers()
```


```cs
public static int[] SamiraMissing()
```


```cs
public static void ShowItemList(int type, Vector2 position, bool showdescription, bool sell)
```


```cs
public static Vector3 ChildScale(Vector3 scale, Transform parent, bool swapZY)
```


```cs
public static Transform NewChild(string name, Transform parent)
```


```cs
public static bool HasQuest(int questid)
```


```cs
public static int GetFreePlayerAmmount()
public static int GetFreePlayerAmmount(bool hponly)
```


```cs
public static Vector3 CardinalSnap(Vector3 angle, bool cameratied)
public static Vector3 CardinalSnap(Vector3 obj, int directions)
public static Vector3 CardinalSnap(Vector3 angle)
public static Vector3 CardinalSnap(Transform obj, int directions)
public static Vector3 CardinalSnap(Transform obj)
```


```cs
public static Vector3 CardinalSnap8(Vector3 angle, bool cameratied)
```


```cs
public static float ClampToMinMax(float v, float min, float max)
public static float ClampToMinMax(float v, float min, float max, bool lowest)
```


```cs
public static bool InBetween(float v, float a, float b)
```


```cs
public static void LookAt(Transform obj, Vector3 targetp)
public static void LookAt(Transform obj, Vector3 targetp, bool keepangle)
```


```cs
public static float CardinalSnap(float angle)
public static float CardinalSnap(float angle, int directions)
```


```cs
public static int GetAlivePlayerAmmount()
```


```cs
public static void ApplySettings()
```


```cs
public static bool AllPartyFree()
```


```cs
public static int BadgeHowManyEquipped(int id)
public static int BadgeHowManyEquipped(int id, int playerid)
```


```cs
public static bool BadgeIsEquipped(int id)
public static bool BadgeIsEquipped(int id, int playerid)
```


```cs
public static bool CheckActiveEntities(int[] ids)
```


```cs
public static void ReloadSave()
```


```cs
public static void ApplyBadges()
```


```cs
public static GameObject CreateRock(Vector3 pos, Vector3 size, Vector3 rotation)
```


```cs
public static void CrackRock(Transform parent, bool destroyparent)
```


```cs
public static void AddBadge(int id)
```


```cs
public static void SetUpList(int type, bool showdescription, bool sell)
```


```cs
private static void ResetPlayerStat(int id)
```


```cs
public static void ResetList()
```


```cs
public static int[] SaveList()
```


```cs
public static void LoadList(int[] v)
```


```cs
public static LoadData? Load(int file, bool lite)
```


```cs
public static float[] Divisions(int divisions)
```


```cs
public static IEnumerator LatePos(Transform obj, Vector3 pos, float delay, bool keeptrying)
```


```cs
public static int SaveProgressIcons()
```


```cs
public static bool Save(Vector3? savepos)
```


```cs
public static string PromptYesNo(int yes, int no)
public static string PromptYesNo()
```


```cs
private static void GetRewardTokens(int score)
```


```cs
public static bool AnyKeyButThis(int id, bool hold)
```


```cs
public static IEnumerator TransferMap(int targetmap, Vector3 targetpos)
public static IEnumerator TransferMap(int targetmap, Vector3 moveto, Vector3 tppos, Vector3 othermovepos)
public static IEnumerator TransferMap(int targetmap, Vector3 moveto, Vector3 tppos, Vector3 othermovepos, NPCControl caller)
```


```cs
public void ForceLoadSprites()
```


```cs
private static IEnumerator FLS()
```


```cs
private static string GetText(bool menu, int id)
```


```cs
public static int GetEnemyPortrait(int id)
```


```cs
public static Vector2 GetDirection(in Vector2 a, in Vector2 b)
```


```cs
public static Vector3 GetDirection(in Vector3 a, in Vector3 b)
public static Vector3 GetDirection(in Vector3 a, in Vector3 b, bool ignoreY)
```


```cs
public static Vector3 GetDirection4(Vector3 a, Vector3 b, bool ignoreY)
```


```cs
public static IEnumerator MoveTowards(Transform obj, Vector3 target, float frametime, bool smooth, bool local)
public static IEnumerator MoveTowards(Transform obj, Vector3 start, Vector3 end, float frametime, bool smooth, Action<bool> caller)
public static IEnumerator MoveTowards(Transform obj, Vector3 start, Vector3 end, float frametime, float startdelay, float shrink, float destroytime, bool smooth, Action<bool> caller)
public static IEnumerator MoveTowards(Transform obj, Vector3 start, Vector3 end, float frametime, float startdelay, float shrink, float destroytime, bool smooth, Action<bool> caller, string soundatend)
```


```cs
public static Vector3 GlobalScale(Vector3 desiredscale, Vector3 parentscale, bool flipZY)
```


```cs
public static void DialogueText(string text, Transform tailtarget, NPCControl caller)
```


```cs
public static IEnumerator OpenQuestBoard(EntityControl caretaker, NPCControl caller)
```


```cs
public static void ChangeBoardQuest(int id)
public static void ChangeBoardQuest(int id, int type)
```


```cs
private static string BoardString()
```


```cs
public static void CompleteQuest(int id)
```


```cs
public static ItemUse GetItemUse(int id)
public static ItemUse GetItemUse(int id, int itemtype)
```


```cs
public static int? GetItemHeal(int id)
```


```cs
public static int DoItemEffect(ItemUsage type, int value, int? characterid)
```


```cs
public static void IgnoreRadius(Collider col, bool ignore)
```


```cs
private static Sprite GetButtonSpriteD(int id)
```


```cs
public static void SwitchParty(bool battle)
```


```cs
public static int[] PrizeBadges(bool caravan)
```


```cs
public static int GetEnemyPrizeID(int flagvalue)
```


```cs
public static EntityControl[] GetEntities(int[] ids)
public static EntityControl[] GetEntities(int[] ids, EntityControl[] ads)
```


```cs
public static Vector3 SmoothLerp(Vector3 a, Vector3 b, float t)
public static Vector3 SmoothLerp(Vector3 a, Vector3 b, float t, float onlythrough, float onlyafter)
```

