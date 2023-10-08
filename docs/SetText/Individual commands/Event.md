# Event

Start an [Event](../../Enums%20and%20IDs/Events.md) or set if we are in an event or not.

## Syntax

(1)

````
|event,eventid|
````

(2)

````
|event,inevent|
````

## Parameters

### `eventid`: int

The [Event](../../Enums%20and%20IDs/Events.md) id to start. This must be a valid event id or an exception will be thrown.

### `inevent`: `true` | `false`

Specify whether we are in an event or not. This must be `true` or `false` or an exception will be thrown.

## Remarks

Starting an event will set [end](End.md) to true. During [Dialogue Cleanup](../Life%20Cycle.md#dialogue-cleanup), `overridefollower` is set to true which disables automatic following behaviours from the followers and SetText is instructed to not release the `minipause` lock which the event starting procedure would have set beforehand. The caller of the EventControl is set to the caller of the SetText call.

Syntax (2) is a way to have the game be restricted as if it was in an event without actually being in one. It is recommended to enclose the whole input string with `|event,true|` and `|event,false|` if this is desired. It should be noted that this is unnecessary with syntax (1): the event starting procedure will already set `inevent` to true.
