# Bug Fables entity format

An entity in Bug Fables is a game object that is dynamically loaded into the game and has specific behaviors. Most entities are specified in files called entity data files which belongs to a specific map. There are many kinds of entities in the game, but they always use the same system format no matter what type it is making it a very global and monolithic system. The logic in the system is defined in the `EntityControl` and `NPCControl` components.

## Entity Data File

Each line of an entity map file contains one line per entity. It contains fields about its `EntityControl` and its `NPCControl` separated by `}`. Each file corresponds to each map in the game where its filename is the [Maps](../Enums%20and%20IDs/Maps.md)'s name.

The names of each entity are not contained in the data files. They are contained in a `names` folder next to the data files. An entity name file contains one string per line that corresponds to the name of entity of the matching line in the matching data file.

Path to the entity data files from the asset tree: `Resources/data/entitydata`
Path to the entity name files from the asset tree: `Resources/data/entitydata/names`

The name of an entity can contain special strings to act as modifiers on it.

An entity, when created, will have the game make the following hierarchy a the root of the main Scene:

* Entity (name in entity name file, has an `EntityControl` and a `CapsuleCollider`)
  * Rotater
    * Sprite (has a `SpriteRenderer`)
  * MoveRotater

The `SpriteRenderer` component of the Sprite GameObject is accessible via the `sprite` field.
The `MoveRotater` GameObject is accessible via the `movetotater` field.
The `CapsuleCollider` component of the entity GameObject is accessible via the `ccol` field.

## Entity Data File Fields

|Position|Description|Type|Additional information|
|--------|-----------|----|----------------------|
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
|190|npcdata.regionalflag|int|The [Regionalflag](../Flags%20arrays/Regionalflag.md) slot bound to this entity|
|191|initialheight|float||
|192|bobrange|float||
|193|bobspeed|float||
|194|npcdata.activationflag|int|A [Flag](../SetText/Commands/Individual%20commands/Flag.md) slot used when this entity is activated|
|195|npcdata.returntoheight|bool or int|Int if the length in string is 1:|

## Name Modifiers

It is possible to combine them by containing multiple of modifiers. Here are all the possible ones:

|Name|Description|
|----|-----------|
|Holo|Sets hologram to true on Start. This renders the entity as a hologram|
|ICE|Sets npcdata.extrafreeze to true on Start. This forces the effect of Extra Freeze to apply. It only works on NPCType.Enemy|
|TIME|Sets extratimer to true on Start. This gives twice as much time for the entity to move to its target before triggering the failsafe|
|COT|Cave Of Trails; On Start, sets hologram to true. This renders the entity as a hologram|
|Fixed|On Start, sets fixedentity to true. This freezes all constraints on the rigid body Invokes SetFixed. This forces the entity to be completly fixed in position, no gravity or physics influence and disables the ccol (can be undone by calling Unfix)|
|FxdCol|On Start, sets fixedentity to true. This freezes all constraints on the rigid body Invokes SetFixedCollider. This forces the entity to be completly fixed in position, no gravity or physics influence WITHOUT disabling the ccol (can be undone by calling Unfix)|
|ALW|Always; Sets alwaysactive to true. This forces the entity to be seen as in the camera range and will always be active on Update|
|PAU|Pause; Sets activeonpause to true. This allows the entity to be active on Update even when paused or in a message|
|HIDE|Sets hideinside to true. This causes the GameObject to be disabled when inside|
|ROT|Rotater; Sets lockrotater to true. This prevents the rotater from updating its angle. Also causes the angle to be set in 0.25 seconds on create|
|ShwEm|Show Emotions; Sets alwaysemotions to true. This allows to see the emotes??? (TODO) of the entity regardless of distance to player|
|COG|Does a Raycast downward and sets startpos to the hitinfo.point??? (TODO)|
|NGS|No Ground Start; sets onground to false on Start|
|ITHD|Interact Horn Dash; Calls SetHitInteract with NPCControl.HitInteract.HornDash|
|NEAR|Alows CheckNear to destroy GameObjects who are 30f away from the player's z position|
|WTA|Water; Adds water effects when Kabbu horns an upstanding bridge|
|TOG|Toggke; Adds hit toggling features (like the toggle switches in the Power Plant puzzle)|
|NGF|No Gravity Fix; GravityFix will not run on Start, only applicable for NPCType.Enemy|
|NDTCT|No Detector; The detector medal will not beep from this entity|
|DDIST|Detector Distance; The Detector medal will only beep from this entity if it is closer than 20.0 from the player's z position|

