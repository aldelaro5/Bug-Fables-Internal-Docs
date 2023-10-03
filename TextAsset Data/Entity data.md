# Entity data

Entity data are split into 2 TextAssets in the game loaded on boot:

* `Resources/data/EntityValues`
* The `Resources/data/entitydata` directory

## `EntityValues` data

The TextAsset `Resources/data/EntityValues` from the root of the asset tree contains data that apply to every entities of a given [AnimID](../Enums%20and%20IDs/AnimIDs.md). Each line id of the data corresponds to the matching [AnimID](../Enums%20and%20IDs/AnimIDs.md). The data is loaded on boot during LoadEssentials in the `endata` static field of MainManager. The field is an array of struct of type `Entity_Data`.

NOTE: This asset uses the enum form of the [AnimIDs](../Enums%20and%20IDs/AnimIDs.md), NOT the int form which means the first line represents `None`.

### A word on the asset's format

This format is very unique in the game: it is not encoded in a similar fashion than other TextAssets in the game. Here is how the data is serialiazed:

* The fields are split by `,`
* Every primitives types are encoded as normal (int, float, string, boolean, etc...)
* A `Vector3` is NOT encoded using 3 fields. It is encoded using 1 field with the format `(x, y, z)` where `x`, `y` and `z` are the floats of the vector. Note that the inner separators are expected: the parser will expect the parenthesizes delimiters to know this is parsed differently.
* An empty string is an empty field.

### TextAsset and `Entity_Data` layout

Here is the layout of the data and the fields associated with this struct (NOTE: the position is given according to the TextAsset which means `Vector3` are treated as one position):

|Position|Description|Type|Additional information|
|--------:|-----------|----|----------------------|
|0|shadowsize|float|The default value is 1.0 if the [AnimIDs](../Enums%20and%20IDs/AnimIDs.md) is higher than the max line id contained in the asset|
|1|startscale|Vector3|The default value is Vector3.one if the [AnimIDs](../Enums%20and%20IDs/AnimIDs.md) is higher than the max line id contained in the asset|
|2|bleeppitch|float|The default value is 1.0 if the [AnimIDs](../Enums%20and%20IDs/AnimIDs.md) is higher than the max line id contained in the asset|
|3|bleepid|int||
|4|ismodel|bool||
|5|modelscale|Vector3||
|6|modeloffset|Vector3||
|7|freezesize|Vector3||
|8|freezeoffset|Vector3||
|9|freezeflipoffset|Vector3||
|10|preloaddata|string\[\]|The strings are split by `?`, can be left empty in which case, the array is left to `null`. The strings represents asset paths relative from the Asset directory. A `$` prefix means to not preload outside of battle while a `$` indicate it is a sprite instead of a game object from a prefab|
|11|shakeondrop|bool||
|12|diganim|bool||
|13|dontoverridejump|bool||
|14|freezenofall|bool||
|15|noshadow|bool||
|16|walktype|WalkType|Encoded in its int form, see below for the possible values|
|17|basestate|int|This field is loaded, but not used. A value must still be provided or an exception will be thrown.|
|18|basewalk|int|This field is loaded, but not used. A value must still be provided or an exception will be thrown.|
|19|minheight|float||
|20|startheight|float|This field is loaded, but not used. A value must still be provided or an exception will be thrown.|
|21|startbobspd|float||
|22|startbobfreq|float||
|23|hasiceanim|bool||
|24|noflyanim|bool|This field is optional, it can be omitted which leaves a default value of false|
|25|forceshadow|bool|This field is optional, it can be omitted which leaves a default value of false|
|26|Object|bool|This field is optional, it can be omitted which leaves a default value of false|

NOTE: Fields 24, 25 and 26 must be omitted TOGETHER if any are to be omitted. Failure to do so will result in an exception to be thrown when parsing the asset.

### WalkType values

Here are the possible values for a WalkType enum value:

|Value|Name|
|-----:|----|
|0|Normal|
|1|Jump|

## `entitydata` directory

The `entitydata` directory contains the names and details that applies to each specific entities that should be loaded upon a specific [Map](../Enums%20and%20IDs/Maps.md) load. The data only gets loaded during MapControl's `CreateEntities` for the concerned [Maps](../Enums%20and%20IDs/Maps.md) which happens on the MapControl's Start. The data ends up in the `entities` field of the MapControl which is an array of EntityControl.

