# Kinematicplayer

Set the `isKinematic` property of the player's entity's RigidBody to a value or turn it on temporarily for the rest of this SetText call (with caveats, see the remarks section).

## Syntax

(1)

````
|kinematicplayer,value|
````

(2)

````
|kinematicplayer,temp|
````

## Parameters

### `value`: `true` | `false`

The value to set the isKinematic of the player's entity's RigidBody to. This must be a valid bool or an exception will be thrown.

### `temp`

The presence of this parameter indicates to operate in syntax (2) which will turn on the isKinematic, but will append `|kinematicplayer,false|` at the end of the input string which would cause it to turn it off at the end of the SetText call (with caveats, see the remarks section).

## Remarks

Syntax (2) assumes the input string will not get redirected because if it is, the off command will not be processed. It is only guaranteed to work if the entire input string of the current call is processed to the end without redirection.

This command is only used in syntax (2) when tattling a map or NPC.
