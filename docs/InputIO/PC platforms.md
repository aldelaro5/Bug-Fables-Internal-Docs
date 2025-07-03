# PC platforms
InputIO contains methods that are specific to each of the PC platforms. There are 3 available:

- Steam
- GOG
- Itch

For the sake of simplicity, only the Steam specific ones will be covered in this page which involves the Steamworks.NET library.

```cs
public static void StartUp()
```
This method is called early during the [boot process](../MainManager/Boot%20and%20reset%20process.md), but the logic within depends on the specific store platform the game uses.

On the Steam version, `steam` is set to the SteamManager component of a newly created GameObject named `Steam Manager` that is rooted to the scene. This will initialise Steamworks.NET to enable Steam's achievements functionality.

```cs
public static void Achivement(int id)
```
The logic of this Unity event depends on the specific store platform the game uses.

On the Steam version, it grants the `achivements[id]` (using SteamUserStats.SetAchievement) if it wasn't already (checked via SteamUserStats.GetAchievement) through Steamworks.NET. This is done only if SteamManager got initialised sucessfully, SteamUtils.IsOverlayEnabled returns true and SteamUserStats.RequestCurrentStats also returns true. If the achievement is granted, SteamUserStats.StoreStats is called.

This is an optional part of the [achievement system](../General%20systems/Achievements.md) so this method is only called when getting an in game achievement.

```cs
public void OnApplicationQuit()
```
The logic of this Unity event depends on the specific store platform the game uses.

On the Steam version, SteamAPI.Shutdown is called which decomissions Steamworks.NET.
