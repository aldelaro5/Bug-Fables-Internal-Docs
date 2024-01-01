# StartBattle
This is a public coroutine that serves as the only way for the game to start a battle.

```cs
public static IEnumerator StartBattle(int[] enemyids, int stageid, int adv, string music, NPCControl calledfrom, bool canescape)
```

## Parameters

- `enemyids`: The list of [enemies](../Enums%20and%20IDs/Enemies.md) id that the player party will be against. If it contains more than 4 elements, all the ones after the 4th one will be placed in `extraeenmies` and the fight will be against the first 4 for now by setting `enemydata` to them. This is subject to be overriden by EnemyCheck during the start process
- `stageid`: The [BattleMaps](../Enums%20and%20IDs/BattleMaps.md) to use for the battle, will be overriden to the ice version if the calledfrom.entity is `inice` and the stageid is `SandCastle` or `SandCastleDark`. If -1 is sent, the stage is determined by instance.`battlestage` which the MapControl initialised on its Start
- `adv`: Decides whose party has the advantage ???. This will be set to battle.`sadv` TODO: the values seems arbitrary, but 3 seems to be in favor of the enemy...
- `music`: The name of the music to use for the battle. This is ignored if map.`nobattlemusic` is true. If it's null, the music is determined automatically based on the [BattleMaps](../Enums%20and%20IDs/BattleMaps.md) and some conditions
- `calledfrom`: The [Enemy](../Entities/NPCControl/Enemy.md) NPCControl that caused this battle if any. If it's null, it means no NPCControl caused it and the battle was started manually. This will be set to battle.`caller`
- `canescape`: Whether the player is allowed to flee or not. This will be set to battle.`canflee`

## Procedure
The procedure for starting a battle is very complex and there are multiple phases to it. The caller is able to interupt it at 2 places to configure the battle further: one early on by setting MainManager.`haltbattleload` to true and one later on by yielding on battle.`halfload` becoming true. All of the phases and the interuptions points will be outlined in this document. They occur in order they are described.

### Pre `haltbattleload`
This section is the first that ever happens. It ends by yielding as long as `haltbattleload` is true which can be set by the caller:

#### instance.`battle` init
This part does basic initialisation and creates the instance.`battle` if needed:

- The current MainManager.`screenshake` is zeroed out
- MainManager.`battleenemyfled` is set to false
- instance.`lastdefeated` is reset to a new list
- All [OverWorldProjectile](../Entities/NPCControl/ActionBehaviors/ShootProjectile.md#an-overview-of-overworldprojectile) are destroyed
- [ItemList](../ItemList/ItemList%20State%20Machine.md)'s `inlist` is set to false
- The `camanglespeed` is set to 0.1 and `camanglechange` to false
- MainManager.`battleresult` is set to true
- MainManager.`battlenoexp` is reset to false
- The discovery HUD element is hidden by setting instance.discoveryhud to 0.0
- CancelAction is called on the player
- If it's not an existing battle, MainManager.`battle` is set to a new BattleControl component added to the instance's gameObject. From now on, all BattleControl fields comes from that instance

#### Start data init
This part only occurs when `saveddata` is false which can only happen when the battle was started for the first time because this field is only set to true at the very end of StartBattle.

This part initialises the `sdata` which is a [StartUpData](StartUpData.md) that persists informations about the battle and restored when it's retried. A retry involves calling StartBattle when the current one still exists which doesn't need this part's logic to occur:

If calledfrom is null or its `eventid` is 0 or below EnemyCheck is called which can modify the enemyids using 2 special overrides.

The first one involves a potential override to `GoldenSeedling` if instance.`inevent` is false and [flags](../Flags%20arrays/flags.md) 41 is true (chapter 1 ended). This is only applicable for the following [enemies](../Enums%20and%20IDs/Enemies.md):

- `Seedling`
- `FlyingSeedling`
- `Acornling`
- `Underling`
- `Plumpling`
- `Flowering`

Any of these have a 1% chance to changed to a `GoldenSeedling`. However, there are 2 cases where these odds increases applied in order:

- If the current [map](../Enums%20and%20IDs/Maps.md) is `SeedlingHaven`, the odds are 7.5%
- If the `Seedling` [medal](../Enums%20and%20IDs/Medal.md) is equipped, it multiplies the base odds by 2.5 (meaning 2.5% or 18.75% at `SeedlingHaven`)

The second override applies if any [enemy](../Enums%20and%20IDs/Enemies.md) is a `SandWyrm` or a `SandWyrmTail`. If this is the case, the entire enemyids array is overriden to be 2 elements with the first being `SandWyrm` and the second being `SandWyrmTail`

After, StartData is called:

- If the player is `dashing`, StopDash is called without hitting a wall
- `sdata` is first reset to default values
- All `sdata` fields except `psc`, `esc` and `enemyweakness` are initialised to their respective values
- From there, GetStartConditions is invoked in 2.5 seconds which completes this saving process by setting `sdata.psc`, `sdata.esc` and `sdata.enemyweakness` to their respective values (the whole thing is enclosed in a try catch block whose catch will invoke it again in 0.5 seconds).

Finally, [flagvar](../Flags%20arrays/flagvar.md) 40 (number of battles fought) is incremented

#### Essential setups
This part will do further initialisation:

- `reservedata` is reset to a new list
- `hideenemyhp` is reset to false
- If the current map.`music[0]` exists and map.`musicid` is not negative, `music[0]`.clip is saved into `overworldmusic` (its time too in `overmusic` if MainManager.`keepmusicafterbattle` is true) and on top of this, if its name doesn't match the sent music while map.`nobattlemusic` is false, FadeMusic is called with a speed of 0.025
- `canflee` is set to the sent canescape
- `currentaction` is set to `BaseAction` (the main vine action menu)
- `currentchoice` is set to `Attack`
- `option` is set to 0
- `action` is set to false
- `alreadyending` is set to false
- `expreward` is set to 0
- `moneyreward` is set to 0
- `partypointer` is set to a new array with {0, 1, 2}
- We enter a `pause`
- `longcancel` is set to the return value of InputIO.LongButton with input 5 (Cancel) which determines if the input rendering should be a wide one or not
- MainManager.[ApplyBadges](ApplyBadges.md) is called
- MainManager.[ApplyStatBonus](ApplyStatBonus.md) is called
- StopMovingEntities is called on the map

#### Battle map and extra enemies setup
This section determins the battle map and the `extraenemies` if needed:

- If the sent stage is -1, it is overriden to the instance.`battlestage` (the value got set on the last MapControl.Start which sets it to the map.`battlemap`)
- If the stageid is `SandCastle` or `SandCastleDark` while calledfrom's entity was `inice`, the stageid is incremented which sets it to `SandCastleIce` or `SandCastleDarkIce` respectively
- If the sent enemyids has more than 4 elements, all the elements after the 4th one are placed in order in `extraenemies` (after clearing it) and enemyids is overriden to only have the first 4 elements

#### Battle in transition
This section setup the battle in transition:

- A `BattleStart0` sound is played (`BattleStart3` if the map.`battleleaftype` is `Bee`) at 0.65 volume
- If the sent adv is 3 while we aren't `inevent`:
    - A `Hurt` sound is played followed by a `AtkFail` one
    - HurtParticle is called on the player position + Vector3.up with the `Hurt` sound (meaning the same sound will play twice on that frame)
    - All `playerdata` entities have their [animstate](../Entities/EntityControl/Animations/animstate.md) set to 11 (`Hurt`) with a SetAnimForce call which will force a [SetAnim](../Entities/EntityControl/Animations/SetAnim.md) call with force which renders the animation immediately
- If the player.entity.`sound` exists, it is stopped
- PlayTransition is called with id 2 (the battle in transition), the data being the map.`battleleaftype` at 0.075 speed and the color being the map.`battleleafcolor` unless adv was 3 when we aren't `inevent` where the color is overriden to pure red
- 1.5 seconds are yieled which lets the transition renders
- If MainManager.`haltbattleload` is true, all frames are yielded until it goes to false

### Post `haltbattleload`
This section occurs after the `haltbattleload` yield, but before battle.`halfload` is set to true:

#### Further initialisations
This part only contains miscellaneous initialisations logic:

- If the stageid is `FarGrasslands`, the existing RenderSettings.fogEndDistance is saved into `fogdist` while it gets overriden to 75.0 TODO ???
- The money HUD is hidden by setting instance.showmoney to -1
- MainManager.RefreshHUDValues is called which forces the displayed hp, tp and berry count to their real ones
- If `tskybox` exists (meaning this is a retry as it was saved before the StartBattle call), RenderSettings.skybox is set to it
- All [NPCControl](../Entities/NPCControl/NPCControl.md) have [StopForceBehavior](../Entities/NPCControl/Notable%20methods/StopForceBehavior.md) called on them
- A frame is yielded
- The `LevelUp` music and `Gameover` sound are loaded, but not stored. This is purely to preload them to not get a stutter when they eventually play
- `damcounters` is reset to a new list
- `charmdance` is set to 2 elements being sprite 98 and 99 of `Sprites/Entities/moth0`. These are the 2 Charmy's sprites used when they performs a charm
- `scopeequipped` is set to whether or not the `HPScope` [medal](../Enums%20and%20IDs/Medal.md) is equipped
- `dimmer` is set to a new Sprite of 1 pure black pixel named `MainDimmer` scaled to 100.0 on layer 5 (UI) uniformely with position Vector3.zero and with a pivot at (0.5, 0.5, 0.5)
- If this isn't a retry (meaning it wasn't an existing battle):
    - The camera limits are removed, but saved to their temporary fields before
    - `oldcamoffset` is set to instance.`camoffset`
    - `oldcamrotation` is set to instance.`camangleoffset`
    - `oldcamtarget` is set to instance.`camtarget`
    - `oldcamspeed` is set to instance.`camspeed`