Each line of an entity map asset contains one line per entity. It contains fields about its `EntityControl` and its `NPCControl` (via the `npcdata` field) separated by `}`. Each asset corresponds to each map in the game where its asset's filename is the [Map](../Enums%20and%20IDs/Maps.md)'s id.

The names of each entity are not contained in the data assets. They are contained in a `names` directory next to the data assets. An entity name asset contains one string per line that corresponds to the name of entity of the matching line in the matching data asset. The names asset filename is `Xnames` where X is the [Map](../Enums%20and%20IDs/Maps.md) id.

This gives the following layout:

* Path to the entity data files: `Resources/data/entitydata`
* Path to the entity name files: `Resources/data/entitydata/names`

The name of an entity can contain special strings to act as [modifiers](../Entities/EntityControl/Modifiers.md) on it which means the name is both used for debugging purposes as well as to modify its behavior.

### Entity Data File Fields

|Position|Description|Type|Additional information|
|--------:|-----------|----|----------------------|
|0|npcdata.entitytype|NPCControl.NPCType||
|1|npcdata.objecttype|NPCControl.ObjectTypes||
|2|npcdata.behaviors\[0\]|NPCControl.ActionBehaviors||
|3|npcdata.behaviors\[1\]|NPCControl.ActionBehaviors||
|4|npcdata.interacttype|NPCControl.Interaction||
|5|destroytype|NPCControl.DeathType||
|6|startpos.x|float||
|7|startpos.y|float||
|8|startpos.z|float||
|9|animid|int||
|10|flip|bool||
|11|ccol.height|float||
|12|ccol.radius|float|Also assigned to npcdata.colliderheight|
|13|npcdata.radius|float||
|14|npcdata.timer|float||
|15|speed|float||
|16|npcdata.actionfrequency\[0\]|float||
|17|npcdata.actionfrequency\[1\]|float||
|18|npcdata.speedmultiplier|float||
|19|npcdata.radiuslimit|float||
|20|npcdata.wanderradius|float||
|21|npcdata.teleportradius|float||
|22|NPCControl has a BoxCollider?|bool|Only applies for NPCType.Object|
|23|npcdata.boxcol.isTrigger|bool|Requires field at position 22 to be true|
|24|npcdata.boxcol.size.x|float|Requires field at position 22 to be true|
|25|npcdata.boxcol.size.y|float|Requires field at position 22 to be true|
|26|npcdata.boxcol.size.z|float|Requires field at position 22 to be true|
|27|npcdata.boxcol.center.x|float|Requires field at position 22 to be true|
|28|npcdata.boxcol.center.y|float|Requires field at position 22 to be true|
|29|npcdata.boxcol.center.z|float|Requires field at position 22 to be true|
|30|npcdata.freezetime|float||
|31|freezesize.x|float||
|32|freezesize.y|float||
|33|freezesize.z|float||
|34|freezeoffset.x|float||
|35|freezeoffset.y|float||
|36|freezeoffset.z|float||
|37|npcdata.eventid|int||
|38|Length of npcdata.requires|int|Maximum of 10|
|39-48|npcdata.requires|int\[\]|lists of [flags](../Flags%20arrays/flags.md) slots that needs to be true for the entity to be enabled|
|49|Length of npcdata.limit|int|Maximum of 10|
|50-59|npcdata.limit|int\[\]|lists of [flags](../Flags%20arrays/flags.md) slots that needs to be false for the entity to be enabled|
|60|Length of npcdata.data|int|Maximum of 10|
|61-70|npcdata.data|int\[\]|General purpose int array|
|71|Length of npcdata.vectordata|int|Maximum of 10|
|72-101|npcdata.vectordata|Vector3\[\]|General purpose triplets of floats array|
|102|Length of npcdata.dialogues|int|Maximum of 20|
|103-162|npcdata.dialogues|Vector3\[\]||
|163|transform.eulerAngles.x|float||
|164|transform.eulerAngles.y|float||
|165|transform.eulerAngles.z|float||
|166|Length of npcdata.battleids|int|Maximum of 4|
|167-170|npcdata.battleids|int\[\]|Lists of [Enemies](../Enums%20and%20IDs/Enemies.md) ids that forms a battle if encountered|
|171|npcdata.tagcolor.r|float||
|172|npcdata.tagcolor.g|float||
|173|npcdata.tagcolor.b|float||
|174|npcdata.tagcolor.a|float||
|175|emoticonoffset.x|float||
|176|emoticonoffset.y|float||
|177|emoticonoffset.z|float||
|178|npcdata.insideid|int||
|179-188|npcdata.emoticonflag|string\[10\]|Each element contains a comma separated tuple list where both elements are int and corresponds to each npcdata.emoticonflag.x and npcdata.emoticonflag.y|
|189|npcdata.tattleid|int||
|190|npcdata.regionalflag|int|The [Regionalflags](../Flags%20arrays/Regionalflags.md) slot bound to this entity|
|191|initialheight|float||
|192|bobrange|float||
|193|bobspeed|float||
|194|npcdata.activationflag|int|A [Flag](../SetText/Commands/Individual%20commands/Flag.md) slot used when this entity is activated|
|195|npcdata.returntoheight|bool or int|Int if the length in string is 1, bool otherwise|

