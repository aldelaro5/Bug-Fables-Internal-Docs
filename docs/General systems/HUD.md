
## HUD

```cs
private void RefreshDiscovery()
```
Updates the local position of the `discoverymessage` UI element according to `discoveryhud`. If `discoveryhud` is higher than 0.0, the local position is lerped from the existing one to (-8.0, -3.0, 0.0) with a factor of TieFramerate(0.1) which will smoothly reveal `discoverymessage`. Otherwise, the same lerp happens, but towards (-8.0, -6.0, 0.0) instead which will hide `discoverymessage`.

On the first time this is called, `discoverymessage` will be null which will cause this method to create it from scratch (any subsequent calls reuses the same object). It involves the `Prefabs/Objects/logbookicon` prefab and 3 SetText calls in non dialogue mode. After the creation, `discoverymessage` will be the parent of the prefab.

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
public static void RefreshHUDValues()
```
Does the following to update the displayed HUD values to their backing value:

- Set all `playerdata`'s `hpt` to their `hp`
- Set `tpt` to `tp`
- Set `moneyt` to `money`
