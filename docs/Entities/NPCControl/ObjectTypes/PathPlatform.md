# PathPlatform
A platform that moves on a set path defined by multiple absolute position vectors called nodes and can conditionally be stopped depending on the `hit` value of other NPCControl present on the map. Can be operated in loop mode using only 2 nodes or with multiple nodes in path nodes with more options.

## Data Arrays
- `data`: The map entities index to check for their `hit` being true and if any are, it will tell the platform to set its `hit` to true and to start moving along its path in nodes mode. This is optional, the platform is considered as continually moving as normal without any elements
- `vectordata`: The nodes the platform will travel in absolute positioning.
- `dialogues[0].x`: The starting node after truncating to int. This can only be 0 or 1 in loop mode.
- `dialogues[0].y`: The speed multiplier at which the platform moves.
- `dialogues[0].z`: ???
- `dialogues[1].x`: if it's 1, it means to only loop between the first 2 `vectordata` and disregard the `currentnode` logic.
- `dialogues[1].y`: The `actioncooldown` to apply when the platform has reached the end of its path and it should go inactive.
- `dialogues[1].z`: ???
- `dialogues[2].x`: The local scale of the entity's `model` will be multiplied by the 1/10 of this, but this is optional and the scale will not change if it is 0.1 or below
- `dialogues[2].y`: The `electime` of the GlowTrigger. This is optional and the value defaults to 260.0 if it's 0.
- `dialogues[2].z`: ???

## Setup
First, the entity's `rigid` gets effectively locked by disabling gravity, making it kinematic and freezing all constraints. The isStatic on the gameObject is set to false, the `nointeract` to true and the entity's `soundonpause` to true.

From there, the model's scale is modified according to the value of `dialogues[2].x`.

The nodes ??? logic is initialised according to `dialogues[1].x` and `dialogues[0].x`. If we decide to follow the nodes, `currentmode` is set to its respective value, the transform's absolute position is set to the `vectordata` at the index of `currentmode` and the entity's `startpos` is set to that same position.

If the entity's `originalid` is the `Lilypad` [AnimID](../../../Enums%20and%20IDs/AnimIDs.md), the `scol` is disabled and the `boxcol` is recreated with trigger and a size of (5.0, 1.0, 5.0) which overrides all the `boxcol` fields obtained when loading the data.

After, the entity's `alwaysactive` is set to true and its `model`'s tag to `PlatformNoClock`.

If the entity's `originalid` is the `ElectroPlatform` [AnimID](../../../Enums%20and%20IDs/AnimIDs.md), a GlowTrigger is added on the first child of the `model`:
- `getactivecolorfromstart` is set to true
- `parent` is set to this NPCControl
- `glowparts` is initialised to a single element corresponding to the MeshRenderer of the first child of the `model`
- `electimr` is initialised according to the value of `dialogues[2].y` described in the previous section

## Update
First, if the [AnimId](../../../Enums%20and%20IDs/AnimIDs.md) is a `Lilypad`, the `boxcol` if present is kept enabled, otherwise, its enabled will bet set to the `hit` value.

From there, the platform can operate in 2 distinct modes: forever loop between the first 2 `vectordata` or follow a series of nodes defined in `vectordata`. The former is done if `dialogues[1].x` is 1, the latter is done otherwise.

### Loop mode
This is a much simpler mode where only the first 2 `vectordata` are used as nodes.

In this mode, `hit` is set to true and it is never toggled back. This means the platform will continually move according to the `speedmultiplier` field which servers as the factor of the position lerp to perform between `vectordata[0]` and `vectordata[1]`.

On the first update, if `dialogues[0].x` isn't negative, it will be the starting value of `speedmultiplier`. This is a one off: after setting the field, it is reset to -1. It should be noted that the value set is a cast to int meaning it can only be 0 or 1 which coresponds to the starting node chosen among the 2.