### NPC Types

Here are the different `NPCType`:
TODO: move this to another page

* NPC 
* SemiNPC 
* Enemy
* Object 

### Object Types

Here are the different `ObjectTypes`:
TODO: move this to another page

Name | 
-------- | -------
None | 
BeetleGrass |
PushRock |
PressurePlate |
ANDGate |
CameraChange | 
Item |
DoorOtherMap |
SetPlayerRespawn |
DoorSameMap |
Beemerang |
EventTrigger |
DialogueTrigger |
ANDBlock |
SavePoint |
JumpSpring |
DigSpot |
Switch |
MusicChange |
CoiledObject |
FixedAnim |
DigWall |
ItemSpawner |
EnemySpawner |
Dropplet |
PathPlatform |
BreakableRock |
RotatingPlatform |
Geizer |
MusicRange |
TempPlatform |
ScrewSwitch |
ResetCamera |
StencilSwitch |
RollingRock |
TriggerSwitch |
WindPusher |
WaterSwitch |
BattleMapChange |

### Action Behaviors

Here are the different `ActionBehaviors`:
TODO: move this to another page

|Name|Description|
|----|-----------|
|None|Nothing, only does a `StopForceMove`.|
|FacePlayer|Faces towards the player. Will also face towards the player when interacting|
|ChasePlayer|Basic chase player behavior; the entity will move towards the player with an emote. If paused or inevent, `StopForceMove` is called|
|FleeFromPlayer|Move away from the player if not tattling or paused?|
|TurnRandomly|Flip and set the `actioncooldown` to a random amount of frames between half of the frequency and double the frequency. The first `actioncooldown` is `actionfrequency[0]`|
|Wander|Basic wandering behavior; The entity will attempt to move to a random position in the radiuslimit up to 50 times and `actioncooldown` is then set to a random range between the third of the frequency and the frequency itself|
|FaceAwayFromPlayer|Flip to the opposite direction of the x position of the player|
|TurnFixedInterval|Same as `TurnRandomly`, but the `actioncooldown` is always the frequency itself|
|Disguise|Basic diguise behavior; The entity will not appear as is and will instead appear as its disguise obj until the player gets in range. After that, the entity will go back to its startpos and put back the disguise after 120 frames (flips each 40 frames)|
|DisguiseOnce|Same as disguise, but the behavior becomes Wander after the player is inrange|
|FollowPlayer|UNUSED|
|WalkAwayFromPlayer|UNUSED, Same as FleeFromPlayer|
|FaceAhead|Face to the right|
|FaceBehind|Face to the left|
|FaceUp|Face forward (+z)|
|FaceDown|Face backward (-z)|
|SetPath|Is `alwaysactive` on setup; The entity will follow a path on loop made up by the vectors in vectordata|
|ChargeAtPlayer|The entity will move towards the player while attacking (similar to ChasePlayer, but with custom behavior)|
|ChargeAtPlayerFlipSprite|Same as ChargeAtPlayer, but the sprite will flip towards the player?|
|ShootProjectile|The entity will shoot an OverworldProjectile?|
|ShootProjectilePredict|UNUSED, Same as ShootProjectile|
|ChargeAndAttack|Same as `ChasePlayer`, but will also attack while chasing (diff with `ChargeAtPlayer`???) TODO|
|AlwaysWander|Same as Wander, but there is no cooldown between the wandering at the points|
|Unmoveable|The entity cannot be launched upwards when getting dizzy|
|ChargeAttackUnderground|Same as `ChargeAndAttack`, but the entity is digging and can `ChargeAndAttack` if it has been digging for 30 fames|
|WanderUnderground|Same as Wander, but the entity is digging|
|StealthAI|The entity will have a `StealthCheck` component, be alwaysactive and have extratimer on Setup. The entity remaining `alwaysactive`, will do the same as SetPath, but if inrange or if hit by the Beemerang and closer than sqrt(30) to the player, `StealthSpot` is called. The entity will shatter an ice cube if it touches one|
|SetPathJump|Same as `SetPath`, but jumps while following it|
|DisguiseOnceJumpForward|UNUSED, Same as Disguise|
|ChangeSpriteInRandius|Changes the animstate depending if in or out of the `wanderradius`; the values are in actionfrequency (0 is in, 1 is out)|
|ChaseWhenAnim|If the animstate is 23, same as ChasePlayer, otherwise, `behaviorcooldown` is set to 20, `StopForceMove` is called and animstate is set to the frequency|
|WalkWhenAnim|Same as `ChaseWhenAnim`, but the condition to be ChasePlayer is the animstate is 1 or it is 0 and the `behaviorcooldown` expired|
|WanderOffscreen|Same as Wander, but `activeonpause` and `alwaysactive` are set to true in `LateUpdate`|
|WanderNoWarp|UNUSED, Same as Wander, but without `DeathSmoke`? (TODO)|
|WanderOnWater|Same as Wander, but the Y is clamped the the water level of the map, and the startpos Y changes to current pos when on water|
|ChaseOnWater|Same as ChasePlayer, but the Y is clamped the the water level of the map, and the startpos Y changes to current pos when on water|

