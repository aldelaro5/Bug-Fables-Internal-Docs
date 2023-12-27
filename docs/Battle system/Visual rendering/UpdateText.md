# UpdateText
This methods updates various textual information via [SetText](../../SetText/SetText.md) in [non dialogue mode](../../SetText/Dialogue%20mode.md#non-dialogue-mode).

All SetText calls in this method have the following parameters on top of being in non dialogue mode:

- [fonttype](../../SetText/Notable%20states.md#fonttype) of 0 (`BubblegumSans`)
- linebreak of 9999.0
- No tridimensional
- position of (0.0, -0.1, 0.0)
- No cameraoffset
- size of (0.85, 0.7)
- parent of `actiontext`
- No caller

## Chompy
This section applies only if the `currentaction` is `Chompy`:

- `actiontext` local position is set to (-5.0, 4.0, 10.0)
- All SetText objects under `actiontext` are destroyed via MainManager.DestroyText
- A `menutext` id is determined depending on `coption[option]` (the selected chompy menu option):
    - 0: 60 (`Attack`)
    - 1: 65 (`Do Nothing`)
    - 2: This depends on [flagvar](../../Flags%20arrays/flagvar.md) 56 (the [item](../../Enums%20and%20IDs/Items.md) id if the item chompy has):
        - `PoisonRibbon`: 234
        - `NumbRibbon`: 235
        - `SleepRibbon`: 236
    - 3: 273 (`Change Ribbons`)
- SetText is called in non dialogue mode using the `menutext` line selected earlier prepanded with `|`[center](../../SetText/Individual%20commands/Center.md)`|`

## During the [player phase](../Battle%20flow/Update.md#player-phase) of a [controlled flow](../Battle%20flow/Update.md#controlled-flow)
This section applies if the chompy one didn't and `enemy`, `action` and `inevent` are false while the [message](../../SetText/Notable%20states.md#message) lock is released:

### `cancelb`
If `cancelb` doesn't exist, it is created as a new GameObject named `cancelhelp` with a ButtonSprite component added to it and its SetUp called on it. That SetUp call receives:

- buttonid 5 (cancel)
- description of `menutext[43]` (`Cancel`) (prepanded with `|`[size](../../SetText/Individual%20commands/size.md#size)`,1,1.5|` if the [languageid](../../SetText/languageid.md#languageid) is `Japanese`)
- position of Vector3.zero
- iconsize of (0.5, 0.5, 0.5)
- sortingOrder of 1 
- parentobj `GUICamera`

After the creation, a DelayCancelBox coroutine is started which yields a frame followed by creating a new UI object named `back` childed to `cancelb` with a position of (2.25, 0.
0, 0.0), a size of (1.1, 2.75, 1.0) using `guisprites[0]` (a rectangle) and with a sortingOrder of -10. The SpriteRenderer color of this object is set to pure white with half opacity.

If the `currentaction` is `BaseAction` (the vine main action menu), `cancelb` position is set offscreen to (0.0, 999.0, 0.0). Otherwise, the local position is set to (2.1 or 2.5 if `longcancel` is true, 4.2, 10.0).

### `currentaction` logic
This section depends on the `currentaction`

#### `BaseAction`

- `actiontext` local position is set to (-5.0, -4.0, 10.0)
- All SetText objects under `actiontext` are destroyed via MainManager.DestroyText
- A string is built staring with `|`[center](../../SetText/Individual%20commands/Center.md)`|` followed by a `menutext` line that depends on `option`:
    - 0: 60 (`Attack`)
    - 1: 61 (`Skills`)
    - 2: 62 (`Items`)
    - 3: 63 (`Strategies`)
    - 4: 64 (`Turn Relay`)
- Further parts can be appended to the string if `currentturn` isn't negative (there is a player selected for an action):
    - `option` is 0 and the `AntlionJaws` [medal](../../Enums%20and%20IDs/Medal.md) on the `playerdata[currentturn].trueid`: ` |`[size](../../SetText/Individual%20commands/size.md)`,1,0.6||`[icon](../../SetText/Individual%20commands/Icon.md)`,183|` is appended
    - `option` is 1 and the `StatusBoost` [medal](../../Enums%20and%20IDs/Medal.md) on the `playerdata[currentturn].trueid`: ` |`[size](../../SetText/Individual%20commands/size.md)`,1,0.6||`[icon](../../SetText/Individual%20commands/Icon.md)`,220|` is appended
    - `option` is 2 and the `HealPlus` [medal](../../Enums%20and%20IDs/Medal.md) on the `playerdata[currentturn].trueid`: ` |`[size](../../SetText/Individual%20commands/size.md)`,1,0.6||`[icon](../../SetText/Individual%20commands/Icon.md)`,218|` is appended
    - `option` is 4 and the `RelayTransfer` [medal](../../Enums%20and%20IDs/Medal.md) on the `playerdata[currentturn].trueid`: ` |`[size](../../SetText/Individual%20commands/size.md)`,1,0.6||`[icon](../../SetText/Individual%20commands/Icon.md)`,216|` is appended
- A SetText call is done in non dialogue mode using the text obtained

#### `SelectEnemy` 

- `actiontext` local position is set to (-5.0, -2.5, 10.0)
- All SetText objects under `actiontext` are destroyed via MainManager.DestroyText
- A string is built depending on the `itemarea`, but it always starts with `|`[center](../../SetText/Individual%20commands/Center.md)`|`:
    - `SingleEnemy`:
        - If the [languageid](../../SetText/languageid.md#languageid) is `Japanese` or the `availabletargets[option].entityname` (the selected enemy's name) has a length of 5 or above, `|`[size](../../SetText/Individual%20commands/size.md)`,X,0.7|` is appended where `X` is 1.0 - 0.05 * `availabletargets[option].entityname.length` - 5 clamped from 0.5 to 0.8
        - `avaliabletargets[option].entityname` is appended
        - If `currentchoice` is `Strategy` while the [bestiary entry](../../Enums%20and%20IDs/librarystuff/Bestiary%20entry.md) of the `avaliabletargets[option].animid` [enemy](../../Enums%20and%20IDs/Enemies.md) id exists (the selected enemy id), ` |`[stars](../../SetText/Individual%20commands/Stars.md)`,1|` is appended
    - `All`: `menutext[77]` is appended
    - `AllEnemies`: `menutext[75]` is appended
    - `AllParty`: `menutext[76]` is appended
- A SetText call is done in non dialogue mode using the text obtained

#### `SelectPlayer`

- `actiontext` local position is set to (-5.0, -2.5, 10.0)
- All SetText objects under `actiontext` are destroyed via MainManager.DestroyText
- A string is built depending on the `itemarea`, but it always starts with `|`[center](../../SetText/Individual%20commands/Center.md)`|`:
    - `SingleAlly`:
        - `playerdata[option].entityname` is appended
        - If `currentchoice` is `Item` while the `WeakStomach` [medal](../../Enums%20and%20IDs/Medal.md) is equipped on `playerdata[option].trueid`, ` |`[size](../../SetText/Individual%20commands/size.md)`,1,0.6||`[icon](../../SetText/Individual%20commands/Icon.md)`,184|` is appended
    - `All`: `menutext[77]` is appended
    - `AllEnemies`: `menutext[75]` is appended
    - `AllParty`: `menutext[76]` is appended
- A SetText call is done in non dialogue mode using the text obtained

#### `SkillList` / `ItemList` / `StrategyList`
`actiontext` local position is set to offscreen at (0.0, 999.0. 0.0)

## Any other times
This section happens when neither of the other 2 did.

The only things that happens is the `actiontext` and `cancelb` local position are moved offscreen at (0.0, 999.0, 0.0).