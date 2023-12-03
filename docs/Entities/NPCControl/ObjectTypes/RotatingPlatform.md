# RotatingPlatform
The same than [PathPlatform](PathPlatform.md), but path mode will have the nodes be euler angles vectors instead and the platform will rotate through these angles instead of moving. Loop mode is left unchanged, but is considered invalid.

> NOTE: While it is possible to use loop mode on an object of this type, this isn't considered valid because on SetUp, the angles of the platform are set to the first node so they can't be position vectors in loop mode. It also excludes the `Lilypad` exclusive logic on SetUp. Use [PathPlatform](PathPlatform.md) in loop mode for this instead and only use this type in path mode when rotating is desired instead of movement.

## Data Arrays
- `data`: The same than [PathPlatform](PathPlatform.md)
- `vectordata`: The nodes the platform will rotate in euler angles
- `dialogues[0].x`: In path mode, this isn't used because the starting node is always the first one (0). In loop mode, the same than [PathPlatform](PathPlatform.md)
- `dialogues[0].y`: The same than [PathPlatform](PathPlatform.md)
- `dialogues[1].x`: The same than [PathPlatform](PathPlatform.md)
- `dialogues[1].y`: The same than [PathPlatform](PathPlatform.md)
- `dialogues[2].x`: The same than [PathPlatform](PathPlatform.md)
- `dialogues[2].y`: The same than [PathPlatform](PathPlatform.md)

## Setup
- entity.`rigid` gets effectively locked by disabling gravity, making it kinematic and freezing all constraints
- isStatic on the gameObject is set to false
- `nointeract` is set to true
- entity.`soundonpause` is set to true
- entity.`model` angles are set to `vectordata[0]` (It is assumed to be in path mode)
- entity.`model` scale is multiplied by a 1/10 of `dialogues[2].x`
- entity.`alwaysactive` is set to true
- entity.`model` tag is set to `PlatformNoClock`
- If entity.`originalid` is the `ElectroPlatform` [AnimID](../../../Enums%20and%20IDs/AnimIDs.md), a GlowTrigger is added on the first child of the `model`:
  - `getactivecolorfromstart` is set to true
  - `parent` is set to this NPCControl
  - `glowparts` is initialised to a single element corresponding to the MeshRenderer of the first child of the `model`
  - `electime` is initialised to 260.0 unless `dialogues[2].y` exists and isn't 0 where it will take that value instead

## Update
If the [AnimId](../../../Enums%20and%20IDs/AnimIDs.md) is a `Lilypad`, the `boxcol` if present is kept enabled, otherwise, its enabled will bet set to the `hit` value.

From there, the platform can operate in 2 distinct modes: forever move in a loop between the first 2 `vectordata` or rotate by following a series of nodes defined in `vectordata` which are angles vectors. The former is done if `dialogues[1].x` is 1, the latter is done otherwise.

### Loop mode
The same logic than [PathPlatform](PathPlatform.md) occurs here with no changes.

### Path mode
The same logic than [PathPlatform](PathPlatform.md) occurs here, but with one change. If `hit` is true and `bounces` isn't `currennode` then the angles of the platform are lerped instead of its position with a regular lerp (SmoothLerp isn't called even with only 2 nodes). The same parameters are sent to Lerp so the only changes is `vectordata` are euler angles vector instead of positions.

## Hazards.OnObjectStay
Unlike [PathPlatform](PathPlatform.md), this object will return false which disallows it to stay in collision with the Hazards.

## Effects of the `Platform` and `PlatformNoClock`
The same then [PathPlatform](PathPlatform.md).