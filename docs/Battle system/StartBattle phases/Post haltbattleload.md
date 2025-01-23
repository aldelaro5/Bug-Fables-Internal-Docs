# StartBattle - Post `haltbattleload`
This is the second of 3 phases of [StartBattle](../StartBattle.md).

This phase occurs after the `haltbattleload` yield, but before battle.`halfload` is set to true. Since the value is only visible after the first yield, the phase ends after yielding when `halfload` becomes true.

## Further initialisations
This part only contains miscellaneous initialisations logic:

- If the stageid is `FarGrasslands`, the existing RenderSettings.fogEndDistance is saved into `fogdist` while it gets overriden to 75.0 (this adds a foggy effect more suitable visuablly for this [BattleMap](../../Enums%20and%20IDs/BattleMaps.md))
- The money HUD is hidden by setting instance.showmoney to -1
- MainManager.RefreshHUDValues is called which forces the displayed hp, tp and berry count to their real ones
- If `tskybox` exists (meaning this is a retry as it was saved before the StartBattle call), RenderSettings.skybox is set to it
- All [NPCControl](../../Entities/NPCControl/NPCControl.md) have [StopForceBehavior](../../Entities/NPCControl/Notable%20methods/StopForceBehavior.md) called on them
- A frame is yielded
- The `LevelUp` music and `Gameover` sound are loaded, but not stored. This is purely to preload them to not get a stutter when they eventually play
- `damcounters` is reset to a new list
- `charmdance` is set to 2 elements being sprite 98 and 99 of `Sprites/Entities/moth0`. These are the 2 Charmy's sprites used when they performs a charm
- `scopeequipped` is set to whether or not the `HPScope` [medal](../../Enums%20and%20IDs/Medal.md) is equipped
- `dimmer` is set to a new Sprite of 1 pure black pixel named `MainDimmer` scaled to 100.0 on layer 5 (UI) uniformely with position Vector3.zero and with a pivot at (0.5, 0.5, 0.5). This covers the whole screen during the load
- If this isn't a retry (meaning it wasn't an existing battle):
    - The camera limits are removed, but saved to their temporary fields before
    - `oldcamoffset` is set to instance.`camoffset`
    - `oldcamrotation` is set to instance.`camangleoffset`
    - `oldcamtarget` is set to instance.`camtarget`
    - `oldcamspeed` is set to instance.`camspeed`
- [SetDefaultCamera](../Visual%20rendering/SetDefaultCamera.md) is called
- `chompylock` is set to false
- `lockmmatter` is set to false
- `gameover` is set to null
- `attackedally` is set to -1
- instance.`camspeed` is set to 99.0
- MainManager.parent (the `MainCam` object) angles are set to instance. `camangleoffset`
- `caller` is set to calledfrom

## Battle map creation
This part disables the overworld map and creates/enables the battle one:

- If the map is enabled:
    - MainManager.[RefreshEntities](../Actors%20states/RefreshEntities.md) is called
    - A frame is yielded
    - All `playerdata` entities have the following happen to each of them:
        - `flyinganim` is set to false
        - SetAnimForce is called
        - A frame is yielded
        - The entity.gameObject is disabled
    - The map is disabled
- `battlemap` is set to a new GameObject named `Battle`
- The corresponding `Prefabs/BattleMaps/X` battle map prefab is instantiated where X is the [BattleMaps](../../Enums%20and%20IDs/BattleMaps.md) enum name of the stageid and it is childed to `battlemap`
- All colliders in the battle map are disabled
- A BoxCollider is added to `battlemap` with center (0.0, -0.5, 0.0) and size (100.0, 1.0, 50.0). This is essentially the physical ground for both parties
- `battlemap`.layer is set to 8 (`Ground`)
- A frame is yielded

## Music setup
This part selects the music to play if any and plays it:

