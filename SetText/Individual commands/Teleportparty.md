# Teleportparty

Teleports the party's entities near the player's [Entity](../../Entities/Entity.md) which optionally includes any temporary non party follower or teleport just the party entities at a specified distance and direction from the player's [Entity](../../Entities/Entity.md).

## Syntax

(1) (Teleports only the party followers near the player's entity)

````
|teleportparty|
````

(2)

````
|teleportparty,includetemp|
````

(3)

````
|teleportparty,distance,direction|
````

## Parameters

### `includetemp`: `true` : `false`

Whether to include any non party temporary followers to teleport near the player. This must be a valid bool or an exception will be thrown. Specifying `false` is equivalent to syntax (1) and is thus the default value.

### `distance`: float

The distance to teleport the party entities from the player. This must be a valid float or an exception will be thrown.

### `direction`: int | `string`

The direction to teleport the party entities from the player. Here are the different values possible in int and string form (the string form is case insensitive):

|int|string|
|---:|------|
|0|Right|
|1|Left|
|2|Up|
|3|Down|
|4|Away|
|5|Center|

## Remarks

For syntax (1) and (2), the party entities are teleported so they match the position of the player with removal of velocity, but each party [Entity](../../Entities/Entity.md) is offset forward to the camera by 0.025 from each other and the player. This is done to combat possible Z fighting.

For syntax (2), it does the same than (1), but it also teleports the temporary non party followers at the player's position. Unlike the party teleport, there is no Z fighting mitigation here. 

For either (1) and (2), Chompy if she exists gets teleported 0.3 in front of the player after a frame.

Syntax (3) operates using a different override of TeleportFollowers which accepts the `distance` and `direction` parameters. This only applies to the party entities and it will not teleport the temporary non party followers. The teleport happens only if the current distance between the entity is more than `distance` and it is done relative to the entity's following target (which prevents Z fighting). The teleport for the party entities includes removing any velocity.

Here are the different directions's logic in determining the position:

|Direction|Description|
|---------:|-----------|
|Right|Teleports to the right of the entity's following|
|Left|Teleports to the left of the entity's following|
|Up|Teleports behind the entity's following|
|Down|Teleports in front of the entity's following|
|Away|Teleports away from the caller with 0.1 forward to the camera relative to the entity's following|
|Center|Teleports each at the entity's following with an offset of 0.1 forward of the camera between each other and the entity's following|

This will also teleports Chompy if she exists 0.1 forward to the camera if she is further away from the player than `distance`.
