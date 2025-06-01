# Special procedures
These methods performs special predefined procedures the game does.

```cs
public static void LoadMap()
public static void LoadMap(int id)
public static void LoadMap(int id, bool recreateplayers)
```
Decomission and destroy the current `map` before loading a new one with a Maps id of `id`. If `recreateplayers` is true, all `playerdata` gets destroyed followed by setting `player` to null before this process.

This method is documented in detail in the [map loading](../../MapControl/Map%20loading.md) documentation.

```cs
public static IEnumerator TransferMap(int targetmap, Vector3 targetpos)
public static IEnumerator TransferMap(int targetmap, Vector3 moveto, Vector3 tppos, Vector3 othermovepos)
public static IEnumerator TransferMap(int targetmap, Vector3 moveto, Vector3 tppos, Vector3 othermovepos, NPCControl caller)
```
Changes the current map with a loading transition. Check the [TransferMap](../../MapControl/Map%20loading.md#transfermap) documentation to learn more.

The first overload is UNUSED, but remains functional.

```cs
public static void UpdateArea(int newarea)
```
Performs the tasks needed to process an area change which happens if needed on a MapControl's [Start](../../MapControl/Init%20process.md#start). More precisely, the following is done:

- If flag 596 is true (talked to the Roach Elder in the postgame) and `areaid` is 15 (meaning this is changing the area from the Giant's Lair), flag 597 is set to true (the `elder` entity on GiantLairSaplingPlains has a `limit` on it so this will cause his disappearence from that map).
- All regional flags are reset
- `areaid` is updated to `newarea`
- The matching `librarystuff[4]` area flag of `newarea` is set to true which marks it as visible on the world map

```cs
public static void UpdateShops()
```
Set all `avaliablebadgepool` to each be a RandomSort version of their matching `badgeshops`. This happens on MapControl's Start, NPCControl's SetBadgePool(true) for a medals shop and during event 22 (loading a save file) when upgrading the save file to 1.2.0.

Additionally, for `avaliablebadgepool[0]` (Merab) specifically, if flag 587 is true (bought all Merab's medals), but `badgeshops[0]` isn't empty, flag 587 is reset to false. This corrects an inconsistency which can happen on a freshly upgraded 1.2.0 save file where the flag saying all medals were bought is true, but there's still some to be bought after the upgrade.

```cs
public static void EndMiniGame(bool antialias, int score)
```
Performs the necessary operations to end a Termacade game where the FXAA component's enablement is set to `antialias` where the game ended with a score of `score`. Among the tasks done by this method includes calling SetRenderTexture(0), calling GetRewardTokens(`score`).

The `antialias` parameter is expected to come from the component containing the Termacade game logic and it is expected to contain the initial enablement of the FXAA component when the game starts. This is used to restore the enablement to its original state. The `score` is used to determine how much game tokens should be awarded by GetRewardTokens.

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
public static void SetMonitor()
```
This is an empty method, it does nothing, but remains used by FlappyBee under normal gameplay.
