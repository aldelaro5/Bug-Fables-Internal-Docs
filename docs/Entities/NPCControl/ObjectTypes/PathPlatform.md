# PathPlatform
A platform that moves on a set path defined by multiple absolute position vectors called nodes and can conditionally be stopped depending on the `hit` value of other NPCControl present on the map. Can be operated in loop mode using only 2 nodes or with multiple nodes in path mode with more options.

## Data Arrays
- `data`: The map entities ids to check for their `hit` being true and if any are, it will tell the platform to set its `hit` to true and to start moving along its path in path mode. This is optional, the platform is considered as continually moving as normal without any elements
- `vectordata`: The nodes the platform will travel in absolute positioning
- `dialogues[0].x`:The meaning depends on the mode:
  - In path mode, this is the starting node after truncating to int
  - In loop mode, this is the starting `speedmultiplier` (the starting node index) after truncating to int. This means only \[0.0, 2.0\[ are valid values
- `dialogues[0].y`: The speed multiplier at which the platform moves.
- `dialogues[1].x`: if it's 1, it means to place the platform in loop mode by only using the first 2 `vectordata` and disregard the `currentnode` logic
- `dialogues[1].y`: The `actioncooldown` to apply when the platform has reached the end of its path and it should go inactive.
- `dialogues[2].x`: The local scale of the entity.`model` will be multiplied by the 1/10 of this, but this is optional and the scale will not change if it is 0.1 or below
- `dialogues[2].y`: If not 0, the `electime` of the GlowTrigger when the entity.`originalid` is the  `ElectroPlatform` [AnimID](../../../Enums%20and%20IDs/AnimIDs.md). This is optional and the value defaults is 260.0

## Additional data
- `speedmultiplier`: OVERRIDEN (this is used as a value between 0.0 and 1.0 that represents the lerp factor of the platform position between 2 nodes as it moves)
- `boxcol`: If entity.`originalid` is the `Lilypad` [AnimID](../../../Enums%20and%20IDs/AnimIDs.md), the `boxcol` is created manually, but it doesn't remove the existing one meaning under these circumstances, the entity shouldn't be specified that it has one since it would make it so there's 2 `boxcol` with the loaded version not being accessible by the game

## Setup
- entity.`rigid` gets effectively locked by disabling gravity, making it kinematic and freezing all constraints
- isStatic on the gameObject is set to false
- `nointeract` is set to true
- entity.`soundonpause` is set to true
- entity.`model` scale is multiplied by a 1/10 of `dialogues[2].x`
- If `dialogues[1].x` is 1 (loop mode) and `dialogues[0].x` is 1 (the starting node is the second one), the `speedmultiplier` is set to 1.0
- If `dialogues[1].x` is 0 (path mode):
  - `currentnode` is set to `dialogues[0].x`
  - The position is set to the `vectordata` at the `currentnode`
  - entity.`startpos` is set to the new position
- If the entity.`originalid` is the `Lilypad` [AnimID](../../../Enums%20and%20IDs/AnimIDs.md):
  - The `scol` is disabled
  - the `boxcol` is recreated with trigger and a size of (5.0, 1.0, 5.0) which overrides all the `boxcol` fields obtained when loading the data.
- entity.`alwaysactive` is set to true
- entity.`model` tag is set to `PlatformNoClock`
- If entity.`originalid` is the `ElectroPlatform` [AnimID](../../../Enums%20and%20IDs/AnimIDs.md), a GlowTrigger is added on the first child of the `model`:
  - `getactivecolorfromstart` is set to true
  - `parent` is set to this NPCControl
  - `glowparts` is initialised to a single element corresponding to the MeshRenderer of the first child of the `model`
  - `electime` is initialised to 260.0 unless `dialogues[2].y` exists and isn't 0 where it will take that value instead

## Update
If entity.`originalid` is the `Lilypad` [AnimId](../../../Enums%20and%20IDs/AnimIDs.md), the `boxcol` if present is kept enabled, otherwise, its enabled will bet set to the `hit` value.

From there, the platform can operate in 2 distinct modes: forever loop between the first 2 `vectordata` where the current one is tracked by `speedmultiplier` or follow a series of nodes defined in `vectordata` where the current one is tracked by `currentnode`. The former is done if `dialogues[1].x` is 1, the latter is done otherwise.

### Loop mode
This is a much simpler mode where only the first 2 `vectordata` are used as nodes.

In this mode, `hit` is set to true and it is never toggled back. This means the platform will continually move according to the `speedmultiplier` field which serves as the factor of the position lerp to perform between `vectordata[0]` and `vectordata[1]`.

On the first update, if `dialogues[0].x` isn't negative (which it shouldn't be on the first Update), it will be the starting value of `speedmultiplier`. This is a one off: after setting the field, it is reset to -1. It should be noted that the value set is a cast to int meaning it can only be \[0.0, 2.0\[ which coresponds to the starting node chosen among the 2.

The platform is considered active when either `data` is empty or any of its elements refers to a map NPCControl whose `hit` is true. If it is active, `speedmultiplier` gets incremented by the game's frametime * `dialogues[0].y` / 1000.0. If it's inactive, it gets decremented instead. Basically, the platform moves forward when active and backward when inactive.

The position set portion starts by clamping `speedmultiplier` from 0.0 to 1.0 and then setting the position using a Vector3.Lerp from `vectordata[0]` to `vectordata[1]` with a factor of `speedmultiplier`.

Unless entity.`originalid` is the `Lilypad` [AnimId](../../../Enums%20and%20IDs/AnimIDs.md), `speedmultiplier` is below 1.0 and the entity's `sound` is playing, then [PlaySound](../../EntityControl/EntityControl%20Methods.md#PlaySound) is called on the entity with the `PlatformMove` clip set to loop.

### Path mode
What happens each updates depends on the value of `hit` which gets toggled on and off in a very systematic manner. The details involves many different fields that interacts with each other:
- `actioncooldown`: The cooldown used to stop movement whenever the platform reaches its last node AND it is considered deactivated. This is refreshed to be `dialogues[1].y` each time the platform goes to a new node with `hit` being set to true.
- `speedmultiplier`: The factor to use when lerping the platform positions between nodes. It's set to 0.0 when going to a new node until it reaches 1.0+. It progressively increases by the frametime of the game * `dialogues[0].y` / 1000.0.
- `bounces`: The previous node index visited which will be used as the `from` vector when lerping.
- `currentnode`: The next node index to visit which will be used as the `to` vector when lerping.

#### When `hit` is false
Before anything happens, the entity's `sound` is set to not loop and its `model` tag is set to `Platform`.

The platform is considered active when either `data` is empty or any of its elements refers to a map NPCControl whose `hit` is true. When a platform is active, it will have its `hit` set to true on the first update in which it was false. This will cause the fields to be set in such a way that `bounces` will always lag behind by 1 from `currentnode` when going forward. When the platform reaches the end and it's still active, it will set `currentnode` to 0 which will make the platform go back to its starting point, but ignoring all the nodes, it simply moves straight towards the start and continues from there.

But it is possible for the platform to go inactive during its forward path. When this happens, this is where the `actioncooldown` comes into play. It will be exhausted which will make the platform stationnary until it expires. When it does expire, if the platform's `currentnode` is higher than 0 (it's further than the starting node), it will decrement it, but have `bounces` be one node above. This essentially makes the platform go backwards from the nodes list, but following its movement path until the start (with cooldown applied) and it also sets `hit` to true.

#### When `hit` is true
There is a special case before anything happens: whenever `hit` goes to true, it is possible that `currentnode` and `bounces` points to the same node. It can happen if there's only one node defined in `vectordata`. If this occurs, the update logic just ends abrutply because there is no need to move the platform and it will remain stationnary forever.

Then, the entity.`model` tag is set to `PlatformNoClock`. Unless entity.`originalid` is the `Lilypad` [AnimId](../../../Enums%20and%20IDs/AnimIDs.md) and the entity's `sound` is playing, then [PlaySound](../../EntityControl/EntityControl%20Methods.md#PlaySound) is called on the entity with the `PlatformMove` clip set to loop.

From there, this is where the movement is done until `speedmultipiler` reaches 1.0 or above in which case, it's reset to 0.0 and `hit` is set to false again.

When moving the platform, the lerping using will be the standard Vector3.Lerp, but there is a special case if there's only 2 nodes in `vectordata`. When this happens, the game's SmoothLerp is used instead which is a much slower and spread out lerping procedure.

## OnTriggerEnter if the other collider is the player
There is some dead logic here where nothing happens, but the conditions are all of these being true:
- The current [area](../../../Enums%20and%20IDs/librarystuff/Areas.md) is `WildGrasslands`
- The entity `originalid` is the `Lilypad` [animid](../../../Enums%20and%20IDs/AnimIDs.md)
- The player entity is `onground`

## Hazards.OnObjectStay
If the NPCControl passed has this type, it returns true which allows it to stay in collision with the Hazards.

## Effects of the `Platform` and `PlatformNoClock`
There are some special logic implicated by the platform having these tags. Mainly, the game will often child GameObjects of interests to the platform when they are getting on it and child them back to where they were when no longer being on it.

Specififcally, the GroundDetector's OnTriggerStay will check if the other collider has either of these tags and do the childing of the entity if it does. Additionally, the entity's `noclock` is set to true if the tag is `PlatformNoClock` and to false if it's `Platform`. On the component's OnTriggerExit, this is all undone, but there's a special case for the player or `PFolower` where it's parented to the root instead of the map.

The Hornable component also does this which is related to the ice cube of a [Dropplet](Dropplet.md). If the other collider has either tag, OnTriggerEnter / OnCollisionEnter will child the other transform to the platform. This is undone on the component's OnTriggerExit if the other collider has either of the tags.

As for what the entity's `noclock` does, normally, on MainManager.DoClock, the method RefreshPlayer is called when the player is free and it would normally set the `onground` to false and root all playerdata entities (this incidentally has a known issue where the frictions gets toggled off for one frame every second). `noclock` is a field that will prevent this logic from happening so it prevents the rooting of the player to the scene. In the case of the PathPlatform, it means that this logic doesn't happen as long as the player is on the platform AND it is active (the logic is free to do its thing when the platform goes inactive).

