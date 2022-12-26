# Switch

Toggle or set the hit field of an [Entity](../../../Data%20format/Entity.md)'s npc (this is meant to toggle or set the state of an [Entity](../../../Data%20format/Entity.md) of type switch).

## Syntax

(1)

````
|switch,entity|
````

(2)

````
|switch,entity,value|
````

## Parameters

### `entity`: int

The [Entity id](../Entity%20id.md) to toggle or set its npc's hit field. This must be a valid [Entity id](../Entity%20id.md) resolving to a non null value or an exception will be thrown.

### `value`: `true` | `false`

The value to set the hit field of the `entity`'s npc. This must be a valid bool or an exception will be thrown.

## Remarks

This command was specifically made to allow the Gen & Eri switch puzzle to work inside the Honey Factory.