## NPC Types

The type will be the tag of the entity GameObject except for `SemiNPC` which 
will be `NPC`. Here are the different `NPCType`:

### NPC

It has the most unique behaviors out of every types. It generally acts as a full NPC
that can be talked to, have emotes and has a pusher. 
Shares some behaviors with `SemiNPC` and `Enemy`.

* Can shatter ice blocks if it has `StealthAI`
* Can have emotes??? (TODO)
* Can get locked if insides??? (TODO)
* It will have its radius increased by 0.35 if it is smaller than 1.75
* It will have a pusher GameObject parented to the sprite which has a `CapsuleCollider`
* ActiveSelf works in `MapControl.LateUpdate`
* Can get `fixedcol` if outside and a shopkeeper??? (TODO)

Shared with `SemiNPC`

* Can be interacted with the jump button
* Can have special emotes when close by??? (TODO)

Shared with `Enemy`

* Can get affected by `FixEntities`
* Will stop force move when inactive every 3 frames if force move?
* Can be Unfixed without force
* It will have its `scol` disabled

### SemiNPC

It is recognised as if it was the same as an `NPC` in Setup, but only has a very limited
set of behaviors shared with `NPC`. It has no exclusive behaviors so it's basically a limited
`NPC` type.

Shared with `NPC`

* Can be interacted with the jump button
* Can have special emotes when close by??? (TODO)

### Enemy

This type is specifically made for special enemies behavior such as the ability to go into battle.
It shares some behaviors with `NPC`.

* Will get `GravityFix` on Start
* The help arrow for Kabbu is cyan
* Has lost and found behaviors with the player
* Its mass is 10 times less than other types
* The entity has a `CapsuleCollider` called `secondcoll` that is trigger
* Has custom freeze behavior on `Update` and `OnTriggerEnter`
* Has custom behaviors on Update that manages battling and being dizzy
* Has custom respawn behavior on `LateUpdate`
* The `digpart` has a different `RenderQueue`? (TODO)
* Custom `Death` behavior
* Custom hazard stuff??? (TODO)

Shared with `NPC`

* Can get affected by `FixEntities`
* Will stop force move when inactive every 3 frames if force move? (TODO)
* Can be Unfixed without force
* It will have its scol disabled

### Object

Almost has no specific behaviors. It doesn't share any and only has its own logic.
Most of the details of the logic are defined in the `ObjectTypes`.

* Has `OnTriggerExit` and `OnTriggerStay` behaviors
* It will have its `ccoll` disabled on `Start`

## Ojbject Types

Here are the different `ObjectTypes`:

### None:

No object behavior

### BeetleGrass:

A grass that is cutable by Kabbu's horn.

* The rotater is tagged "Hornable"
* The layer is 8
* The rigid body is frozen
* NearSomething considers it to be near an NPC
* OnTriggerEnter with a collider of "BeetleHorn" or "BeetleDash", the grass will be cut

Cutting the bush will drop the crystal berry if not obtained already.
Otherwise, if there are items in the vectordata, it pick a random element from this array and if that element has a valid item id, it will be dropped
Cutting the bush can activate a flag or a regional speciifed in activationflag and regionalflag respectively. If it drops an item, these flags are sent to the item
Assuming the grass doesn't have a crystal berry, it also has a 13% chance to drop a berry

data\[0\]: The grass sprite to use
data\[1\]: A crystal berry flag that the bush contains
vectordata: an array of items that can drop (x component is the id). The array can be arranged to created a weighted distribution in the odds
The detector will beep if the crystal berry in data is not obtained yet

### PushRock

TODO A cube shaped rock that can be moved using Kabbu

* NearSomething considers it to be near an NPC
* The GameObject tag is PushRock
* The Rotater tag is Hornable
* The layer is 13
* The scol is disabled
* The boxcol is disabled
  data\[0\]: 
  data\[1\]: If 1, the rigid is not frozen
  data\[2\]: \<0: 
  0: PushRockStuff???
  1: Freeze rotation and position Z
  2: Freeze rotation and position X
  3: Freeze and if data\[3\] is NOT 1, also freeze position Y
  data\[3\]: !- 1 if the position Y is frozen when data\[2\] is 3

