# Object
Most of the logic depends entirely on the `objecttype` field.

There are a couple of special logic to mention that applies by default to all objects:

- In OnTriggerEnter, if the other collider isn't the player, `collisionammount` is incremented.
- This is the only type with logic in OnTriggerStay and OnTriggerExit for specific `objecttype`

## ObjectTypes
An `ObjectTypes` is an enum that determines the type of object an NPCControl with an `entitytype` of `Object` or `SemiNPC` has. This information is stored on the `objecttype` field.

A type dictates most of the logic of an `Object` in various situation notably Unity events, but also outside of NPCControl. Most prebaked logic in NPCControl is disabled by default, but any `ObjectTypes` can override this. It also greatly changes the meaning of several fields of NPCControl notably the general purpose `data` and `vectordata` array.

### ObjectTypes enum table
The following are all the possible `ObjectTypes` (NOT REFERENCED means the game never references the value and it has no logic, UNUSED means the logic is present, but it is either dead or not used under normal gameplay):

|Value|Name|Summary|
|----:|----|----------|
|0|None|No operations or this NPCControl isn't an `Object`|
|1|[BeetleGrass](ObjectTypes/BeetleGrass.md)|A grass that can be cut by Kabbu's Horn Slash. Can either be bound to a [crystalbfflags](../../Enums%20and%20IDs/crystalbfflags.md) slot which will be dropped, or a list of [items](../../Enums%20and%20IDs/Items.md) that can be dropped alongside the activation of a [flag](../../Flags%20arrays/flags.md) and [regionalflag](../../Flags%20arrays/Regionalflags.md) slots.|
|2|[PushRock](ObjectTypes/PushRock.md)|A rock or a pushable ice cube that can be moved using Kabbu's Horn Slash with configurable movement force. See the simplified explanation section for an intuitive description of how this object functions.|
|3|[PressurePlate](ObjectTypes/PressurePlate.md)|A plate that can be pushed down in its receptacle using a `PushRock` or optionally, the player or an NPCControl entity.`icecube`. This can optionally start an [event](../../Enums%20and%20IDs/Events.md) when actuated for the first time.|
|4|[ANDGate](ObjectTypes/ANDGate.md)|A multipurpose condition checker that can be operated in 4 distinct modes involving either [flag](../../Flags%20arrays/flags.md) or other map entities's `hit` value which results in either a new `hit` value or an [event](../../Enums%20and%20IDs/Events.md) being started.|
|5|[CameraChange](ObjectTypes/CameraChange.md)|A trigger zone to change the camera's properties.|
|6|[Item](ObjectTypes/Item.md)|A collectable [item](../../Enums%20and%20IDs/Items.md), [medal](../../Enums%20and%20IDs/Medal.md) or crystal berry bound to a [crystalbfflags](../../Enums%20and%20IDs/crystalbfflags.md) slot whose item id is the entity.`animid` field. This extends the logic of an [item entity](../EntityControl/Item%20entity.md).|
|7|[DoorOtherMap](ObjectTypes/DoorOtherMap.md)|A loading zone to another map with customisable movement before and after the load.|
|8|[SetPlayerRespawn](ObjectTypes/SetPlayerRespawn.md)|A trigger zone that sets the player.`lastpos` which is the player last spawn point. It can either be managed by this NPCControl or be a completely independant collider that PlayerControl can collider with.|
|9|[DoorSameMap](ObjectTypes/DoorSameMap.md)|A transition zone to and away from an inside of the current map.|
|10|[Beemerang](ObjectTypes/Beemerang.md)|Vi's Beemerang when thrown. This object type is only created dynamically.|
|11|[EventTrigger](ObjectTypes/EventTrigger.md)|An [event](../../Enums%20and%20IDs/Events.md) trigger zone. It can either unconditionally trigger as a one shot event on the first Update cycle or trigger via OnTriggerStay that starts the event when the player collides with it with the option to destroy the NPCControl after.|
|12|[DialogueTrigger](ObjectTypes/DialogueTrigger.md)|A zone where a [SetText](../../SetText/SetText.md) call starts in [dialogue mode](../../SetText/Dialogue%20mode.md) using a map dialogue line id. It can either unconditionally trigger as a one shot call on the first Update cycle or trigger via OnTriggerStay that starts the call when the player collides with it with the option to destroy the NPCControl after.|
|13|[ANDBlock](ObjectTypes/ANDBlock.md)|A multipurpose condition checker that can be operated in 3 distinct modes involving either [flags](../../Flags%20arrays/flags.md) or other NPCControl's `hit` value which results in a new `hit` value that can contol the entity.`sprite` local position.|
|14|[SavePoint](ObjectTypes/SavePoint.md)|A save crystal that can be hit using field abilities. It can be blue which only prompts to save the game, yellow which also heals the party and red which alerts a DeadLanderOmega, but doesn't prompt for saving and can optionally heal the party.|
|15|[JumpSpring](ObjectTypes/JumpSpring.md)|A spring that makes an entity jump up high via [Jump](../EntityControl/EntityControl%20Methods.md#jump) when touched or makes the entity jump straight to a specific point via [JumpTo](../EntityControl/EntityControl%20Methods.md#JumpTo) with the option to trigger an inside transition via a `DoorSameMap` present on the map without physically moving the player.|
|16|[DigSpot](ObjectTypes/DigSpot.md)|A spot where an [item](../../Enums%20and%20IDs/Items.md), [medal](../../Enums%20and%20IDs/Medal.md) or Crystal Berry can be dug out or have an [event](../../Enums%20and%20IDs/Events.md) be triggered when doing so using Kabbu's Dig ability.|
|17|[Switch](ObjectTypes/Switch.md)|A switch that can be turned on or off offering a wide variety of actuation flow such as starting an [event](../../Enums%20and%20IDs/Events.md), changing other `hit` values, offer a toggling feature and be only actuable using Kabbu's horn. Consult the switch actuation section for more details.|
|18|MusicChange|NOT REFERENCED|
|19|[CoiledObject](ObjectTypes/CoiledObject.md)|A coil that can trap another map entity within using a configurable local position and requires to be hit by a player ability to untrap the object. It can optionally set a [flag](../../Flags%20arrays/flags.md) slot to true when hit.|
|20|[FixedAnim](ObjectTypes/FixedAnim.md)|An object where the entity is left in a fixed [animstate](../EntityControl/Animations/animstate.md) with the option to also disables the entity.`ccol` and gravity on it.|
|21|DigWall|NOT REFERENCED|
|22|ItemSpawner|NOT REFERENCED|
|23|[EnemySpawner](ObjectTypes/EnemySpawner.md)|An enemy spawner attached to another [Enemy](Enemy.md) entity that will continuously respawn in a given circle range whenever it dies after a certain amount of time. The map entity id of the enemy, the time interval to wait before respawn and the range at which the enemy will spawn is configurable with the option to spawn the enemy immediately on the first Update cycle.|
|24|[Dropplet](ObjectTypes/Dropplet.md)|A water dropplet that can be frozen by Leif's Freeze which then turns it into an ice cube that can be moved by launching it using Kabbu's Horn Slash. The push velocity vector when launched is configurable and the object also supports many optional features, see the data array definition for details.|
|25|[PathPlatform](ObjectTypes/PathPlatform.md)|A platform that moves on a set path defined by multiple absolute position vectors called nodes and can conditionally be stopped depending on the `hit` value of other NPCControl present on the map. Can be operated in loop mode using only 2 nodes or with multiple nodes in path mode with more options.|
|26|[BreakableRock](ObjectTypes/BreakableRock.md)|A rock that can be broken using Kabbu's Horn Dash or shook by Kabbu's Horn Slash.|
|27|[RotatingPlatform](ObjectTypes/RotatingPlatform.md)|The same than `PathPlatform`, but path mode will have the nodes be euler angles vectors instead and the platform will rotate through these angles instead of moving. Loop mode is left unchanged, but is considered invalid.|
|28|[Geizer](ObjectTypes/Geizer.md)|A water or honey geizer that can be frozen using Leif's Freeze and it can elevate a `Dropplet` ice cube. The oscillation and time to stay frozen are configurable with the option to conditionally activate it when another NPCControl `hit` is true.|
|29|[MusicRange](ObjectTypes/MusicRange.md)|A spherical radius that when the player enters it, changes the current music playing. The music, amount of frames to wait between distance checks, radius, rate of volume change and max volume are configurable.|
|30|[TempPlatform](ObjectTypes/TempPlatform.md)|A platform that can only be stood on by the player for a limited amount of time before respawning the player. The amount of frames the player can stay on it and some animation timings are configurable.|
|31|[ScrewSwitch](ObjectTypes/ScrewSwitch.md)|A crank switch actuated by Vi's Beemerang Toss with an analog actuation that needs to be held for a certain period before being fully actuated. The angles of rotation when actuated, the rate of actuation / deactuation and the maximum value to reach before fully actuating the crank are all configurable.|
|32|[ResetCamera](ObjectTypes/ResetCamera.md)|A zone where the camera is reset to default via MainManager.ResetCamera when the player enters it.|
|33|[StencilSwitch](ObjectTypes/StencilSwitch.md)|A switch that, when activated, causes a frozen area range to appear or to disappear when deactivated.|
|34|[RollingRock](ObjectTypes/RollingRock.md)|A rolling rock that either appears and rolls on its own or is shot from a canon. The target direction, velocity, radius, rotation, spawn timing and minimum y position limit are all configurable with the option to only have the rock roll once it hits the ground and an option to only activate the rock when another NPCControl `hit` value is true.|
|35|[TriggerSwitch](ObjectTypes/TriggerSwitch.md)|A trigger zone that controls either its own `hit` value when the player enters or exits the zone or it controls the `hit` value of another NPCControl when the player enters the zone, but not when they exit it.|
|36|[WindPusher](ObjectTypes/WindPusher.md)|A zone that renders wind particles moving towards a point. If the player enters that zone, they will be pushed towards the same direction then the wind. The wind force, wind target position, wind sizes and the visual lifetime / speed are all congurable. There is also the option to only have this object in effect when another map entity `hit` value is true.|
|37|[WaterSwitch](ObjectTypes/WaterSwitch.md)|A switch with 2 states to control the vertical position of a child of the map's main mesh GameObject between 2 points (the lower one when not actuated and the upper one when actuated). The child index, the bounds of the y position to set and the amount of frames to spread the movement are configurable.|
|38|[BattleMapChange](ObjectTypes/BattleMapChange.md)|UNUSED, A zone that causes a change of the map.`battlemap`.|