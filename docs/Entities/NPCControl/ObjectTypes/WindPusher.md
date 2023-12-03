# WindPusher
A zone that renders wind particles moving towards a point. If the player enters that zone, they will be pushed towards the same direction then the wind. The wind force, wind target position, wind sizes and the visual lifetime / speed are all congurable. There is also the option to only have this object in effect when another map entity `hit` value is true.

## Data Arrays
- `data[0]`: If 0 or above, the map entity id of the entity whose `hit` value determines if the wind particle should be playing or not on Update. If it's negative, the wind particles always play
- `vectordata[0]`: The end position of the wind from the wind pusher position
- `vectordata[1].x`: The base push force of the wind in units per second when the player is inside the wind zone
- `vectordata[1].y`: The `boxcol`.size.x
- `vectordata[1].z`: The `boxcol`.size.y when operated horizontally or `boxcol`.size.z when operated vertically
- `vectordata[2].x`: If 0.1 or above, the startLifetime of the wind particles. This is optional, if it doesn't exist or it's below 0.1, the startLifetime will be 1/5 of the distance between the root transform and `vectordata[0]`
- `vectordata[2].y`: If 0.1 or above, the startSpeed of the wind particles. This is optional, if it doesn't exist or it's below 0.1, the startSpeed will be to the default constant speed defined in the prefab (5.0) * `vectordata[1].x` * 20.0.

## Additional data
- `boxcol`: OVERRIDEN. The `boxcol` is created on SetUp manually, but it doesn't remove the existing one meaning under these circumstances, the entity data shouldn't specify that it has one since it would make it so there's 2 `boxcol` with the loaded version not being accessible by the game

## Setup
The entity.`alwaysactive` is set to true.

The method then determines if we should operate the windpusher vertically or horizontally. It's vertically if the y distance between the wind pusher and `vectordata[0]` is greater than 3.0 and it's horizontally otherwise.

The wind pusher y angle (not the x and z ones) is then set to face `vectordata[0]` which alignes the wind pusher horizontally with its wind end position (no vertical alignement is needed because the `boxcol` will be adjusted accordinginly and the wind particles gets their own rotation when operating vertically).

The `boxcol` is created by adding a fresh trigger one. This implies that all fields pertaining to the `boxcol` when loading the entity data are overriden and they are not honored. The `boxcol` size and center depends on whether we operate horizontall or vertically:

- horizontally:
    - size of (`vectordata[1].y`, `vectordata[1].z`, distance between the root transform and `vectordata[0]`) 
    - center of (0.0, 0.0, hald of the distance between the root transform and `vectordata[0]`)
- vertically: 
    - size of (`vectordata[1].y`, distance between the root transform and `vectordata[0]`, `vectordata[1].z`) 
    - center of (0.0, hald of the distance between the root transform and `vectordata[0]`, 0.0)

`internalvector` is initialised to a single element containing the up vector of the transform if operated vertically or the forward one if operated horizontally. This is supposed to indicate the normalised wind flow direction.

`internalparticle` is initialised to a single element that corresponds to the particle system of an instance of the `Prefabs/Particles/WindFunnel` prefab childed to the wind pusher with a position of (0.5, 0.5, 0.5). The angles are set to Vector3.zero when operating horizontally or (-90.0, 0.0, 0.0) when operating vertically (which makes it point up due to the prefab itself already having a 89.855 degrees rotation in y). From there, the data arrays loading as described in the section above occurs to adjust the particle system.

## Update
This update's sole purpose is to determine if we should keep the `internalparticle[0]` playing by calling Play on it or stop it by calling Stop on it. This only applies if `data[0]` isn't negative.

If the `hit` field of the NPCControl at id `data[0]` is true or this object is itself `hit`, then we are playing the particle if it wasn't already playing. Otheriwse, if it was playing, we stop it.

## OnTriggerEnter if the other collider is the player
All collisions between the player.entity.`detect` and the `boxcol` are ignored and the player.entity.`hitwall` is set to false.

## OnTriggerStay
If the player is free (with flying ignored), the other gameObject is the player and either `data[0]` is -1 or the `hit` field of the entity `npcdata` whose map entity id is `data[0]` is true, the player position is incremented by `internalvector[0]` (the direction of the wind normalized) * (the game's frametime * `vectordata[1].x`) * (0.25 if the player entity is `onground`, 1.05 if it's not).