- [SetDefaultCamera](Visual%20rendering/SetDefaultCamera.md) is called
- `chompylock` is set to false
- `lockmmatter` is set to false
- `gameover` is set to null
- `attackedally` is set to -1
- instance.`camspeed` is set to 99.0
- MainManager.parent (the `MainCam` object) angles are set to instance. `camangleoffset`
- `caller` is set to calledfrom

#### Battle map creation
This part disables the current map and enables the battle one which is created:

- If the map is enabled:
    - MainManager.[RefreshEntities](Actors%20states/RefreshEntities.md) is called
    - A frame is yielded
    - All `playerdata` entities have the following happen to each of them:
        - `flyinganim` is set to false
        - SetAnimForce is called
        - A frame is yielded
        - The entity.gameObject is disabled
    - The map is disabled
- `battlemap` is set to a new GameObject named `Battle`
- The corresponding `Prefabs/BattleMaps/X` battle map prefab is instantiated where X is the [BattleMaps](../Enums%20and%20IDs/BattleMaps.md) enum name of the stageid and it is childed to `battlemap`
- All colliders in the battle map are disabled
- A BoxCollider is added to `battlemap` with center (0.0, -0.5, 0.0) and size (100.0, 1.0, 50.0)
- `battlemap`.layer is set to 8 (`Ground`)
- A frame is yielded

#### Music setup
This part selects the music to play if any and plays it:

- MainManager.`music[0]` pitch is set to normal
- If map.`nobattlemusic` is false or we are `inevent`, the music is changed via MainManager.ChangeMusic with a 1.0 fadespeed where the clip varies depending on some conditions:
    - If the sent music isn't null, that will be the clip used
    - Otherwise, If the current [area](../Enums%20and%20IDs/librarystuff/Areas.md) is `UpperSnakemouth`, `Battle6` will be used
    - Otherwise, if [flag](../Flags%20arrays/flags.md) 348 (Start of chapter 5) is true and the current [area](../Enums%20and%20IDs/librarystuff/Areas.md) is among `BarrenLands`, `FarGrasslands`, `WildGrasslands`, `RubberPrison`, `MetalLake` or `StreamMountain`, `Battle6` will be used
    - Otherwise, `Battle0` will be used

#### Enemy party setup
This part goes through all the enemyids and initialises the corresponding `enemydata` element. The array is initialised to a new array with a length of enemyids.length for this purpose. The main portion of the data comes from [GetEnemyData](../TextAsset%20Data/Enemies%20data.md) with entity creation.

From there, the following happens on the `enemydata`:

- battleentity.`battleid` is set to the corresponding index
- `turnsnodamage` is set to -1
- battleentity.`alwaysactive` is set to true
- battle.`estimatedexp` is incremented by the `enemydata.exp`
- If the enemy's `animid` (the [enemy](../Enums%20and%20IDs/Enemies.md) id) is less than the amount of instance.`enemyencounter` elements (which is normally 256) and battleentity.`cotunknown` is false (it's not a Cave of Trials battle entity), the corresponding `enemyencounter` element's seen counter is incremented
- The battleentity position is determined where it depends on the amount of enemies and this one's position:
    - x: This depends on the amount of enemies and half of the `enemydata`'s size clamped from 1.0 to 1.5:
        - 2: 1.5 + 3.5 * enemy index * the clamped size
        - 3: 1 + 2.5 * enemy index * the clamped size
        - 4: 0.5 + 2.0 * enemy index * the clamped size
    - y: always 0.1
    - z: 0.0 for the 1st enemy, 0.15 for the 2nd, 0.30 for the 3rd and 0.45 for the 4th
- The battleentity gets childed to the `battlemap`
- battleentity.`hologram` gets set to the value of [flag](../Flags%20arrays/flags.md) 162 (we are in a B.O.S.S or Cave of Trials battle)
- `battlepos` is set to the battleentity position
- Some enemies have additional logic here:
    - `VenusBoss`: `extraentities` is initialised to a new array with one element being a new entity created via [CreateNewEntity](../Entities/EntityControl/EntityControl%20Creation.md#createnewentity) with a name of `Venus`, an [animid](../Enums%20and%20IDs/AnimIDs.md) of 80 (`Venus`) and a position of (0.0, 0.0, 4.0) childed to the `battlemap`. This entity gets further adjustements:
        - `hologram` gets set to the value of [flag](../Flags%20arrays/flags.md) 162 (we are in a B.O.S.S or Cave of Trials battle)
        - `alwaysactive` gets set to true
        - `activeonpause` gets set to true
        - A VenusBattle component gets added to the gameObject and its SetUp gets called on it with itself as the parent and the battleentity as the target
    - `Scarlet`: battleentity.`basesate` is set to 5 (`Angry`)
    - `BeeBoss`: `battlepos` and the battleentity position are set to (4.5, 0.0, 0.0)
    - `SandWyrmTail`: entity.`height` and entity.`initialheight` are set to 0.0
    - `Cenn` and `Pisci`: If [flags](../Flags%20arrays/flags.md) 497 is true (beat them for the first time) , `eventondeath` is set to -1. If it's false, `AlwaysSurvive` is added to `weakness`. In either cases, `battlepos` and the battleentity position are set to (1.0, 0.0, 0.0)
    - `Spuder` and `SpuderReal`: If [flags](../Flags%20arrays/flags.md) 41 is false (before the end of chapter 1) and either flags 613 (RUIGEE) or 614 (HARDEST) is true, the `hp` and `maxhp` are decreased by 15
    - `Zasp` and `Mothiva`: If [flags](../Flags%20arrays/flags.md) 606 is true (after the second round of the colosseum in chapter 6), `hp` / `maxhp` are increased by 15, `hardatk` is incremented and `lockrelayreceive` is set to true
    - `Pitcher`: battleentity position is incremented by Vector3.right
    - `Maki`, `Kina` and `Yin`: If [flags](../Flags%20arrays/flags.md) 614 (HARDEST) is true, `hp` / `maxhp` are increased by 10 and `def` is incremented
- If the calledfrom.entity or the battleentity is `inice` while it's a `Krawler` or `CursedSkull` [enemy](../Enums%20and%20IDs/Enemies.md):
    - `freezeres` is incremented by 70
    - battleentity.`inice` is set to true
    - `weakness` is set to a new list with one element being `HornExtraDamage`
- If battleentity.`forcefire` or we are in the `GiantLair` [area](../Enums%20and%20IDs/librarystuff/Areas.md) except for the `GiantLairFridgeInside` [map](../Enums%20and%20IDs/Maps.md) while the enemy is a `Krawler`, `CursedSkull` or `Cape`:
    - If instance.`partylevel` is less than 27 (meaning it's not maxed) and [flags](../Flags%20arrays/flags.md) 613 (RUIGEE) is false, the `enemydata.exp` is incremented by the floored result of a lerp from 10.0 to 3.0 with a factor of instance.`partylevel` / 27.0
    - `freezeres` is set to 110
    - `hp` and `maxhp` are increased by 3
    - `weakness` is set to a new list with one element being `Magic`
- A frame is yielded
- battleentity.[animstate](../Entities/EntityControl/Animations/animstate.md) is set to its `basesate`

After, there are more setup logic, but this time, it's based on the entire `enemydata` array instead of each element:

- If it contains a `MotherChomper` [enemy](../Enums%20and%20IDs/Enemies.md) with a length of 3, the `battlepos` are changed as follows:
    - 0: (1.2, 0.0, 1.0)
    - 1: (2.7, 0.0, -0.5)
    - 2: (4.6, 0.0, -0.1)
- If it contains a `UltimaxTank` [enemy](../Enums%20and%20IDs/Enemies.md), the `battlepos` of the first element is incremented by (0.0, 0.0, 1.0)
- If it contains a `SandWyrm` [enemy](../Enums%20and%20IDs/Enemies.md), the first 2 elements gets a VenusBattle component added to their battleentity.gameObject with its SetUp called on them where the target is the battleentity itself and the parent being the other of the 2 battleentity. It also changes their `battlepos`:
    - 0: (3.0, 0.0, 0.0)
    - 1: (5.9, 0.0, 0.0)
- All battleentity in `enemydata` gets their position set to their corresponding `battlepos`
- A frame is yielded

#### Player party setup
This part initialises the `playerdata` elements for the battle. It first involves initialising `partyentities`, `tiredpart` and `receivedrelay` to new arrays with a length of the instance.`playerdata` length.

From there, the following happens to each `playerdata`:

- battleentity is set to a new entity created via [CreateNewEntity](../Entities/EntityControl/EntityControl%20Creation.md#createnewentity) with the name `PlayerX` where X is the index of the element
- `tired`, `cantmove` and `charge` gets set to 0
- `atkdownonloseatkup` gets set to false
- battleentity position gets set to the corresponding `partypos` which depends on the index:
    - 0: (-3.0, 0.1, 0.0)
    - 1: (-5.0, 0.1, 1.0)
    - 2: (-6.0, 0.1, 0.0)
- battleentity tag gets set to `Player`
- battleentity.[animid](../Enums%20and%20IDs/AnimIDs.md) gets set to the corresponding `playerdata.trueid`
- battleentity.`flip` gets set to true
- battleentity.`battle` gets set to true
- `moreturnnextturn` gets set to 0
- `turnssincedeath` gets set to 0
- `turnsalive` gets set to 0
- battleentity.`overridejump` gets set to true
- battleentity.`alwaysactive` gets set to true
- `eatenby` gets set to null
- `didnothing` gets set to false
- battleentity.`nocondition` gets set to false
- `isnumb` gets set to false
- `pointer` gets set to the index of the element
- `plating` gets set to whether or not the `trueid` has the `Plating` [medal](../Enums%20and%20IDs/Medal.md) equipped
- `isasleep` gets set to false
- `isdefending` gets set to false
- battleentity.`battleid` gets set to the index of the element
- The battle entity gets childed to the `battlemap`
- battleentity.gameObject.layer gets set to 9 (`Follower`)
- battleentity.`emoticonoffset` gets set to (0.0, 1.8 + 0.25 * the index of the element, 0.0)
- battleentity.`destroytype` gets set to [PlayerDeath](../Entities/EntityControl/Notable%20methods/Death.md#playerdeath)
- battleentity.[animstate](../Entities/EntityControl/Animations/animstate.md) gets set to 13 (`BattleIdle`)
- `eventondeath` gets set to -1
- `cursoroffset` gets set to (0.0, 2.5, 0.0)
- The corresponding `partyentities` element gets set to the battlentity
- `haspassed` gets set to false
- `weakness` gets set to a new empty list
- `condition` gets set to a new empty list
- The corresponding battle.`tiredpart` gets initialised to the transform of a new `Prefabs/Particles/Tired` with the following:
    - Childed to the battleentity
    - Local position being the `cursoroffset` + (0.0, -5.0, 0.0)
    - Angles of (-45.0, 270.0, 0.0)
    - Disabled for now
- If the `hp` is 0, battleentity.`dead` is set to true

Finally, MainManager.[ApplyBadges](ApplyBadges.md) is called.

#### Chompy setup
This part involves the initialisation of the `chompy` field. It happens if [flags](../Flags%20arrays/flags.md) 402 is true (Chompy is with Team Snakemouth).

`chompy` is initialised to a new entity created via [CreateNewEntity](../Entities/EntityControl/EntityControl%20Creation.md#createnewentity) with name `chompy` with the `ChompyChan` [animid](../Enums%20and%20IDs/AnimIDs.md) with a position of (-0.5, 0.0, 1) with the following:

- Childed to the `battlemap`
- gameObject.layer set to 9 (`Follower`)
- `flip` set to true
- `battle` set to true

#### AI setup
This part setup battle `aiparty` which is a battle AI. A battle AI is an entity that sits closely to the player party, but has scripted actions after the player party's turn is over. They are only added under specific conditions, usually involving having a follower. 

The method used for this is called AddAI and it initalises `aiparty` to a new entity created via [CreateNewEntity](../Entities/EntityControl/EntityControl%20Methods.md#createnewentity) with name `ai party` using the matching  with position (-3.0, 0.0, 2.0) childed to the `battlemap` and with a `flip` and `battle` field set to true. The method takes an [animid](../Enums%20and%20IDs/AnimIDs.md) and am [animstate](../Entities/EntityControl/Animations/animstate.md) to set to the entity (the animstate parameter is called basestate, but it actually sets the `animstate` field).

While technically, the method can be called multiple time, this isn't supported and will likely result in entities not being tracked by the game should this happen.

Here are the different AIs added and their conditions:

- `Maki` is added with [animstate](../Entities/EntityControl/Animations/animstate.md) 13 (`BattleIdle`) if any of the following is true:
    - The map.`mapid` is the `FGClearing` [map](../Enums%20and%20IDs/Maps.md)
    - [flags](../Flags%20arrays/flags.md) 594 is true (Chapter 7 when maki should be present in battle) 
    - `Maki` is a [tempfollower](../SetText/Individual%20commands/Addfollower.md#Remarks) while the current [area](../Enums%20and%20IDs/librarystuff/Areas.md) is `FarGrasslands` or `WildGrasslands` or the map is the `TestRoom`
- `KungFuMantis` is added with [animstate](../Entities/EntityControl/Animations/animstate.md) 5 (`Angry`) if they are a [tempfollower](../SetText/Individual%20commands/Addfollower.md#Remarks)
- `Madeleine` is added with [animstate](../Entities/EntityControl/Animations/animstate.md) 0 (`Idle`) if they are a [tempfollower](../SetText/Individual%20commands/Addfollower.md#Remarks)
- `AntCapitain` is added with [animstate](../Entities/EntityControl/Animations/animstate.md) 13 (`BattleIdle`) if map.`mapid` is the `AbandonedCity` [map](../Enums%20and%20IDs/Maps.md) and [flags](../Flags%20arrays/flags.md) 400 is true (reserved temporary flag). If they are added, it also causes every element of `enemydata` to have MainManager.[SetCondition](Actors%20states/Conditions%20methods/SetCondition.md) called twice on them as the entity with `AttackUp` and `DefenseUp` for 999999 actor turns

Finally, a frame is yielded.

#### Last pre `halfload` initialisations
This part only contains miscellaneous initialisations logic before `halfload` is set to true:

- MainManager.[RefreshEntities](Actors%20states/RefreshEntities.md) is called
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
- `avaliableplayers` is set to the return of MainManager.[GetFreePlayerAmmount](Actors%20states/GetFreePlayerAmmount.md)
- A frame is yielded
- The `cursor` position is set to (0.0, 999.0, 0.0)
- The SpriteRenderer material of the `cursor` has its renderQueue set to 10000
- [UpdateText](Visual%20rendering/UpdateText.md) is called
- MainManager.[RefreshSkills](RefreshSkills.md) is called
- RefreshAllData is called which sets `alldata` to a new list which consists of all the `playerdata` followed by all the `enemydata` appended together
- If the `fronticon` doesn't exist, it is initialised to the SpriteRenderer of a new UI object with the name `attackicon` childed to the `GUICamera` with position of Vector.zero, size of Vector3.one / 3.0, sortingOrder of 20 and with a sprite of `guisprites[63]` (the red arrow of the front icon in battle)
- A frame is yielded
- We exit the `pause` entered earlier
- If there's more than one `playerdata` element, then as long as the first `partypointer` doesn't match the first instance.`partyorder` (meaning the `battle` doesn't point to the leading party member), a [SwitchParty](Battle%20flow/Action%20coroutines/SwitchParty.md) action coroutine is started with fast (which ends up calling MainManager.SwitchParty immediately) followed by a frame yield until they both match NOTE: it does mean the coroutine calls can be spammed, but in theory, the while condition gets updated information everytime since the StartCoroutine call immediately causes the party to be updated
- If calledfrom exists with a `dizzytime` above 0.0:
    - The `playerdata` with a `trueid` (which is the [animid](../Enums%20and%20IDs/AnimIDs.md)) that matches the first `partypointer` is found and its `cantmove` is set to -1 and `receivedrelay` of the corresponding `playerdata` index is set to true TODO what does the received relay part means ???. This essentially gives an additional turn to the lead party member
    - calledfrom.`dizzytime` is set to 0.0
- If the `StrongStart` [medal](../Enums%20and%20IDs/Medal.md) is equipped on a party member, the corresponding `playerdata` gets its `cantmove` decremented which gives it an additional turn
- UpdateConditionIcons is called which calls UpdateConditionBubbles on all battleentity (all `playerdata` with right to false and all `enemydata` with `hp` above 0 with right to true)
- All frames are yielded while `action` is true until it goes to false (this is done to ensure all remainings SwitchParty calls done earlier are fully finished)
- [RefreshEXP](Visual%20rendering/RefreshEXP.md) is called
- SetLastTurns is called which resets `lastturns` to a new aray with the length being the amount of free players - 1 and all elements being -1 (this resets the player selection cycle)
- `halfload` is set to true
- All `playerdata` battleentity have their `rigid` gravity enabled, `onground` set to true and [animstate](../Entities/EntityControl/Animations/animstate.md) set to 13 (`BattleIdle`)
- 0.1 seconds are yielded

### Last adjustements
This section occurs after `halfload` is set to true and the coroutine yielded making the new value visible to the rest of the game.

#### Last player and enemy setup
This part only contains miscellaneous initialisations logic mainly about the `plyaerdata` and `enemydata`: 

- All `enemydata` battleentity have their `rigid` gravity enabled
- The `dimmer` gets childed to the `battlemap` with a clear color and then disabled
- If instance.`firstbattleaction` is false TODO ???
    - A [DoAction](Battle%20flow/Action%20coroutines/DoAction.md) call starts with the first `playerdata` battleentity with action -555
    - All frames are yielded while `action` is true until it goes to false
    - 0.5 seconds are yielded
    - instance.`firstbattleaction` gets set to true
- A frame is yielded

#### Battle out transition
This part only plays the battle out transition by calling PlayTransition with id 3 (the battle out transition), the data being the map.`battleleaftype` at 0.025 speed and the color being the same one used for the previous transition played earlier.

0.2 seconds are yielded after.

#### `switchicon` initialisation
This part initialises the `switchicon` if it didn't exist as a new GameObject with name `switchicon` childed to the `GUICamera` with a local position of (0.0, 99.0, 0.0). 

It also causes 3 child to get added to it:

- A new UI object named `arrow0` with size Vector3.one * 0.4 using the sprite `guisprites[92]` (right turn black arrow) with a sortingOrder of 3 with angles (0.0, 0.0, 180.0). The local y/z positions is 0 and the x is -1.0 except if `longcancel` is true where it becomes -1.25
- Another instance of the above, but the name is `arrow1`, there is no angles and the x position is the positive version instead of the negative one
- A new GameObject named `button` with a ButtonSprite component with SetUp called on it using input 5 (Cancel), -1 as onlytype, no description, local position of Vector3.zero, iconsize of Vector3.on * 0.4, sortingOrder of 0 and the `switchicon` as the parent making it also the actual parent when its Start occurs

#### State reset
This resets some important state fields:

- `currentturns` is set to -1 (no player selected)
- `turns` is set to 0
- `cancelupdate` is set to false
- `enemy` is set to false
- instance.`inbattle` is set to true
- if `sadv` is not negative, adv is set to it
- `sadv` is set to adv (which only does anything if it had a non negative value)

#### Enemy first strike
This part only happens if adv is 3:

- `enemy` is set to true
- `firststrike` is set to true
- [UpdateAnim](Visual%20rendering/UpdateAnim.md) is called
- instance.`hudcooldown` is set to 10.0 which shows the main HP / TP HUD
- instance.`showmoney` is set to -1.0 which hides the berry count HUD
- `action` is set to true
- [RefreshEnemyHP](Visual%20rendering/RefreshEnemyHP.md) is called
- 0.15 seconds are yielded
- A [DoAction](Battle%20flow/Action%20coroutines/DoAction.md) call is started on `enemydata[0]` with action 0
- All frames are yielded while `action` is true until it goes to false
- `enemydata[0].tired` is set to 0
- `enemydata[0].cantmove` is set to 0
- `action` is set to true
- A [CheckDead](Battle%20flow/Action%20coroutines/CheckDead.md) is started with the coroutine being stored in `checkingdead`
- All frames are yielded while `checkingdead` is in progress
- All `enemydata` have their `tired` and `cantmove` set to 0
- `currentturns` is set to -1
- `action` is set to false
- `firststrike` is set to false
- `enemy` is set to false
- SetLastTurns is called which resets `lastturns` to a new array with the length being the amount of free players - 1 and all elements being -1

#### Conditions special logic
This part manages the player and enemies conditions as they have special logic to them that must occur at the very end.

If `saveddata` is true (this is a retry), SetStartConditions is called which will do the following:

- Add all the corresponding `sdata.psc` to each `playerdata`'s `condition` from the existing ones
- Add all the corresponding `sdata.esc` to each `enemydata`'s `condition` from the existing ones
- Add all the corresponding `sdata.enemyweakness` to each `enemydata`'s `weakness` from the existing ones
- SetLastTurns is called which resets `lastturns` to a new aray with the length being the amount of free players - 1 and all elements being -1

After, all `playerdata`'s `condition`'s element that have a turn count of 0 or below are removed

From there, the `RandomStart` [medal](../Enums%20and%20IDs/Medal.md) takes effect if it's equipped for each `playerdata` that has it. The effec involves a GetChance test that is performed with 16 weights with outcomes from 0 to 6 and what happens here depends on the result:

- 0 (3/16): [StatusEffect](Actors%20states/Conditions%20methods/StatusEffect.md) is called on the `playerdata` with `AttackUp` for 2 actor turns with effect
- 1 (3/16): [StatusEffect](Actors%20states/Conditions%20methods/StatusEffect.md) is called on the `playerdata` with `DefenseUp` for 2 actor turns with effect
- 2 (2/16): `StatUp` sound is played if it wasn't already followed by a [StatEffect](Visual%20rendering/StatEffect.md) coroutine being started with the battleentity using type 4 (green up arrow) followed by the `playerdata`'s `charge` being incremented
- 3 (2/16): If the `playerdata`'s `poisonres` is 100 or above (it's immune to poison), this is treated as an outcome of 2 described above. If it's not, the `Poison` sound is played if it wasn't already followed by the `PoisonEffect` particles being played on the battleentity position followed by [SetCondition](Actors%20states/Conditions%20methods/SetCondition.md) being called with `Poison` on the `playerdata` for 2 actor turns unless the `Eternal Venom` [medal](../Enums%20and%20IDs/Medal.md) is equipped on it where it's 99999 actor turns instead
- 4 (2/16): The `StatUp` sound is played if it wasn't already followed by a [StatEffect](Visual%20rendering/StatEffect.md) coroutine being started with the battleentity using type 5 followed by the `playerdata`'s `cantmove` being decremented
- 5 (2/16): The `Heal3` sound is played if it wasn't already followed by the `MagicUp` particles being played on the battleentity position without sound followed by [SetCondition](Actors%20states/Conditions%20methods/SetCondition.md) being called with `GradualHP` on the `playerdata` for 2 actor turns
- 6 (2/16): The `Heal3` sound is played if it wasn't already followed by the `MagicUp` particles being played on the battleentity position without sound followed by [SetCondition](Actors%20states/Conditions%20methods/SetCondition.md) being called with `GradualTP` on the `playerdata` for 2 actor turns

Finally, UpdateConditionIcons is called which calls UpdateConditionBubbles on the battleentity (all `playerdata` with right to false and all `enemydata` with `hp` above 0 with right to true)

#### Final setup
These are the final steps of StartBattle:

- `turns` is set to 0
- MainManager.SortLights is called
- If MainManagr.`music[0]` is played and its clip exists, `sdata.music` is set to the clip's name
- `saveddata` is set to true
