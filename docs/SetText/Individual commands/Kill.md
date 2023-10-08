# Kill

Kills an [Entity](../../Entities/Entity.md) by calling [Death](../../Entities/EntityControl/Notable%20methods/Death.md) (with `activatekill` true) on it.

## Syntax

````
|kill,entity|
````

## Parameters

### `entity`:  `this` | `caller` | int

The [Entity](../../Entities/Entity.md) to kill:

* `this`: Refers to [tailtarget](../Notable%20states.md#tailtarget)'s entity (This only reliably works in [Dialogue mode](../Dialogue%20mode.md) otherwise, it will take the last value of [tailtarget](../Notable%20states.md#tailtarget) which is potentially undefined behavior).
* `caller`: Refers to the caller if it is not null. If it is, this command does nothing.
* int: Refers to any other [Entity id](../Common%20commands%20id%20schemes/Entity%20id.md). If it resolves to null, an exception will be thrown.

Any non int value that isn't in one of the predefined values above will cause an exception to be thrown.

## Remarks

There is a special case for Shop and CaravanBadge entities where it will call SetBadgeShop(true) on the entity's [NPCControl](../../Entities/NPCControl/NPCControl.md) for the former and on the shopkeeper of the [entity](../../Entities/Entity.md)'s [NPCControl](../../Entities/NPCControl/NPCControl.md) for the latter.

This command only kills the entity if its [NPCControl](../../Entities/NPCControl/NPCControl.md) isn't null and it's not a CaravanBadge.

Once the [entity](../../Entities/Entity.md) has been killed if applicable, the player's list of [NPCControl](../../Entities/NPCControl/NPCControl.md) is reset to a new list.