### Interactions

Here are the different `Interaction`:
TODO: move this to another page

|Name|Description|
|----|-----------|
|None|No interactions|
|Talk|Basic talking features? TODO|
|Check|Same as Talk, but with a different emote? TODO|
|SavePoint|Same as Talk, but the text is the save prompt|
|Event|Starts the event specified in eventid, same emote as Check? TODO|
|TalkReturnToOriginalFlip|Same as Talk??? TODO|
|Shop|Exempted in a lot of stuff??? TODO|
|ShopKeeper|Seems to handle the shop keepers interface? TODO|
|QuestBoard|Handles the quest board interaction for opening a quest board|
|StorageAnt|Same as talk, but the text is either the storage service prompt or the explanation of the services|
|CaravanBadge|Exempt from a lot of stuff? TODO|
|VenusHeal|Same as talk, but with a special tattle and the text is from Venus's dialogues|
|LockedDoor|Same as event, but the event is 59 and it has the same emote as Check? TODO|

### Death Types

Here are the different `DeathType`:
TODO: move this to another page

|Name|Description|
|----|-----------|
|None|`LockRigid` is called with true, does not get destroyed at the end|
|SpinSmoke|The entity will spin for 0.75 seconds before adjusting to the left for 0.3 seconds before disappearing with `DeathSmoke`|
|Smoke|UNUSED|
|Shrink|The entity will shrink to 0,0,0 for 80 frames before disappearing with `DeathSmoke`|
|PlayerDeath|The entity will spin as `Death2` SFX plays, followed by dropping dead on the floor as `Drop` SFX plays, this is the player death animation|
|SpinNoSmoke|Same as `SpinSmoke`, but without `DeathSmoke`|
|SpinKO|The entity will spin for 0.75 seconds, followed by dropping dead on the floor. The entity will not be destroyed at the end|
|KO|Same as `SpinKO`, but without the spin|
|SpinSmokeNoSprite|The sprites gets disabled and animstate reset to none. Despite its name, there is no smoke or spins|
|ShrinkNoSmoke|Same as `Shrink`, but without `DeathSmoke`|
|NinjaLog|`LockRigid` is called with true, then `DeathSmoke` is called and finally, `LeafDeathPoof` SFX plays|
|Sink|The entity will be moved down by 10 over a period of 90 frames, the GameObject is disabled after|
|ExplodeAnim|The `explosionsmall` particle plays followed by `Explosion` SFX playing followed by disabling the sprite followed by a screen shake|
|DropSprites|A rigid body is added to the sprite or model of the entity which will fall down/bounce on the floor for 1 second before being destroyed|