The platform is considered active when either `data` is empty or any of its elements refers to a map NPCControl whose `hit` is true. If it is active, `speedmultiplier` gets incremented by the game's frametime * `dialogues[0].y` / 1000.0. If it's inactive, it gets decremented instead. Basically, the platform moves forward when active and backward when inactive.

The position set portion starts by clamping `speedmultiplier` from 0.0 to 1.0 and then setting the position using a Vector3.Lerp from `vectordata[0]` to `vectordata[1]` with a factor of `speedmultiplier`.

Finally, Unless the [AnimId](../../../Enums%20and%20IDs/AnimIDs.md) is a `Lilypad`, `speedmultiplier` is below 1.0 and the entity's `sound` is playing, then [PlaySound](../../EntityControl/EntityControl%20Methods.md#PlaySound) is called on the entity with the `PlatformMove` clip set to loop.

### Path mode
What happens each updates depends on the value of `hit` which gets toggled on and off in a very systematic manner. The details involves many different fields that interacts with each other:
- `actioncooldown`: The cooldown used to stop movement whenever the platform reaches its last node AND it is considered deactivated. This is refreshed to be `dialogues[1].y` each time the platform goes to a new node with `hit` being set to true.
- `speedmultiplier`: The factor to use when lerping the platform positions between nodes. It's set to 0.0 when going to a new node until it reaches 1.0+. It progressively increases by the frametime of the game * `dialogues[0].y` / 1000.0.
- `bounces`: The previous node index visited which will be used as the `from` vector when lerping.
- `currentnode`: The next node index to visit which will be used as the `to` vector when lerping.

#### When `hit` is false
Before anything happens, the entity's `sound` is set to not loop and its `model` tag is set to `Platform`.

The platform is considered active when either `data` is empty or any of its elements refers to a map NPCControl whose `hit` is true. When a platform is active, it will have its `hit` set to true on the first update in which it was false. This will cause the fields to be set in such a way that `bounces` will always lag behind by 1 from `currentnode` when going forward. When the platform reaches the end and it's still active, it will set `currentnode` to 0 which will make the platform go back to its starting point, but ignoring all the nodes, it simply moves straight towards the start and continues from there.

But it is possible for the platform to go inactive during its forward path. When this happens, this is where the `actioncooldown` comes into play if one was defined (the platform doesn't move if it's 0.0 or below). It will be exhausted which will make the platform stationnary until it expires. When it does expire, if the platform's `currentnode` is higher 0 (its angles wasn't the starting ones already), it will decrement it, but have `bounces` be one node above. This essentially makes the platform go backwards from the nodes list, but following its rotation path until the start (with cooldown applied) and it also sets `hit` to true.

#### When `hit` is true
There is a special case before anything happens: whenever `hit` goes to true, it is possible that `currentnode` and `bounces` points to the same node. It can happen if there's only one node defined in `vectordata`. If this occurs, the update logic just ends abrutply because there is no need to move the platform and it will remain stationnary forever.

Then, the entity's `model` tag is set to `PlatformNoClock`. Unless the [AnimId](../../../Enums%20and%20IDs/AnimIDs.md) is a `Lilypad` and the entity's `sound` is playing, then [PlaySound](../../EntityControl/EntityControl%20Methods.md#PlaySound) is called on the entity with the `PlatformMove` clip set to loop.

From there, this is where the movement is done until `speedmultipiler` reaches 1.0 or above in which case, it's reset to 0.0 and `hit` is set to false again.

When moving the platform, the lerping using will be the standard Vector3.Lerp, but there is a special case if there's only 2 nodes in `vectordata`. When this happens, the game's SmoothLerp is used instead which is a much slower and spread out lerping procedure.

## OnTriggerEnter if the other collider is the player
There is some dead logic here where nothing happens, but the conditions are all of these being true:
- The current [area](../../../Enums%20and%20IDs/librarystuff/Areas.md) is `WildGrasslands`
- The entity `originalid` is the `Lilypad` [animid](../../../Enums%20and%20IDs/AnimIDs.md)
- The player entity is `onground`
