# Bug Fables entity format

An [entity](../Entities/Entity.md) in Bug Fables is a game object that is dynamically loaded into the game and has specific behaviors. Entities can be considered as the building blocks of the game. Some entities's data are predefined in files inside the `Resources/data/entitydata` and they belong to a specific map. When that happens, the entity also will get an [NPCControl](../Entities/NPCControl/NPCControl.md) as that is how that component is defined: any [EntityControl](../Entities/EntityControl/EntityControl.md) predefined in a map. There are many kinds of entities in the game, but they always use the same system format no matter what type it is making it a very global and monolithic system. The logic in the system is defined in the `EntityControl` and `NPCControl` components.

## Entity Data File

Each line of an entity map file contains one line per entity. It contains fields about its `EntityControl` and its `NPCControl` separated by `}`. Each file corresponds to each map in the game where its filename is the [Map](../Enums%20and%20IDs/Maps.md)'s name.

The names of each entity are not contained in the data files. They are contained in a `names` folder next to the data files. An entity name file contains one string per line that corresponds to the name of entity of the matching line in the matching data file.

This result in the following important folders:
- Path to the entity data files from the assets: `Resources/data/entitydata`
- Path to the entity name files from the assets: `Resources/data/entitydata/names`

The name of an entity can contain special strings to act as [modifiers](../Entities/EntityControl/Modifiers.md) on it.

## Entity Data File Fields

The following contains all the fields that the game loads for each entities upon loading the map.

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
|190|npcdata.regionalflag|int|The [Regional flag](../Flags%20arrays/Regionalflags.md) slot bound to this entity|
|191|initialheight|float||
|192|bobrange|float||
|193|bobspeed|float||
|194|npcdata.activationflag|int|A [Flag](../Flags%20arrays/flags.md) slot used when this entity is activated|
|195|npcdata.returntoheight|bool or int|Int if the length in string is 1:|