### PressurePlate

A plate that can be pressed on
data\[0\]: 1 if the player can press the plate by standing on it
data\[1\]: 1 if it requires a PushRock, icecube or hornable being placed to press the plate
data\[2\]: An event id started when the plate is pressed, it is a one shot, it comes back to -1 after start
vectordata\[0\]: Position when pressed?
activationflag: Flag used to tell if the plate is pressed

### ANDGate

TODO
data\[0\]: event id?
data\[1\]: ???
data\[2-19\]: ???
activationflag: ???

### CameraChange

A zone that causes the camera to change angle
data\[0\]: if 1, camoffset is vectordata\[0\]

data\[1\]: if 1, camlimitpos is vectordata\[1\] and 
camlimitneg is vectordata\[2\]

data\[2\]: if 1, changecamspeed is true and camspeed is changed to vectordata\[3\].x

data\[3\]: if 1, camangleoffset is vectordata\[4\]

data\[4\]: if 1, camtarget is the transform of the entity at index data\[5\]

data\[5\]: index of the entity whose transform is camtarget (if data\[4\] is 1)

data\[6\]: if 1, camtargetpos is vectordata\[5\]

data\[7\]: if 1, camanglechange is true and camanglespeed is changed to vectordata\[3\].x

vectordata\[0\]: camoffset (if data\[0\] is 1)

vectordata\[1\]: camlimitpos (if data\[1\] is 1)

vectordata\[2\]: camlimitneg (if data\[1\] is 1)

vectordata\[3\]: X is 
camspeed (if data\[2\] is 1) or/and camanglespeed (if data\[7\] is 1)

vectordata\[4\]: camangleoffset (if data\[4\] is 1)

vectordata\[5\]: camtargetpos (if data\[6\] is 1)

### Item

A pickable item

* The ccol is disabled
* If the Beemerang touches the item and it was fixed, Unfix is called
* The item is picked up on OnTriggerEnter where other is the player and the game isn't paused
* The bounciness of the ccol material is 0.75
  data\[0\]: Item type, crystal berry index if anim id is 3
  data\[1\]: Event id to start after picking up the item
  data\[3\]: Crystal berry index
  activationflag: The flag index to bind to the item
  regionalflag: The regional index to bind to the item
  The detector will beep on a cb that is not obtained yet or activationflag isn't on yet?

### DoorOtherMap

A loading zone to another map

* insideid is -2
  data\[0\]: Target map id
  vectordata\[0\]: Position to move to for the transition
  vectordata\[1\]: Position to spawn at after the load
  vectordata\[2\]: Position to move to after the load from the spawn
  activationflag: The flag index to turn on when crossing the zone
  regionalflag: The regional index to turn on when crossing the zone

### SetPlayerRespawn

A zone that sets the lastpos once inside

* boxcol is a GameObject named Respawner with a trigger BoxCollider and a tag of Respawn
  vectordata\[0\]: Position to set lastpos

### DoorSameMap

A bidirectional zone to move inside and outside in the same map

* insideid is -2
  data\[0\]: Inside id to go to

data\[1\]: Music id to change to
vectordata\[0\]: Position to move to during the transition going inside

vectordata\[1\]: Position to move to during the transition going outside

vectordata\[2\]: Camoffset to change to during the transition going inside

vectordata\[3\]: camangleoffset to change to during the transition going inside

vectordata\[4\]: Camoffset to change to during the transition going outside

vectordata\[5\]: camangleoffset to change to during the transition going outside

activationflag: The flag index to turn on when crossing the zone

regionalflag: The regional index to turn on when crossing the zone

### Beemerang

The Beemerang when thrown

* Tag is BeeRang on Update
  vectordata\[0\]: Hold position

### EventTrigger

A zone that triggers an event

data\[0\]: Event id to trigger

data\[1\]: 1 if the event can be triggered again after the event

data\[2\]: 1 if the trigger is enabled

activationflag: The flag index to turn on when triggering the event

regionalflag: The regional index to turn on when triggering the event

### DialogueTrigger

A zone that triggers a dialogue

