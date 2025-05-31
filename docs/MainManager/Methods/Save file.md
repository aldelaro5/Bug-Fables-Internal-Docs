# Save file
These methods perform operations on the [save file](../../External%20data%20format/Save%20File.md).

```cs
public static string SaveFile(Vector3? savepos)
```
Returns a save file compliant text from the current state of MainManager. Check the save file format documentation to learn more.

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
