# WindPusher
A gust of winds that pushes the player away from it ???

## Data Arrays
- `data[0]`: The map NPCControl id whose `npcdata.hit` field determines if the wind particle should render on [SetUp](../SetUp.md). If that `hit` field is false, the particle will be stopped at the start without prewarm. If that field is true OR this data element is negative, it will play the particle starting at the 1 second mark.
- `vectordata[0]`: The absolute position to push the wind at
- `vectordata[1].x`: The push force of the wind ???
- `vectordata[1].y`: The horizontal length of the `boxcol`
- `vectordata[1].z`: The size.y of the `boxcol` when operated horizontally or size.z when operated vertically
- `vectordata[2].x`: The startLifetime of the wind particles, defaults to the 1/5 of the distance between the root transform and `vectordata[0]`
- `vectordata[2].y`: The startSpeed of the wind particles, defaults to the default constant speed defined in the prefab (5.0) * `vectordata[1].x` * 20.0.

## Setup
First, the entity's `alwaysactive` is set to true.

The method then determines if we should operate the windpusher vertically or horizontally. It's vertically if the vertical distance between the root transform and `vectordata[0]` is greater than 3.0 and it's horizontally otherwise.

Then, the `boxcol` is recreated by adding a fresh trigger one. This implies that all fields pertaining to the `boxcol` when loading the entity data are overriden and they are not honored. The `boxcol` size and center depends on whether we operate horizontall or vertically:
- horizontally:
  - size of (`vectordata[1].y`, `vectordata[1].z`, distance between the root transform and `vectordata[0]`) 
  - center of (0.0, 0.0, hald of the distance between the root transform and `vectordata[0]`)
- vertically: 
  - size of (`vectordata[1].y`, distance between the root transform and `vectordata[0]`, `vectordata[1].z`) 
  - center of (0.0, hald of the distance between the root transform and `vectordata[0]`, 0.0)

`internalvector` is initialised to a single element containing the up vector of the transform or the forward one. It's the former if the vertical distance between the root transform and `vectordata[0]` is greater than 3.0. It is the latter otherwise. This is supposed to indicate the wind flow direction.

The root transform is then set to face `vectordata[0]`.

`internalparticle` is initialised to a single element that corresponds to the particle system of an instance of the `Prefabs/Particles/WindFunnel` prefab childed to the root transform with a position of (0.5, 0.5, 0.5). The angles are set to Vector3.zero when operating horizontally or (-90.0, 0.0, 0.0) when operating vertically. From there, the data arrays loading as described in the section above occurs to adjust the particle system.

## Update
This update's sole purpose is to determine if we should keep the `internalparticle[0]` playing by calling Play on it or stop it by calling Stop on it. This only applies if `data[0]` isn't negative.

If the `hit` field of the NPCControl at id `data[0]` is true or this object is itself `hit`, then we are playing the particle if it wasn't already playing. Otheriwse, if it was playing, we stop it.

## OnTriggerEnter if the other collider is the player
All collisions between the player entity `detect` and the `boxcol` are ignored and the player entity `hitwall` is set to false.

## OnTriggerStay
If the player is free (with flying ignored), the other gameObject is the player and either `data[0]` is -1 or the `hit` field of the entity `npcdata` whose map entity id is `data[0]` is true, the player position is incremented by `internalvector[0]` * (the game's frametime * `vectordata[1].x`) * (0.25 if the player entity is `onground`, 1.05 if it's not).
