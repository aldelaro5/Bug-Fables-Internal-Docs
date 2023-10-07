# Overfollower

Toggles or set an override on followers to stop following their followee.

## Syntax

(1)

````
|overfollower|
````

(2)

````
|overfollower,enable|
````

## Parameters

### `enable`: `true` | `false`

Whether to enable the follower override or to disable it. This must be equal to `true` or `false` without case sensitivity or an exception will be thrown.

## Remarks

(1) Toggles the follower overridefrom false to true or true to false depending on its existing state and does a StopForceMove on all party entities except the lead.
(2) Set the follower override depending on the value of `enable`.

This toggles `overridefollower` which is an instance field of MainManager. It is a way for the game to force followers (including player ones) to not follow their followee. Setting it to true will force followers to not follow while setting it to false will have followers do their standard behaviors.