data\[0\]: Dialogue id to trigger

data\[1\]: 1 if the dialogue can be triggered again after the dialogue

data\[2\]: 1 if the trigger is enabled
activationflag: The flag index to turn on when triggering the dialogue
regionalflag: The regional index to turn on when triggering the dialogue

### ANDBlock

TODO

data\[0\]: event id?
data\[1\]: ???
data\[2-19\]: ???
activationflag: ???
vectordata\[0\]: ???
vectordata\[1\]: ???
vectordata\[2\]: scale thing???

### SavePoint

A save crystal
data\[0\]: 1 if the save crystal has a sphere of light around it
data\[1\]: 10 or higher if red
data\[2\]: 0 if the save crystal will heal, 1 otherwise (will be yellow if data\[1\] \< 10)
vectordata\[0\]: Position of Dead Lander Omega to look at if data\[1\] >- 10

### JumpSpring

A spring to either jump in the air or to go to another position
data\[0\]: 1 if the spring moves to another position, jumps in the air otherwise
data\[2\]: Entity id of a DoorSameMap to transition between inside and outside
vectordata\[0\]: Height of the jump
vectordata\[1\]: Position to go to
vectordata\[2\]: X is the height of the jump when going to another position

### DigSpot

A spot that Kabbu can dig out an item from
data\[0\]: 0: The dig spot holds an item
1: The dig spot holds a crystal berry
2: The dig spot triggers an event
data\[1\]: Item type if data\[0\], Crystal berry id if data\[0\] is 1 or event id if data\[0\] is 2
data\[2\]: Item id if data\[0\] is 0
activationflag: The flag index to turn on when picking the item
regionalflag: The regional index to turn on when picking the item

### Switch

TODO

data\[0\]: ???
data\[1\]: Event id to trigger
data\[2\]: actioncooldown
data\[3\]: 
data\[4\]: 1 if the rotater is hornable
activationflag: The flag index to turn on when the switch is activated
regionalflag: The regional index to turn on when the switch is activated

### MusicChange

UNUSED

### CoiledObject

A coiled vine that traps an item until hit
data\[0\]: Index of the trapped entity or mapid???
data\[1\]: Flags index to turn on when hit
vectordata\[0\]: position of the coil?
regionalflag: The regional index to turn on when the coil is hit

### FixedAnim

Locks the entity into an animation?
data\[0\]: if 1, gravity and ccol are disabled, the ccol is enabled otherwise
data\[1\]: The animstate to lock into?

### DigWall

UNUSED

### ItemSpawner

UNUSED

### EnemySpawner

An entity that can respawn an enemy entity when killed
data\[0\]: Entity index of the enemy to respawn
data\[1\]: if 1, actioncooldown is 0 on SetUp, it is data\[4\] otherwise
data\[4\]: actioncooldown to use
data\[5\]: NOT STORED; assigned to the animid on SetUp
vectordata\[0\]: Base position to respawn the enemy
vectordata\[1\]: Base offset to apply to the position (actual offset is random between -x,-y,-z and x,y,z) 

### Dropplet

A water droplet that can be frozen to an ice cube, only one droplet can exist at all time
data\[0\]: The time in frame between drops
data\[1\]: Distance threshold of vectordata\[0\] before shattering the ice???
data\[2\]: ??? if 1, actioncooldown is set to data\[3\] if higher than 0 and the position of the droplet is set to underneath the map in 0.1 seconds instead of instantly
data\[3\]: actioncooldown if data\[2\] is 1; this cooldown shatters the ice when expired
vectordata\[0\]: Offset of startpos???
vectordata\[1\]: Hornable's pushamount formed by the x and y component

### PathPlatform

TODO

A platform that goes along a path?
dialogues\[0\]: x is speed???
y is acceleration???
dialogues\[1\]: x is ???
y is actioncooldown???
dialogues\[2\]: x is the amount to scale the platform * 10
y is the electime for electrical platform
vectordata: the nodes???

### BreakableRock

A rock that can be broken with Kabbu

* NearSomething considers it to be near an NPC
  data\[0\]: The color of the renderer material:
  0: white
  1: light orange
  2: light yellow
  3: light blue
  4: light green
  5: pink
  activationflag: The flag index to turn on when the rock is broken
  regionalflag: The regional index to turn on when the rock is broken

