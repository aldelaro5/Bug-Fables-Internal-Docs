# StartMenu
This component implements the title screen menu of the game. It is one of the last stage of the game's [boot process](../MainManager/Boot%20and%20reset%20process.md) and it mostly serves as a dispatcher to configure the game or loading / creating a save.

Most of the component contains verbose UI logic and the boot part has been detailed in the documentation linked above, but StartMenu also has 2 key part that is worth to document further here. These are the `Prefabs/Title/Title0` prefab instance and the menu naviguations.

## `Prefabs/Title/Title0` prefab
The background of the StartMenu features a scene that looks like a whole map, but it's prepared mostly for the StartMenu. It's normally just the prefab, but depending on the save that has the maximum `progression` value, different entities appear. For more information on this value, check the [save file](../MainManager/Methods/Save%20file.md) documentation.

These entities are created via a call to EntityControl.[CreateNewEntity](../Entities/EntityControl/EntityControl%20Creation.md#createnewentity) with name, [animid](../Enums%20and%20IDs/AnimIDs.md) and position. Everything created is childed to the prefab. They will periodically toggle their `talking` or `flip` state or change their [animstate](../Entities/EntityControl/Animations/animstate.md) depending on the entity, but this won't be detailed for the sake of brevity.

Here are the effects gained depending on this maximum (each are cumulative so any maximum implies the effects of everything mentioned above it):

- 0: Nothing, no entities are created
- 1: 3 entities:
    - `bee` with animid 0 (`Bee`) at (-5.6, 0.0, -4.27)
    - `beetle` with animid 1 (`Beetle`) at (-4.25, 0.0, -3.7)
    - `moth` with animid 2 (`Moth`) at (2.85, 0.0, -3.85)
- 2: `aria` with animid 78 (`MantisAcolyte`) at (8.94, 0.0, -1.7)
- 3: 2 entities and an object prefab:
    - `hb` with animid 159 (`DocHB`) at (-10.15, 0.0, -0.25)
    - `crow` with animid 160 (`HBAssistant`) at (-8.75, 0.0, -0.15)
    - An instance of `Prefabs/Objects/CrowPaper` at (-8.75, 0.015, -0.15)
- 4: 2 entities:
    - `bandit` with animid 138 (`Burglar`) at (-13.3, 6.9, 6.1)
    - `asto` with animid 195 (`Astotheles`) at (-10.45, 6.9, 5.9)
- 5: 2 entities:
    - `zasp` with animid 20 (`Zasp`) at (10.4, 0.0, 5.75)
    - `mothiva` with animid 10 (`Mothiva`) at (8.6, 0.0, 5.4)
- 6: `Jayde` with animid 249 (`Jayde`) at (-6.0, 0.0, 3.25)
- 7: `queen` with animid 96 (`AntQueen`) at (4.77, 0.0, -4.5)

## Menu naviguation
All of the menu naviguation logic happens through Update which mostly processes inputs, but also other tasks such as positioning the cursor correctly depending on the specific menu used. The system uses instance.`option` to tracke the selected options within a menu and instance.`maxoption` to denote the amount of options within a menu.

StartMenu is divided into multiple menus designed by an id that is tracked internally with a field called `menuid`. Here are the values mapping:

- 0: This is only used when StartMenu starts before getting to the main menu so it eventually goes to `menuid` 1
- 1: The main menu, offers 3 options:
    - Start Game which goes to `menuid` 2 upon confirmation
    - Settings which launches a [PauseMenu](../General%20systems/PauseMenu.md) on window 4 (Settings) with its `calledfrommain` set to true
    - Languages which sets the [languageid](../SetText/languageid.md) to -1 (unselected) upon confirmation and forces a settings save via InputIO.[LoadSettings](../InputIO/Disk%20files%20handling.md#config-file) with overwrite. This invalidates the language, it will also reload the main scene which will bring back the StartMenu at the language selection screen enforcing that a new language is selected since a language is required for booting
    - It's also possible from here to press the cancel [input](../InputIO/Inputs.md) to get to `menuid` 12
- 2: The save selection screen which has its own system of sub menus internally tracked with a field called `submenu`. Here are its values mapping:
    - 0: Selecting a save, a copy operation or a delete operation. Confirming a save here is explained in a section below. Pressing the cancel input here brings back `menuid` 1
    - 1: Copy operation which first requires a non empty save to be selected (this uses a special `copycursor` to differentiate it from the main cursor). Once this is done, a different save slot must be selected for the copy to be prepared which will go to `submenu` 3. Pressing the cancel input at any point brings back `submenu` 0
    - 2: Delete operations which requires a non empty saves to be selected. Doing so will go to `submenu` 4. Pressing the cancel input at any point brings back `submenu` 0
    - 3: Copying, this is where the actual copy from the first slot selected to the second happens by doing an InputIO.[CreateFile](../InputIO/Disk%20files%20handling.md#save-files) using the filename of the second selected file, but with the data being a InputIO.[ReadFile](../InputIO/Disk%20files%20handling.md#save-files) of the first selected file (so it's essentially overwritting the second file's data with the first or creating it if it didn't exist). Upon completion, this goes back to `submenu` 0 with the saves reloaded. This isn't cancellable. If an error occurs, this goes to `submenu` 5
    - 4: Deleting, this is where the actual delete happens with the selected file by doing a InputIO.[DeleteFile](../InputIO/Disk%20files%20handling.md#save-files) using the filename of the selected file. Upon completion, this goes back to `submenu` 0 with the saves reloaded. This isn't cancellable. If an error occurs, this goes to `submenu` 5
    - 5: Copy or delete error, this just goes back to `submenu` 0
- 3: A menu that can't be backed out of where a save is being created or loaded. When reaching it, the only way to go back to previous `menuid` is to reset the game by reloading the main scene
- 12: The exit prompt. Pressing the confirm inputs goes back to `menuid` 1 and pressing the cancel input calls Application.Quit

### Confirming a save
There are 3 possible ways to confirm a save. All of them leads to go to `menuid` 3 and all of them set MainManager.`saveslot` to the slot number which selects the save for the rest of the game. Here are the details:

- Pressing the confirm [input](../InputIO/Inputs.md) on an empty save. This will start [event](../Enums%20and%20IDs/Events.md) 8 which will go through the save creation process and eventually end with the game's intro cutscene (the event only ends when control is gained). It won't be possible to cancel this unless permitted by the event which is done through an option on the confirmation prompt that will reset the game
- Pressing the confirm [input](../InputIO/Inputs.md) on a non empty save. This will start [event](../Enums%20and%20IDs/Events.md) 22 which will load the save (or upgrade it from a previous version if needed) and the event ends only when control is gained back
- Pressing the help [input](../InputIO/Inputs.md) on a non empty save. This is the same as pressing the confirm input, but [flagvar](../Flags%20arrays/flagvar.md) 0 is set to 985 which informs event 8 (save creation) that it should offer the secret menu which allows to toggle various menu codes that modifies the save. This is only allowed if at least 2 codes are unlocked from the [config file](../External%20data%20format/Config%20File.md)

## Other Notes
Here are some other important details about StartMenu:

- If a language is selected and the game isn't `inevent`, GC.Collect is called every 600.0 frames which is presumably done due to the same reasoning than [DoClock](../MainManager/DoClock.md#resourcesunloadunusedassets-and-gccollect), but it notably doesn't call Resources.UnloadUnusedAssets (DoClock is not running during StartMenu)
- On Start, the RenderSettings.skybox is set to the `Materials/Skybox/Grass1` material with a `_Tint` color of Color.gray
- Once the game boots with valid [languageid](../SetText/languageid.md), the [music](../General%20systems/Music%20playback.md) changes to `Title`
- All text shown in the `menus` and `submenu` that aren't part of a save load or create [event](../Enums%20and%20IDs/Events.md) are done using [SetText](../SetText/SetText.md) in [non dialogue mode](../SetText/Dialogue%20mode.md#non-dialogue-mode), but every glyphs shown at the bottom in prompts are done by manually adding [ButtonSprites](ButtonSprite.md)
