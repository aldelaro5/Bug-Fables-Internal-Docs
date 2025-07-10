# EventControl
EventControl is a component that allows the game to start [events](../Enums%20and%20IDs/Events.md). It technically has the most logic in the whole game, but that's because it has the logic of ALL events in the game. MainManager has a reference to the only EventControl instance: MainManager.`events`.

## Starting an event
An event is mostly a cutscene that plays when triggered. There are various ways to trigger it managed by the game, but all of them has to end up calling StartEvent:

```cs
public void StartEvent(int id, NPCControl caller)
```
The `id` is the event id to start and `caller` is used to denote what map entity has caused the event (For example, if the event was caused by interacting with an NPC that has the [event interaction](../Entities/NPCControl/Interaction/Event.md), the `caller` will be the NPC). The `caller` may be null if there's no map entity associated with the event.

The method will prepare everything for the event such as setting MainManager.`lastevent`, resetting some fields, positioning the party, interupting some overworld abilities, etc... The exact details won't be mentioned for brevity, but it essentially puts the game in a state where EventControl can become the sole controller on what happens in the game. It actually puts the game in a `minipause` and marks the game as `inevent` which prevents many kinds of logic from happening.

At the end of this process, a coroutine named `EventX` is started where `X` is the `id` parameter sent. This implies that every event started must have a matching coroutine named like this in EventControl. Each event coroutine is parameterless and for the most part don't really have any restrictions when it comes to what they can do. The event coroutine effectively has exclusive control over the entire game so it can script movements, animations and everything it needs to do.

## Ending an event
There is however one responsibility each event coroutine must do: all of them which are terminal (meaning they don't start another event themselves) must call a method named EndEvent:

```cs
private static void EndEvent()
private static void EndEvent(bool overridefollower)
```
The default value for `overridefollower` for the first overload is false.

This method does the necessary steps to undo what StartEvent did. It resets `inevent` to false, exits the `minipause`, etc... The rest of the game will not gain control back until this is called.

## Other methods and utilities
Every other methods in EventControl are general utilities that events frequently needs such as giving an item, messing with the [map limits](../MapControl/Camera%20system.md#camera-limits), etc...

There are also some cached WaitForSeconds static objects instances that are useful for the rest of the game when a yield is needed:

- `sec`: 1.0 second
- `halfsec`: 0.5 seconds
- `quartersec`: 0.25 seconds
- `thirdsec`: 0.33 seconds
- `tenthsec`: 0.1 seconds

There's also some coroutines that are useful outside the game which are static public. When those are used, it's intended to set the coroutine to the `temproutine` public static Coroutine field because these coroutines will set it to null once it's done.

TODO: possibly detail more the private methods since they can be useful