### RotatingPlatform

TODO

A platofrm that can rotate???
dialogues\[0\]: x is speed???
y is acceleration???
dialogues\[1\]: x is ???
y is actioncooldown???
dialogues\[2\]: x is the amount to scale the platform * 10
y is the electime for electrical platform
vectordata\[0\]: Initial angle
vectordata: the nodes???

### Geizer

TODO

A geizer of water than can be frozen to a platform
data\[0\]: ???
data\[1\]: entity index???
data\[3\]: if 1, ???
data\[4\]: if not 0, the geizer cannot be frozen with an IceRadius
vectordata\[0\]: z is the cooldown before thawing (times 3 if extra freeze is equipped)

### MusicRange

A zone to play a song at
data\[0\]: NOT STORED; assigned to data\[1\], then decrement each Update, when 0, performs a distance check
data\[1\]: Amount of frames to wait between distance checks
data\[2\]: Music id to play
data\[3\]: Disabled if -1???
vectordata\[0\]: x is the distance threshold
y is lerp thingy???
z is volume scaler???
vectordata\[1\]: NOT STORED; x is assigned to 0, y to the end loop point and z to the start loop point

### TempPlatform

A platform that can only be stood on temporarilly (such as a flytrap)

* Tagged as Platform
  data\[0\]: Time before the platform timer expires
  data\[1\]: if 1, ???
  data\[2\]: if 1, ???
  data\[3\]: if 1, the entity will shake when starting a collision with the player

### ScrewSwitch

A crank that can be spun with the Beemerang
vectordata\[0\]: x is the rate to increase the crank timer when cranking
y is the rate to decrease the crank timer when no longer cranking
z is the max timer cooldown when cranked to the max
vectordata\[1\]: angle stuff?
activationflag: The flag index to turn on when the crank has been activated once

### ResetCamera

Will reset the camera when the player enter the zone

### StencilSwitch

TODO

A switch that changes the ice radius around it (stencil shader stuff???)
data\[1\]: Entity index of the parent???
data\[2\]: if 1, starts activated
data\[3\]: if 1, disable all colliders of the entity???
vectordata\[0\]: x is ???
y is ???
z is ??? 
vectordata\[1\]: Parent offset position???
activationflag: The flag index to turn on when the switch is activated
regionalflag: The regional index to turn on when the switch is activated

### RollingRock

TODO

* If the player collider with it without diggint, event 116 is started
* The rock breaks if it touches an object with the tag RockLimit
* When rolling, it will break any BreakableRock that it touches
  A rock that rolls and crushes the player on touch
  data\[0\]: if 1, ???
  data\[2\]: 1 if the rock is shot from a canon
  data\[3\]: Entity index???
  vectordata\[0\]: Direction to roll???
  vectordata\[1\]: size stuff???
  vectordata\[2\]: Rotation angles?

### TriggerSwitch

TODO

A switch that ???
data\[0\]: Entity index???
data\[1\]: Hit thingy???
data\[2\]: Destroy the Beemerang when activated
activationflag: The flag index to turn on when the switch is activated
regionalflag: The regional index to turn on when the switch is activated

### WindPusher

A zone where winds flows and flying in it will increase horizontal speed
data\[0\]: Entity index of the entity that must be hit to have the zone in effect
vectordata\[0\]: Other end of the zone along with the current position
vectordata\[1\]: x is speed scaler of the wind when vectordata\[2\].y is absent
y is size of the boxcol in x
z is size of the boxcol in z
vectordata\[2\]: x is the startLifeTime (if not present, it is distance / 5)
y is the speed of the wind

### WaterSwitch

A switch that changes the water level
data\[0\]: Index of the main mesh's child to use as the water
vectordata\[0\]: x is ???
y is water level if switch is activated
z is water level if switch is deactivated
activationflag: The flag index to turn on when the switch is activated

### BattleMapChange

A zone that causes a change of the battlemap when entering it
data\[0\]: Battle map id to change to

## Action Behaviors

Here are the different `ActionBehaviors`:

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

## Interactions

Here are the different `Interaction`:

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

## Death Types

The death type govern what the entity will do on death. Unless otherwise specified, the entity gets destroyed at the end.
Here are the different `DeathType`:

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
