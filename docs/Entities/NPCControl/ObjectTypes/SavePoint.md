# SavePoint
A save crystal that can be hit using field abilities. It can be blue which only prompts to save the game, yellow which also heals the party and red which alerts a DeadLanderOmega, but doesn't prompt for saving and can optionally heal the party.

## Data Arrays
- `data[0]`: If 1, a `Prefabs/Objects/PolySphere` will be rendered as illumination, it won't otherwise
- `data[1]`: If it's 10 or above, this is a red save crystal that will force a DeadLanderOmega to look somewhere whose `thisid` is this value -10.
- `data[2]`: If 0, this is a yellow save crystal (unless `data[1]` is 10 or above making it render red) which will also heal the party, this a blue save crystal otherwise unless.
- `vectordata[0]`: if `data[1]` is 10 or above, this is the position the DeadLanderOmega will be forced to look at

## Additional data
- `interacttype`: OVERRIDEN (to [SavePoint](../Interaction/SavePoint.md))

## Setup
- The object's isStatic is set to true   
- The `scol` is disabled. 
- The `colliderheight` is set to 3.0.
- The entity.`ccol` is enabled with a radius set to 1.0
- `overrideanim` is set to true
- The entity.`rigid` is effectively fixed by putting it in kinematic mode without gravity and with all constraints frozen
- The `interacttype` is overriden to [SavePoint](../Interaction/SavePoint.md).

`internaltransform` is initialised to be 2 elements and the first one is set as the entity.`model`'s second child (the crystal GameObject) and the second is left default for now.

If `data[0]` is 1, the PolySphere is setup from a `Prefabs/Objects/PolySphere` prefab:
- childed to to `internaltransform[0]`
- local position of (0.0, 0.0, 0.01)
- Local scale of (8.0, 8.0, 5.0)
- `internaltransform[1]` is set to it
- `internalrender` is set as all child MeshRenderer of the Polysphere and for each of them, the renderQueue is set to 3000 + the absolute z position of the PolySphere * 100.0. Additionally, if `data[2]` is 0, each of their color is set to pure yellow without changing the existing transparency.

If `data[1]` is 10 or above, the crystal's color and the `_Emission` color property of the shader is set to pure red.

Otherwise, if `data[2]` is 0, the crystal's color and the `_Emission` color property of the shader is set to pure yellow (this overrides `data[1]` if it was red already).

## Update
The only thing that occurs here is the `internaltransform[0]` (the crystal game object) is rotated 0.3 degrees in the z axis. This actually rotates it on the world's y axis because the pivot point of the crystal is rotated.

## OnTriggerEnter
This does nothing when any of the following are true:
- We are in a `pause` or `minipause`
- The [message](../../../SetText/Notable%20states.md#message) lock is grabbed
- The other gameObject tag isn't `BeetleHorn`, `BeetleDash` or `Icecle` while it's also not the player `beemerang`

If the other gameObject tag was `Icecle` and it has a DestroyOnLayer Kill is called on it. This component normally destroys the object attached when another trigger collider of a specific layer collides to it, but Kill allows it to do it manually.

Then, the `BounceUp` animation is played on the entity.`anim` followed by playing the `Save` sound on the entity at 0.5 volume.

If `data[2]` is 0, Heal is called.

From there, a HitPart particle is played at this position + (0.5, 0.5, 0.5).

If the other gameObject is the player.`beemerang`, its `hit` is set to true.

If `data[1]` is 10 or above and `hit` is false, the game will find a DeadLanderOmega whose `thisid` is `data[1]` - 10 and call ForceLook on it using `vectordata[0]` as the vector. After, `hit` is set to true making it a one time event.

Otherwise, if we aren't in a `timeddemo` (we never are under normal gameplay) and the square distance between this and the player is 30.0 or less, [Interact("save")](../Interact.md) is called.