- MainManager.`music[0]` pitch is set to normal
- If MainManager.map.`nobattlemusic` is false or we are instance.`inevent`, the music is changed via MainManager.ChangeMusic with a 1.0 fadespeed where the clip varies depending on some conditions:
    - If the sent music isn't null, that will be the clip used
    - Otherwise, If the current [area](../../Enums%20and%20IDs/librarystuff/Areas.md) is `UpperSnakemouth`, `Battle6` will be used
    - Otherwise, if [flag](../../Flags%20arrays/flags.md) 348 (Start of chapter 5) is true and the current [area](../../Enums%20and%20IDs/librarystuff/Areas.md) is among `BarrenLands`, `FarGrasslands`, `WildGrasslands`, `RubberPrison`, `MetalLake` or `StreamMountain`, `Battle6` will be used
    - Otherwise, `Battle0` will be used

## Enemy party setup
This part goes through all the enemyids and initialises the corresponding `enemydata` element. The array is initialised to a new one with a length of enemyids.length for this purpose. The main portion of the data comes from [GetEnemyData](../../TextAsset%20Data/Enemies%20data.md) with entity creation.

From there, the following happens on each `enemydata` after setting it to the return of GetEnemyData:

- battleentity.`battleid` is set to the corresponding index of the `enemydata`
- `turnsnodamage` is set to -1
- battleentity.`alwaysactive` is set to true
- battle.`estimatedexp` is incremented by the `enemydata`'s `exp`
- If the enemy's `animid` (the [enemy](../../Enums%20and%20IDs/Enemies.md) id) is less than the amount of instance.`enemyencounter` elements (which is normally 256) and battleentity.`cotunknown` is false (it's not a Cave of Trials battle entity), the corresponding `enemyencounter` element's seen counter is incremented
- The battleentity position is determined:
    - x: This depends on the amount of enemy party member and the `enemydata` index of the element:
        - 2 enemies: 1.5 * `enemydata`'s index + 3.5
        - 3 enemies: 1.5 * `enemydata`'s index + 2.5
        - 4 enemies: 0.75 * `enemydata`'s index + 2.0
    - y: always 0.1
    - z: 0.0 for the 1st enemy, 0.15 for the 2nd, 0.30 for the 3rd and 0.45 for the 4th
