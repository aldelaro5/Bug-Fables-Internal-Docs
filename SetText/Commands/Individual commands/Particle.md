# Particle

Play a particle effect at an [Entity data](../../../TextAsset%20Data/Entity%20data.md)'s position + a specific offset.

## Syntax

````
|particle,name,entity,offsetx,offsety,offsetz|
````

## Parameters

### `name`: string

The name of the particle to play. This must be a valid filename inside the `Assets/Ressources/Prefabs/Particles` or an exception will be thrown.

### `entity`: int | `caller` | `this` | string

The [Entity id](../Entity%20id.md) or designator to play the particle relative to. The int form represents an [Entity id](../Entity%20id.md) and it must be a valid int or an exception will be thrown. If the entity resolves to null, an exception will be thrown. These other values are only applicable if the caller isn't null:

* `this`: Refers to the [tailtarget](../../Notable%20local%20variable/tailtarget.md).
* `caller`: Refers to the caller.
* Anything else: Refers to a [Define](Define.md) if it exists, otherwise, this is interpreted as a regular [Entity id](../Entity%20id.md) which will cause an exception to be thrown.

### `offsetx`: float

The X coordinate of the offset to play the particle relative to `entity`'s position. This must be a valid float or an exception will be thrown.

### `offsety`: float

The Y coordinate of the offset to play the particle relative to `entity`'s position. This must be a valid float or an exception will be thrown.

### `offsetz`: float

The Z coordinate of the offset to play the particle relative to `entity`'s position. This must be a valid float or an exception will be thrown.
