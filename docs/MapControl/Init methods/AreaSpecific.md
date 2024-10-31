# AreaSpecific
This method mostly involves with logic that are specific to an [area](../../Enums%20and%20IDs/librarystuff/Areas.md) or a [map](../../Enums%20and%20IDs/Maps.md).

The method starts with this:

- Test is called which will purposely force a NullReferenceException to be thrown if `mapid` is the `TestRoom` and we aren't in the Unity Editor. The throw is done by casting null to GameObject and then attempting to set its transform position to the default of Vector3. Since the object is null, this will always throw
- If instance.`areaid` isn't `WaspKingdom` or `GiantLair`, [flag](../../Flags%20arrays/flags.md) 401 is set to false which turns off the stealth mode. NOTE: It doesn't exclude the `BanditHideOut` area which is incorrect, see the [CloseMove](../Follower%20system.md#closemove) documentation to learn more

## Area specific logic
Any areas not mentioned here has no special logic defined.

### `GiantLair`

- [flag](../../Flags%20arrays/flags.md) 401 (stealth mode) is set to the value of `closemove` (only the 2 maps in this area with DeadLanderOmega present has it set to true)
- If [discovery](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) 48 (Giant's Lair) isn't unlocked yet, it is unlocked via UpdateJournal

### `BugariaCity`

- If [flag](../../Flags%20arrays/flags.md) 67 is true (Chapter 2 started) and `canfollowID` contains 46 (`Maki`), it is removed from `canfollowID`

### `BanditHideout`

- `render[0]`(the `Base` GameObject's Renderer)'s material color is set to FBFF41 (mostly yellow) which gives a tint of most of the textures
- `nocolorchange` is set to true which prevents most of the [Fader logic](../Graphics%20configuration.md#fader-control) to happen
- At the end of this method, all GameObjects with a `CopyMainColor` tag has their Renderer's material color set to `render[0]`'s material color meaning they will also get the tint that `Base` got

### `ChomperCaves`

- `render[0]` (the `Base` GameObject's Renderer)'s material color is set to a lerp between red and yellow with a factor of 0.65 (FF9903, mostly orange) which gives a tint of most of the textures
- `nocolorchange` is set to true which prevents most of the [Fader logic](../Graphics%20configuration.md#fader-control) to happen
- At the end of this method, all GameObjects with a `CopyMainColor` tag has their Renderer's material color set to `render[0]`'s material color meaning they will also get the tint that `Base` got

### `SandCastle`

- `nocolorchange` is set to true which prevents most of the [Fader logic](../Graphics%20configuration.md#fader-control) to happen

### `WaspKingdom`

- 380 (`WaspTwinA`) is added to `canfollowID`
- `closemove` and `cantcompass` are set to true except for the following maps where they are set to false:
    - `WaspKingdomOutside`
    - `WaspKingdomJayde`
    - `WaspKingdomMainHall`
    - `WaspKingdomPrison`
    - `WaspKingdom5`
    - `WaspKingdomQueen`
    - `WaspKingdomThrone`
- If `skycolor`'s magnitude is higher than 0.9, it is set to pure gray
- If `closemove` was just set to true, flag 401 is set to true (stealth mode enabled)
- If [discovery](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) 38 (Wasp Kingdom) isn't unlocked yet, it is unlocked via UpdateJournal

### `DefiantRoot`

- If [discovery](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) 28 (Defiant Root) isn't unlocked yet, it is unlocked via UpdateJournal

### `BarrenLands`

- If [discovery](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) 39 (Forsaken Lands) isn't unlocked yet, it is unlocked via UpdateJournal

### `Desert`

- 406 (`Roy`) is added to `canfollowID`
- If [discovery](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) 30 (The Lost Sands) isn't unlocked yet, it is unlocked via UpdateJournal

### `StreamMountain`

- If [discovery](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) 33 (Stream Mountain) isn't unlocked yet, it is unlocked via UpdateJournal

### `FarGrasslands`

- 406 (`Roy`) and 380 (`WaspTwinA`) are added to `canfollowID` if `mapid` isn't among the following:
    - `WizardTowerAttic`
    - `WizardTowerBasement`
    - `WizardTowerStairs`
    - `FGCave`
    - `FarGrasslandsWizard`
    - `FGClearing`
- RenderSettings.fogColor is set to a lerp from white to green with a factor of 0.15 (D9FFD9, mostly light green)
- If [discovery](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) 34 (Far Grasslands) isn't unlocked yet and the map isn't `BroodmotherLair` or `FGCave`, it is unlocked via UpdateJournal

### `WildGrasslands`

- 380 (`WaspTwinA`) is added to `canfollowID`
- If [discovery](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) 37 (Far Swamplands) isn't unlocked yet, it is unlocked via UpdateJournal

### `RubberPrison`

- 406 (`Roy`) is added to `canfollowID`
- If [discovery](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) 47 (Rubber Prison) isn't unlocked yet, it is unlocked via UpdateJournal

## Map specific logic
Any areas not mentioned here has no special logic defined. However, if flag 364 is true (got the `WaspKey` in the `WaspKingdom3`), all maps except `WaspKingdom3` will have flag 657 set to true (exited `WaspKingdom3` with the key).

### `FishingVillage`

- If [discovery](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) 35 (Fishing Village) isn't unlocked yet, it is unlocked via UpdateJournal

### `WizardTowerStairs`

- If [discovery](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) 36 (Wizard's Tower) isn't unlocked yet, it is unlocked via UpdateJournal

### `BugariaOutskirtsSnakemouthCorridor2`

- If [flag](../../Flags%20arrays/flags.md) 41 is true (chapter 1 ended), [flag](../../Flags%20arrays/flags.md) 652 is set to true (fast travel from Ant Mines to Snakemouth Den becomes possible)

### `SnakemouthTreasureRoom`

- All [regionalflags](../../Flags%20arrays/Regionalflags.md) are reset to false

### `BarrenLandsRock`

- [flag](../../Flags%20arrays/flags.md) 452 is set to true (the crate in the pumpkin map disappears)

### `GiantLairBeforeBoss`

- If [flag](../../Flags%20arrays/flags.md) 667 and 668 are true (turned both dials of the oven at Giant's Lair), RenderSettings.fogColor is set to 331919 (mostly dark red)

### `SnakemouthTop`

- If [discovery](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) 18 (Healing Sophie) isn't unlocked yet, it is unlocked via UpdateJournal

### `UpperSnekMiddleRoom`

- If [discovery](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) 17 (Snakemouth's Lab) isn't unlocked yet, it is unlocked via UpdateJournal

### `UndergroundBar`

- If [discovery](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) 20 (Underground Tavern) isn't unlocked yet, it is unlocked via UpdateJournal

### `AntTunnels`

- If [discovery](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) 21 (The Ant Mines) isn't unlocked yet, it is unlocked via UpdateJournal

### `ChomperCave1`

- If [discovery](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) 22 (Chomper Cavern) isn't unlocked yet, it is unlocked via UpdateJournal

### `DesertDRSouthEntrance`

- [flag](../../Flags%20arrays/flags.md) 201 is set to true (rescued the merchant at Lost Sands). NOTE: While this is logically correct as far as the event is concerned, it doesn't do the equivalent side effect of setting a crystalbfflag to true meaning this doesn't fix a logical break, it locks out from completion
- [flag](../../Flags%20arrays/flags.md) 170 is set to true (arrived at Defiant Root for the first time)

### `RubberPrisonGym`

- [flag](../../Flags%20arrays/flags.md) 535 is set to false (disengages the gray levers)

### `StreamMountain4`

- [flag](../../Flags%20arrays/flags.md) 316 is set to false (the first water crystal at Stream Mountain is deactivated)

### `StreamMountain5`

- [flag](../../Flags%20arrays/flags.md) 492 is set to false (water crystal on the way to Tidal Wyrm at Stream Mountain is deactivated)

### `BeehiveOutside`

- If [discovery](../../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) 25 (The Bee Kingdom) isn't unlocked yet, it is unlocked via UpdateJournal

### `GiantLairBeforeBoss2`

- [flag](../../Flags%20arrays/flags.md) 666 is set to true (turned the right dial on the stove at Giant's Lair)