- The battleentity gets childed to the `battlemap`
- battleentity.`hologram` gets set to the value of [flag](../../Flags%20arrays/flags.md) 162 (we are in a B.O.S.S or Cave of Trials session)
- `battlepos` is set to the battleentity's position
- Some [enemies](../../Enums%20and%20IDs/Enemies.md) have additional logic here:
    - `VenusBoss`: `extraentities` is initialised to a new array with one element being a new entity created via [CreateNewEntity](../../Entities/EntityControl/EntityControl%20Creation.md#createnewentity) with a name of `Venus`, an [animid](../../Enums%20and%20IDs/AnimIDs.md) of 80 (`Venus`) and a position of (0.0, 0.0, 4.0) childed to the `battlemap`. This entity gets further adjustements:
        - `hologram` gets set to the value of [flag](../../Flags%20arrays/flags.md) 162 (we are in a B.O.S.S or Cave of Trials battle), but if they have the [Holographic](../Damage%20pipeline/AttackProperty.md) property in their `weakness`, the value always gets set to true regardless
        - `alwaysactive` gets set to true
        - `activeonpause` gets set to true
        - A VenusBattle component gets added to the gameObject and its SetUp gets called on it with itself as the parent and the battleentity as the target
    - `Scarlet`: battleentity.`basesate` is set to 5 (`Angry`)
    - `BeeBoss`: `battlepos` and the battleentity position are set to (4.5, 0.0, 0.0)
    - `SandWyrmTail`: entity.`height` and entity.`initialheight` are set to 0.0
    - `Cenn` and `Pisci`: If [flags](../../Flags%20arrays/flags.md) 497 is true (beat them for the first time) , [eventondeath](../Actors%20states/Enemy%20features.md) is set to -1. If it's false, `AlwaysSurvive` is added to the [weakness](../Actors%20states/Enemy%20features.md#weakness). In either cases, `battlepos` and the battleentity position are set to (1.0, 0.0, 0.0)
    - `Spuder` and `SpuderReal`: If [flags](../../Flags%20arrays/flags.md) 41 is false (before the end of chapter 1) and either flags 613 (RUIGEE) or 614 (HARDEST) is true, the `hp` and `maxhp` are decreased by 15
    - `Zasp` and `Mothiva`: If [flags](../../Flags%20arrays/flags.md) 606 is true (after the second round of the colosseum in chapter 6), `hp` / `maxhp` are increased by 15, `hardatk` is incremented and `lockrelayreceive` is set to true
    - `Pitcher`: battleentity position is incremented by Vector3.right
    - `Maki`, `Kina` and `Yin`: If [flags](../../Flags%20arrays/flags.md) 614 is true (HARDEST is active), `hp` / `maxhp` are increased by 10 and `def` is incremented
- If the calledfrom.entity or the battleentity is `inice` while it's a `Krawler` or `CursedSkull` [enemy](../../Enums%20and%20IDs/Enemies.md):
    - `freezeres` is incremented by 70 (affects [freeze](../Actors%20states/BattleCondition/Freeze.md) inflictions)
    - battleentity.`inice` is set to true
    - [weakness](../Actors%20states/Enemy%20features.md#weakness) is set to a new list with one element being `HornExtraDamage`
- If battleentity.`forcefire` or we are in the `GiantLair` [area](../../Enums%20and%20IDs/librarystuff/Areas.md) except for the `GiantLairFridgeInside` [map](../../Enums%20and%20IDs/Maps.md) while the enemy is a `Krawler`, `CursedSkull` or `Cape`:
    - If instance.`partylevel` is less than 27 (meaning it's not maxed) and [flags](../../Flags%20arrays/flags.md) 613 is false (RUIGEE is inactive), the `exp` is incremented by the floored result of a lerp from 10.0 to 3.0 with a factor of instance.`partylevel` / 27.0
    - `freezeres` is set to 110 (immune to [freeze](../Actors%20states/BattleCondition/Freeze.md) inflictions)
    - `hp` and `maxhp` are increased by 3
    - [weakness](../Actors%20states/Enemy%20features.md#weakness) is set to a new list with one element being `Magic`
- A frame is yielded
- battleentity.[animstate](../../Entities/EntityControl/Animations/animstate.md) is set to its `basesate`

After, there are more setup logic, but this time, it's based on the entire `enemydata` array instead of each element:

- If it contains a `MotherChomper` [enemy](../../Enums%20and%20IDs/Enemies.md) with a length of 3, the `battlepos` are changed as follows depending on their `enemydata` indexes:
    - 0: (1.2, 0.0, 1.0)
    - 1: (2.7, 0.0, -0.5)
    - 2: (4.6, 0.0, -0.1)
- If it contains a `UltimaxTank` [enemy](../../Enums%20and%20IDs/Enemies.md), the `battlepos` of the first element is incremented by (0.0, 0.0, 1.0)
- If it contains a `SandWyrm` [enemy](../../Enums%20and%20IDs/Enemies.md), the first 2 elements gets a VenusBattle component added to their battleentity.gameObject with its SetUp called on them where the target is the battleentity itself and the parent being the other of the 2 battleentity. It also changes their `battlepos` depending on their `enemydata` indexes:
    - 0: (3.0, 0.0, 0.0)
    - 1: (5.9, 0.0, 0.0)
- All battleentity in `enemydata` gets their position set to their corresponding `battlepos`
- A frame is yielded

## Player party setup
This part initialises the `playerdata` elements for the battle. It first involves initialising `partyentities`, `tiredpart` and `receivedrelay` to new arrays with a length of the instance.`playerdata` length.

From there, the following fields gets initialised to each `playerdata`:

|Field|Initial value|
|----:|-------------|
|battleentity|A new entity created via [CreateNewEntity](../../Entities/EntityControl/EntityControl%20Creation.md#createnewentity) with the name `PlayerX` where X is the index of the element|
|`tired`|0|
|`cantmove`|0, but if the `LastWind` [medal](../../Enums%20and%20IDs/Medal.md) is equipped on them, the value is -1 (2 actor turns on the first main turn)|
|`charge`|0|
|`atkdownonloseatkup`|false|
|battleentity's position|Depends on the `playerdata` index:<ul><li>0: (-3.0, 0.1, 0.0)</li><li>1: (-5.0, 0.1, 1.0)</li><li>2: (-6.0, 0.1, 0.0)</li></ul>|
|battleentity's tag|`Player`|
|battleentity.[animid](../../Enums%20and%20IDs/AnimIDs.md)|The corresponding `trueid` (see [battle party addressing](../playerdata%20addressing.md#methods-of-addressing-durring-battle) for why this happens)|
|battleentity.`flip`|true|
|battleentity.`battle`|true|
|`moreturnnextturn`|0|
|`turnssincedeath`|0|
|`turnsalive`|0|
|battleentity.`overridejump`|true|
|battleentity.`alwaysactive`|true|
|`eatenby`|null|
|`didnothing`|false|
|battleentity.`nocondition`|false|
|`isnumb`|false|
|`pointer`|The `playerdata` index|
|`plating`|Whether or not the player party member has the `Plating` [medal](../../Enums%20and%20IDs/Medal.md) equipped|
|`isasleep`|false|
|`isdefending`|false|
|battleentity.`battleid`|The `playerdata` index|
|battleentity.parent|`battlemap` (gets childed to it)|
|battleentity.gameObject.layer|9 (`Follower`)|
|battleentity.`emoticonoffset`|(0.0, 1.8 + 0.25 * the `playerdata` index, 0.0)|
|battleentity.`destroytype`|[PlayerDeath](../../Entities/EntityControl/Notable%20methods/Death.md#playerdeath)|
|battleentity.[animstate](../../Entities/EntityControl/Animations/animstate.md)|13 (`BattleIdle`)|
|`eventondeath`|-1|
|`cursoroffset`|(0.0, 2.5, 0.0)|
|`haspassed`|false|
|`weakness`|New empty list|
|[condition](../Actors%20states/Conditions.md#conditions)|New empty list|
|battleentity.`dead`|If `hp` is 0, it's set to true|
|battleentity.`hologram`|If the `HoloCloak` [medal](../../Enums%20and%20IDs/Medal.md) is equipped on them, this is set to true alongside an UpdateSpriteMat call which updates their material to MainManager.`holosprite`|

Additionally, the following also happens for each `playerdata`:

- The corresponding `partyentities` element gets set to the battlentity
- The corresponding battle.`tiredpart` gets initialised to the transform of a new `Prefabs/Particles/Tired` with the following:
    - Childed to the battleentity
    - Local position being the `cursoroffset` + (0.0, -5.0, 0.0)
    - Angles of (-45.0, 270.0, 0.0)
    - Disabled for now

After initialising all player party members, MainManager.[ApplyBadges](../ApplyBadges.md) is called.

## Chompy setup
This part involves the initialisation of the `chompy` field. It happens if [flags](../../Flags%20arrays/flags.md) 402 is true (Chompy is with Team Snakemouth).

`chompy` is initialised to a new entity created via [CreateNewEntity](../../Entities/EntityControl/EntityControl%20Creation.md#createnewentity) with name `chompy` with the `ChompyChan` [animid](../../Enums%20and%20IDs/AnimIDs.md) with a position of (-0.5, 0.0, 1) with the following:

- Childed to the `battlemap`
- gameObject.layer set to 9 (`Follower`)
- `flip` set to true
- `battle` set to true

## AI setup
This part initialises battle `aiparty` which is a battle AI. A battle AI is an entity that sits closely to the player party, but has scripted actions after the player party's actions (and `chompy` if applicable) is over. They are only added under specific conditions, usually involving having a follower. 

The method used for this is called AddAI and it initalises `aiparty` to a new entity created via [CreateNewEntity](../../Entities/EntityControl/EntityControl%20Methods.md#createnewentity) with name `ai party` using the matching [animid](../../Enums%20and%20IDs/AnimIDs.md) and am [animstate](../../Entities/EntityControl/Animations/animstate.md) with position (-3.0, 0.0, 2.0) childed to the `battlemap` and with a `flip` and `battle` field set to true. 

The method takes an [animid](../../Enums%20and%20IDs/AnimIDs.md) and am [animstate](../../Entities/EntityControl/Animations/animstate.md) to set to the entity (the animstate parameter is called basestate, but it actually sets the `animstate` field).

While technically, the method can be called multiple time, this isn't supported and will likely result in entities not being tracked by the game should this happen.

Here are the different AIs added and their conditions by their animid (`aiparty` starts with a value of null):

- `Maki` is added with [animstate](../../Entities/EntityControl/Animations/animstate.md) 13 (`BattleIdle`) if any of the following is true:
    - The MainManager.map.`mapid` is the `FGClearing` [map](../../Enums%20and%20IDs/Maps.md)
    - [flags](../../Flags%20arrays/flags.md) 594 is true (Chapter 7 when maki should be present in battle) 
    - `Maki` is a [tempfollower](../../SetText/Individual%20commands/Addfollower.md) while the current [area](../../Enums%20and%20IDs/librarystuff/Areas.md) is `FarGrasslands` or `WildGrasslands` or the map is the `TestRoom`
- `KungFuMantis` is added with [animstate](../../Entities/EntityControl/Animations/animstate.md) 5 (`Angry`) if they are a [tempfollower](../../SetText/Individual%20commands/Addfollower.md)
- `Madeleine` is added with [animstate](../../Entities/EntityControl/Animations/animstate.md) 0 (`Idle`) if they are a [tempfollower](../../SetText/Individual%20commands/Addfollower.md)
- `AntCapitain` is added with [animstate](../../Entities/EntityControl/Animations/animstate.md) 13 (`BattleIdle`) if map.`mapid` is the `AbandonedCity` [map](../../Enums%20and%20IDs/Maps.md) and [flags](../../Flags%20arrays/flags.md) 400 is true (reserved temporary flag). If they are added, it also causes every element of `enemydata` to have MainManager.[SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) called twice on them as the entity with `AttackUp` and `DefenseUp` for 999999 actor turns
- `LadybugKnight` is added if MainManager.`lastevent` is 47 meaning this battle was caused by [event](../../Enums%20and%20IDs/Events.md) 47 (Chapter 2 approaching the tunnel entrance to the Golden Path)
- `Gen` is added if MainManager.`lastevent` is 93 meaning this battle was caused by [event](../../Enums%20and%20IDs/Events.md) 93 (Chapter 3 approaching the area where the merchants gets ambushed by bandits and wasps)

### `Helper` [medal](../../Enums%20and%20IDs/Medal.md) logic
If the `helper` medal is equipped and `aiparty` is still null at this point (meaning none of the above conditions were fufilled to get an `aiparty`), it's still possible for an `aiparty` to be assigned, but it will be selected randomly amongst all the possible ones. `aiparty` will stay at null (none) if the medal isn't equipped or it is, but no `aiparty` exists in the possible pool.

For an `aiparty` to be in the pool, each requires to have some [flags](../../Flags%20arrays/flags.md) set to true. Here are all the possible ones with their flags slot required:

|animid|animstate|Required flags slot(s)|
|-----:|---------|---------------------|
|`KungFuMantis`|5 (`Angry`)|514 (Completed the Help Me Get it Back! quest)|
|`Gen`|0 (`Idle`)|498 (Completed the Explorer Check! quest)|
|`Maki`|13 (`BattleIdle`)|610 (Beaten Maki’s team for the first time)|
|`LadybugKnight`|0 (`Idle`)|135 (Completed the Requesting Assistance quest)|
|`ShielderAnt`|13 (`BattleIdle`)|135 (Completed the Requesting Assistance quest)|
|`AntCapitain`|13 (`BattleIdle`)|709 (Completed the Loose Ends quest)|
|`Madeleine`|0 (`Idle`)|391 (Completed the Butler Missing Again! quest)|
|`Zasp`|5 (`BattleIdle`)|298 (Chapter 4, beat the Dune Scorpion) and 189 (Completed the Lost Item quest)|

If at least one exists in the possible pool, then a random one is selected and AddAI will be called with the selected party. This is also followed by their battleentity.`hologram` set to true with an UpdateSpriteMat call which will set their `sprite` material to MainManager.`holosprite`.

### `Gen` special handling as `aiparty`
This particular `aiparty` is special because it requires 2 entities, but as said earlier, the AI party system doesn't allow to have multiple entities. As a workaround, if the animid of the `aiparty` that was added is 47 (`Gen`), the following will happen:

- A new entity named `eri` is created with animid 48 (`Eri`) using [CreateNewEntity](../../Entities/EntityControl/EntityControl%20Creation.md#createnewentity) childed to the `aiparty` positioned at the `aiparty` + (-1.15, 0.0, 0.5) with a z scale multiplied by -1.0
- The entity's `hologram` is set to true followed by an UpdateSpriteMat call which will set their `sprite` material to MainManager.`holosprite`
- The entity's `flip` and `overrideflip` are set to true so they will look to the right

## Last pre `halfload` initialisations
This part only contains miscellaneous initialisations logic before `halfload` is set to true:

- Yield for a frame
- MainManager.[RefreshEntities](../Actors%20states/RefreshEntities.md) is called
- instance.`camspeed` is set to 0.1
- If `actiontext` doesn't exist, it is created as a SpriteRenderer component added to a new GameObject named `ActionText` and the following:
    - Childed to the `GUICamera`
    - Local position of (-5.0, 4.0, 10.0)
    - Scale of (1.15, 1.75, 1.1)
    - No angles
    - sprite of `guisprites[0]` (white rectangle)
    - gameObject.layer of 5 (UI)
    - sortingOrder of -1
    - color of pure white with 75% opacity
- CreateCursor is called which initialises `cursor` to a new GameObject named `cursor` with the following:
    - A SpriteRenderer with the sprite being `cursorsprite[0]` (the leaf cursor sprite) with a sortingOrder of -20
    - A SpriteBounce with MessageBounce called on it
    - Childed to the `battlemap`
    - gameObject.layer set to 15 (`3DUI`)
    - Angles of (0.0, 0.0, 270.0)
- `avaliableplayers` is set to the return of MainManager.[GetFreePlayerAmmount](../Actors%20states/Player%20party%20members/GetFreePlayerAmmount.md)
- A frame is yielded
- The `cursor` position is set to (0.0, 999.0, 0.0)
- The SpriteRenderer material of the `cursor` has its renderQueue set to 10000
- [UpdateText](../Visual%20rendering/UpdateText.md) is called
- MainManager.[RefreshSkills](../RefreshSkills.md) is called
- RefreshAllData is called which sets `alldata` to a new list which consists of all the `playerdata` followed by all the `enemydata` appended together and it also resets all enemy party member's `blockTimes` to 0
- If the `fronticon` doesn't exist, it is initialised to the SpriteRenderer of a new UI object with the name `attackicon` childed to the `GUICamera` with position of Vector.zero, size of Vector3.one / 3.0, sortingOrder of 20 and with a sprite of `guisprites[63]` (the red arrow of the front icon in battle)
- A frame is yielded
- We exit the `pause` entered earlier
- If there's more than one `playerdata` element, then as long as the first `partypointer` doesn't match the first instance.`partyorder` (meaning the `battle` doesn't point to the leading party member), a [SwitchParty](../Battle%20flow/Action%20coroutines/SwitchParty.md) action coroutine is started with fast (which ends up calling MainManager.SwitchParty immediately) followed by a frame yield until they both match. NOTE: It does mean the coroutine calls can be spammed, but in theory, the while condition gets updated information everytime since the StartCoroutine call immediately causes the party to be updated. For more information on why this is done, check [battle party addressing](../playerdata%20addressing.md#methods-of-addressing-durring-battle)
- If the `StrongStart` [medal](../../Enums%20and%20IDs/Medal.md) is equipped on a party member, the corresponding `playerdata` gets its `cantmove` decremented which gives it an additional turn
- UpdateConditionIcons is called which calls UpdateConditionBubbles on all battleentity (all `playerdata` with right to false and all `enemydata` with `hp` above 0 with right to true)
- All frames are yielded while `action` is true until it goes to false (this is done to ensure all remainings SwitchParty calls done earlier are fully finished)
- [RefreshEXP](../Visual%20rendering/RefreshEXP.md) is called
- SetLastTurns is called which resets `lastturns` to a new aray with the length being the amount of free players - 1 and all elements being -1 (this resets the player selection cycle)
- `halfload` is set to true
- All `playerdata` battleentity have their `rigid` gravity enabled, `onground` set to true and [animstate](../../Entities/EntityControl/Animations/animstate.md) set to 13 (`BattleIdle`)
- 0.1 seconds are yielded
