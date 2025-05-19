# HUD
The game can present UI elements showing some game informations in a HUD fashion. They are expressed in 5 `hud` elements with matching `hudfont` elements to present their text with a DynamicFont. The first 4 `hud` elements also have a backing `hudpsrites` that changes color depending on the new values on RefreshHUD. Here's an overview of what each of the `hud` elements shows:

- `hud[0]`: `playerdata[0]`'s HP / max HP
- `hud[1]`: `playerdata[1]`'s HP / max HP
- `hud[2]`: `playerdata[2]`'s HP / max HP
- `hud[3]`: The party's TP / max TP
- `hud[4]`: Berry count

There's also the `discoverymessage` HUD element which is handled completely separately. It represents a HUD notification that only pops temporarilly when `discoveryhud` cooldown hasn't expired yet. This typically happens when a new `librarystuff` entry was unlocked.

## RefreshHUD
RefreshHUD is the heart of the HUD system that manages all the `hud` elements and their related data. It's a method called on LateUpdate as long as `playerdata` isn't empty (a map isn't being loaded) and `basicload` is true (LoadEverything ran once). It has the following signature:

```cs
private static void RefreshHUD()
```
The method has 3 main logic (the first one that applies is done, they are mutually exclusive):

- (Re)create all `hud` elements if it's null or empty
- The first 4 `hud` elements updates and are shown if instance.`hudcooldown` hasn't expired yet by calling ShowHUD(true) which is detailed in a section below
- Hides the HUD if neither of the above applies by calling ShowHUD(false) which is detailed in a section below

No matter what however, the following tasks always happen at the end of the method:

- If the instance.`showmoney` cooldown hasn't expired yet (the `hud[4]` berry count HUD element should be shown) and the message lock is released, instance.`showmoney` decreased by `framestep`
- If the instance.`discoveryhud` cooldown hasn't expired yet (the instance.`discoverymessage` HUD element should be shown), it decreases by instance.`framestep`
- All of the `hud[4]` berry count HUD element updates happens as detailed below regardless even if the HUD was recreated and even if it's hidden

## HUD Creation and Recreation
The HUD is meant to be automatically recreated once it's destroyed. This is ensured by RefreshHUD.

### Creation
If the HUD isn't created yet or it was destroyed, RefreshHUD will do the following:

- Calls CreateHUD (detailed below)
- Reset `tphp` and `ptmhp` to new arrays with the same length as `playerdata` (these are last values observed by RefreshHUD for use in values updates, this is detailed in a section below)
- `hud[4]` berry count HUD has its local x position set to the `hud[3]` TP HUD's local x position so they are aligned horizontally with each other

CreateHUD is what does most of the HUD (re)creation. Here is the signature:

```cs
private static void CreateHUD()
```
More precisely, here's what this method does:

- Calls ApplyBadges
- Calls ApplyStatBonus
- Reset `hud`, `husprites` and `hudfont` to new arrays (length is 5 except for `hudsprites` which is 4)
- Creates a new GameObject named `HUD` childed to the `GUICamera` with no angles or local position
- `hudvalue` is reset to {6.5, 7.5, 8.5, 9.5, -6.5, -7.0}
- Each `hud` is created via a call to NewHUDElement where the parent is the `HUD` GameObject, the hudid is the matching `hud` index and the position depends on the specific `hud`:
    - 0: (-7.0, 8.5)
    - 1: (-3.5, 8.5)
    - 2: (0.0, 8.5)
    - 3: (7.0, 8.5)
    - 4: (`hud[3]` x position, 0.0 - `hud[3]` y position)

NewHUDElement is the method that creates each `hud`, here's the signature:

```cs
private static void NewHUDElement(int hudid, Vector2 position, Transform parent)
```
This is the method that ends up assigning `hud[hudid]` to a newly created GameObject with a local position of `position` (the z is 10.0) childed to `parent` (normally, always the `GUICamera`).

Here's the properties of the GameObject assigned:

- Name of `hudX` where `X` is `hudid`
- Childed to `parent`
- Local position of (`position`.x, `position`.y, 10.0)
- No angles
- One child being a new GameObject named `basespriteX` where `X` is `hudid` that has a SpriteRenderer. It has the following properties:
    - Layer 5 (`UI`)
    - sprite of `guisprites[4]` (the HUD background sprite)
    - sortingOrder of 10
    - Scale of (0.55, 0.65, 1.0)
    - No angles
    - A child being a new GameObject named `facesprite` that has the following properties:
        - Layer 5 (`UI`)
        - Local position of (-2.3, 0.0, 0.0)
        - Scale of 80%
        - sortingOrder of 12

From there, each `hudid` gets different properties. For any below 3 (any `playerdata` HP):

- `basesprite`'s color set to `charcolor[instance.playerdata[hudid].animid]`
- `facesprite`'s sprite set to `guisprites[instance.playerdata[hudid].animid + 5]` (player party member HUD icon)
- `hudsprites[hudid]` set to the `basesprite`'s SpriteRenderer
- NewUIObject called to create a new object named `hpicon` childed to the `facesprite` with local position (0.45, -0.6, 0.0) and a scale of 50% using `guisprites[24]` (HP icon) with a sortingOrder of 13
- If `hudid` refers to a `playerdata` that exists, `hudfont[hudid]` is SetUp with an inital text (see the table below for the formatting and values) with the following parameters:
    - Without centralize
    - With noKerning
    - Frequency of 2.0
    - Fonttype of 2 (the unused mapping that redirects to `D3Streetism`)
    - Sortorder of 11
    - Fontsize of 1.75x
    - Childed to the `basesprite`
    - Position of (-0.9, -0.6, 0.0)
    - Color of Color.white
    - Small of (3.0, 0.85)
    - The `dropshadow` is set to true

For a `hudid` of 3 (TP):

- `basesprite`'s color set to `menucolor[3]`
- `facesprite`'s sprite set to `guisprites[28]` (TP icon)
- `hudsprites[3]` set to the `basesprite`'s SpriteRenderer
- `hudfont[3]` is SetUp with an inital text (see the table below for the formatting and values) with the following parameters:
    - Without centralize
    - With noKerning
    - Frequency of 2.0
    - Fonttype of 2 (the unused mapping that redirects to `D3Streetism`)
    - Sortorder of 11
    - Fontsize of 1.75x
    - Childed to the `basesprite`
    - Position of (-0.9, -0.6, 0.0)
    - Color of Color.white
    - Small of (3.0, 0.85)
    - The `dropshadow` is set to true

For a `hudid` of 4 (Berry count):

- `basesprite`'s color set to `menucolor[0]`
- `facesprite`'s sprite set to `guisprites[29]` (Berry icon)
- `hudfont[3]` is SetUp with an inital text (see the table below for the formatting and values) with the following parameters:
    - Without centralize
    - With noKerning
    - Frequency of 2.0
    - Fonttype of 2 (the unused mapping that redirects to `D3Streetism`)
    - Sortorder of 11
    - Fontsize of 1.75x
    - Childed to the `basesprite`
    - Position of (-0.75, -0.6, 0.0)
    - Color of Color.white
    - The `dropshadow` is set to true

### Destruction
The HUD can be destroyed temporarilly with a call to RebuildHUD:

```cs
public static void RebuildHUD()
```
It does the following:

- Calls ApplyBadges
- Calls ApplyStatBonus
- Destroys `hud[0]`'s parent if `hud[0]` exists
- Reset `hud` to null

The effects are temporary because as soon as RefreshHUD gets called, it will recreate the HUD with the process mentioned above, but it has the effect of forcing a reconstruction and thus an update of the HUD.

This method only gets called on event 22 (loading a save file) and on a ChangeParty. It's meant to reflect drastic HUD changes only as if only values updates are needed, ForceHUD or RefreshHUDValues are better suited (they are detailed in a section below).

## HUD Updates
When the `hud` updates via RefreshHUD, a bunch of different tasks and values are involved. As a reminder, the first 4 `hud` elements are only updated if instance.`hudcooldown` hasn't expired yet while `hud[4]` (berry count) is always updated when RefreshHUD is called.

They are organised like the following (the first 3 `hud` elements will be refered to as `hud[0-2]` because they are all similar, but they refer to different associated values indexed by the same `hud` index):

|Transform and DynamicFont|Text format|Text values|Last values|Value updates|`hudsprites` updates|Sounds|
|---------|-----------|-----------|-----------|-------------|-------------------|------|
|`hud[0-2]`, `hudfont[0-2]`|`X/Y`|<ul><li>`X`: `playerdata[0-2]`.`hpt` padded left to 2 digits</li><li>`Y`: `playerdata[0-2]`.`maxhp` padded left to 2 digits</li></ul>|<ul><li>`X`: `tphp[0-2]`</li><li>`Y`: `ptmhp[0-2]`</li></ul>|<ul><li>`X`: `playerdata[0-2]`.`hpt` Incrementally updates towards `playerdata[0-2]`.`hp`, but the text only updates if last value is different</li><li>`Y`: Text is directly updated if last value is different</li></ul>|`hudsprites[0-2]`.color changes depending on `playerdata[0-2]`.`hp`:<ul><li>0: Color.gray</li><li>Higher than 0 and at most 4 while ColorDifference between existing color and instance.`charcolor[instance.playerdata[0-2].trueid]` is lower than 0.1: Color.red</li><li>None of the above applies: Lerp from existing color to instance.`charcolor[instance.playerdata[0-2].trueid]` with a factor of 0.05 * `framestep`</li></ul>|If `playerdata[0-2]`.`hpt` increases as a result of the incremental update while still not being equal to `playerdata[0-2]`.`hp` and `sounds[5]` isn't plauying anything, PlaySound is call to play `TP` on `sounds[5]` with 1.0 volume and a pitch of 1.15 + random number between -0.1 and 0.1|
|`hud[3]`, `hudfont[3]`|`X/Y`|<ul><li>`X`: instance.`tpt` padded left to 2 digits</li><li>`Y`: instance.`maxtp` padded left to 2 digits</li></ul>|<ul><li>`X`: `tmtp[0]`</li><li>`Y`: `tmtp[1]`</li></ul>|<ul><li>`X`: instance.`tpt` incrementally updates towards instance.`tp`, but the text only updates if last value is different</li><li>`Y`: Text is directly updated if last value is different</li></ul>|`hudsprites[3]`.color (if the sprite exists) is updated to a lerp from the existing color to instance.`menucolors[3]` with a factor of 0.05 * `framestep`|If instance.`tpt` increases as a result of the incremental update while still not being equal to instance.`tp` and `sounds[5]` isn't plauying anything, PlaySound is call to play `TP` on `sounds[5]` with 1.0 volume and a pitch of 1.0 + random number between -0.1 and 0.1|
|`hud[4]`, `hudfont[4]`|`X`|`X`: instance.`moneyt` padded left to 3 digits|`X`: `tempmoneh`|`X`: instance.`moneyt` incrementally updates towards instance.`money`, but the text only updates if last value is different|None (`hudsprites[4]` doesn't exist)|Before instance.`moneyt` updates, If it's not being equal to instance.`money` while instance.`showmoney` is above 0.0 and `sounds[4]` isn't plauying anything, PlaySound is call to play `Money` on `sounds[4]` with 1.0 volume and a pitch of 0.7 (1.0 instead if instance.`money` is higher than instance.`moneyt`) + random number between -0.1 and 0.1|

To note, an incremental value update means the backing HUD value only changes by 1 towards its target on each call while a direct update sets the value directly.

Additionally, a HUD update also causes the following to happen:

- If the game isn't `inbattle`, instance.`hudcooldown` decreases by `framestep` (so the HUD is never hidden during battle)
- ShowHUD(true) is called to reveal the hud (this is detailed in a section below)
- If instance.`hudcooldown` expired, it is set to -100.0

### Manually force direct updates
There's a way to force all HUD values to update immediately to their target values. This is done with the RefreshHUDValues method:

```cs
public static void RefreshHUDValues()
```
It the following to update the displayed HUD values to their backing value:

- Set all `playerdata`'s `hpt` to their `hp`
- Set `tpt` to `tp`
- Set `moneyt` to `money`

It's only called on event 22 (save loading), during StartBattle and on PauseMenu.PrepareExit, but it is meant to be callable externally to force the next RefreshHUD to have an updated HUD without any incremental value updates (everything will stabilise by the end of the next refresh).

## Reveal and hiding the HUD
RefreshHUD also controls if it should reveal or show the HUD depending on instance.`hudcooldown`. This is managed by the ShowHUD method:

```cs
public static void ShowHUD(bool show)
```
The HUD is shown if `show` is true, hidden if it's false.

This method will first determine if the HUD should be shown when it wasn't before. It happens if `show` is true while the `hudvisible` is false. If that happens, ForceHUD is called. `hudvisible` is updated to the `show` value either way. 

ForceHUD is a method with the following signature:

```cs
private static void ForceHUD()
```
It sets all `tphp`, `ptmhp`, `tmtp` values and `tempmoneh` to -1.

This essentially forces a call to RefreshHUD on the next LateUpdate because it will find that everything the HUD should display has changed.

After that, what happens in ShowHUD is each `hud` local y position gets lerped from the existing one to a number that depends on the `hud` index and its matching `hudvalue` with a factor of TieFramerate(0.2). The target of the lerp uses the following formula:

`A` * (`hudvalue` + (`hudvalue` - `X`) * (`hudvalue` - `Y`))

Here are the variables values:

- `A`: 1.0 except for `hud[4]` where it's -1.0
- `X`: 6.5 if `show` is false, 4.25 if it's true. If it's `hud[4]`, it's the other way around
- `Y`: 4.25 if `show` is false, 6.5 if it's true. If it's `hud[4]`, it's the other way around

Also, this formula is computed after lerping the `hudvalue` from the existing one to `X` + the `hud` index with a factor of TieFramerate(0.15).

## `discoverymessage`
There's a HUD notification object that is separared from all the rest of the `hud` system and that object is `discoverymessage`.

To show it, `discoveryhud` needs to be set to the amount of frames that it should be shown. This cooldown is updated by RefreshHUD where if it hasn't expired yet, it is decreased by `framestep`.

Setting the value will impact a method call in LateUpdate that happens right after RefreshHUD: RefreshDiscovery. Here is its signature:

```cs
private void RefreshDiscovery()
```
The method updates the local position of the `discoverymessage` UI element according to `discoveryhud`. If `discoveryhud` is higher than 0.0, the local position is lerped from the existing one to (-8.0, -3.0, 0.0) with a factor of TieFramerate(0.1) which will smoothly reveal `discoverymessage`. Otherwise, the same lerp happens, but towards (-8.0, -6.0, 0.0) instead which will hide `discoverymessage`.

On the first time this is called, `discoverymessage` will be null which will cause this method to create it from scratch (any subsequent calls reuses the same object). It is set to a NewUIObject with name `Discovery` childed to the `GUICamera` with a pos of (-8.0, -6.0, 0.0). It has one child GameObject that is instantiated from the `Prefabs/Objects/logbookicon` prefab with a local position of (0.0, -1.0, 10.0).

From there, there's 3 SetText calls all done with the following parameters:

- fonttype: 1 (`D3Streetism`)
- linebreak: null
- dialogue: false
- tridimensional: false
- position: (1.2, -1.0, 10.0)
- No camoffset
- size: Vector3.one
- parent: `discoverymessage`
- caller: null

This means that there's going to be 3 more children to `discoverymessage`, one for each of these SetText call for a total of 4 children. They corresponds to indicate what updated in the `librarystuff`. Here's what they say in order:

- `|single||color,4||dropshadow,1,-1|` followed by `menutext[102]` (Logbook updated)
- `|single||color,4||dropshadow,1,-1|` followed by `menutext[154]` (Quest complete)
- `|single||color,4||dropshadow,1,-1|` followed by `menutext[155]` (Achievement unlocked)

The last 2 of these are initially disabled upon creation.

The way this is supposed to work is UpdateJournal will change the enablement of these 3 children such that only the relevant one is enabled and the other 2 disabled.
