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

|Value|Name|Description|
|----:|----|----------|
|0|None|No operations or this NPCControl isn't an `Object`|
|1|[BeetleGrass](ObjectTypes/BeetleGrass.md)| |
|2|[PushRock](ObjectTypes/PushRock.md)| |
|3|[PressurePlate](ObjectTypes/PressurePlate.md)| |
|4|[ANDGate](ObjectTypes/ANDGate.md)| |
|5|[CameraChange](ObjectTypes/CameraChange.md)| |
|6|[Item](ObjectTypes/Item.md)| |
|7|[DoorOtherMap](ObjectTypes/DoorOtherMap.md)| |
|8|[SetPlayerRespawn](ObjectTypes/SetPlayerRespawn.md)| |
|9|[DoorSameMap](ObjectTypes/DoorSameMap.md)| |
|10|[Beemerang](ObjectTypes/Beemerang.md)| |
|11|[EventTrigger](ObjectTypes/EventTrigger.md)| |
|12|[DialogueTrigger](ObjectTypes/DialogueTrigger.md)| |
|13|[ANDBlock](ObjectTypes/ANDBlock.md)| |
|14|[SavePoint](ObjectTypes/SavePoint.md)| |
|15|[JumpSpring](ObjectTypes/JumpSpring.md)| |
|16|[DigSpot](ObjectTypes/DigSpot.md)| |
|17|[Switch](ObjectTypes/Switch.md)| |
|18|MusicChange|NOT REFERENCED|
|19|[CoiledObject](ObjectTypes/CoiledObject.md)| |
|20|[FixedAnim](ObjectTypes/FixedAnim.md)| |
|21|DigWall|NOT REFERENCED|
|22|ItemSpawner|NOT REFERENCED|
|23|[EnemySpawner](ObjectTypes/EnemySpawner.md)| |
|24|[Dropplet](ObjectTypes/Dropplet.md)| |
|25|[PathPlatform](ObjectTypes/PathPlatform.md)| |
|26|[BreakableRock](ObjectTypes/BreakableRock.md)| |
|27|[RotatingPlatform](ObjectTypes/RotatingPlatform.md)| |
|28|[Geizer](ObjectTypes/Geizer.md)| |
|29|[MusicRange](ObjectTypes/MusicRange.md)| |
|30|[TempPlatform](ObjectTypes/TempPlatform.md)| |
|31|[ScrewSwitch](ObjectTypes/ScrewSwitch.md)| |
|32|[ResetCamera](ObjectTypes/ResetCamera.md)| |
|33|[StencilSwitch](ObjectTypes/StencilSwitch.md)| |
|34|[RollingRock](ObjectTypes/RollingRock.md)| |
|35|[TriggerSwitch](ObjectTypes/TriggerSwitch.md)| |
|36|[WindPusher](ObjectTypes/WindPusher.md)| |
|37|[WaterSwitch](ObjectTypes/WaterSwitch.md)| |
|38|[BattleMapChange](ObjectTypes/BattleMapChange.md)|UNUSED|