# DoClock
There is a method that is called using InvokeRepeating on [event](../Enums%20and%20IDs/Events.md) 8 (selecting new file) and event 22 (loading a save file) such that it gets called every second by Unity.

```cs
public void DoClock()
```
The primary purpose of this method is to count in game time with seconds of precision. The in game timer is composed of 3 fields saved on the [save file](../External%20data%20format/Save%20File.md):

- `clockhour`: The amount of hours (counts up to 9999 where it can no longer increase)
- `clockmin`: The amount of minutes, goes from 0 to 60 before going back to 0 and incrementing `clockhour` (so only from 0 to 59 are visible in game)
- `clocksec`: The amount of seconds, goes from 0 to 60 before going back to 0 and incrementing `clockmin` (so only from 0 to 59 are visible in game). Since the method is assumed to be called each second by Unity, each call causes this field to increases at the start of the method

What's interesting is because DoClock is meant to run every second, the game also treats it as a game life cycle methods where other various logic can happen since a periodically invoking methods like this turns out to be useful for various things. So while managing the in game timer is the primary function of DoClock, it does a lot more than that.

Here's the full logic of the method excluding the timer count mentioned above:

- If [flagvar](../Flags%20arrays/flagvar.md) 37 (Venus's price of healing in berries) is 0, it is set to 8. 8 is the default value it should have, but for some reasons, the game is enforcing this on DoClock rather than early after boot
- If `clocksec` after its increment is a number divisible by 5 (so every 5 seconds) while not in a `roomtransition` (meaning no [map loading](../MapControl/Map%20loading.md) or [save](../SetText/Individual%20commands/Save.md) SetText command is happening), Resources.UnloadUnusedAssets and GC.Collect are called (see below for the implications and reasoning of why these calls are there)
- If the game is rolling over the next minute (`clocksec` becaome 60) and if flagvar 26 (Berry bank balance) is above 0, it is changed if `clockmin` (after rolling it over) is 30 or 60 (so every 30 minutes of in game time). This only applies if [flags](../Flags%20arrays/flags.md) 254 is true (signed up for a bank account). The following is the formula to calculate the new value:
    - Take the existing value of flagvar 26 and multiply it by 0.02 (0.04 instead if flag 630 is true which is reached a balance of at least 500 for the first time)
    - Floor the number obtained above and clamp it from 1 to 75, this is the calculated increases of the balance
    - Set flagvar 26 to its existing value + the balance increase calculated above clamped from 0 to 10000
- If `timeddemo` is true (an UNUSED field), the demo timer logic happens (details in the [unused fields](Unused%20fields.md) documentation, it's an UNUSED feature, but it remains functional)
- If `player` exists (the game isn't loading a map) while not in a `battle`:
    - If FreePlayer returns true, RefreshPlayer is called without onlycollider (both methods are documented [here](../General%20systems/Player%20party.md#other-party-utilities)). NOTE: Due to the incorrect handling of the entities's `onground` by RefreshPlayer, it causes all player entities to be incorrectly seen as being in the air for a frame every second. This can lead to incorrect physics being applied for that frame such as slowly falling down a slope when not moving 
    - If [flag](../Flags%20arrays/flags.md) 377 is false (not able to buy `BlackCherry` yet) while a `BlackCherry` is in `items[0]` (regular items inventory), flag 377 is set to true (the player is now able to buy `BlackCherry` at the Underground Bar)

## Resources.UnloadUnusedAssets and GC.Collect
Every 5 seconds outside of [map loading](../MapControl/Map%20loading.md), 2 methods are called that have huge implications on the performance of the game:

- Resources.UnloadUnusedAssets
- GC.Collect

The first thing to mention is the GC.Collect call does nothing because it turns out that Resources.UnloadUnusedAssets calls it anyway. Double calling GC.Collect doesn't really have bad effects, but it does mean that at least 1 GC.Collect runs every 5 seconds.

Asking Unity to unload all loaded, but unreferenced assets and forcing the GC to collect unreferenced memory is VERY intensive to do every 5 seconds. It's intense enough that it is believed to be the leading cause of occasional stutters that can be observed while playing the game. This is due to the fact that these procedures runs synchronously and notably the GC.Collect part will completely stall the main thread until it is done collecting.

However, as strange of a memory management strategy this might seem, there's profiling evidences suggesting this was necessary to workaround the overall poor assets management of the game. Profiling the game using the Unity profiler reveals that there's a ton of assets allocations that are done ephemerally multiple times without being cached. This means the game frequently and unecessarily creates GameObjects or components that are recycled in the immediate future which puts huge preasure on the RAM and thus the GC and Unity's assets memory.

This is known to be a major problem in [regular letter rendering](../SetText/Letter%20Rendering%20Methods/Regular%20Letter%20Rendering.md) on SetText because each letter slots constantly gets their assets recycled and reallocated. In fact, it's the entire reason [single letter rendering](../SetText/Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md) exists: it was found that the regular rendering involved too many assets at once and caused several performance issues on platforms like the Nintendo Switch. That platform even has a history of crashing due to causes related to the game allocating too much RAM at once. Lastly, it's also known to be a huge problem with the way material properties are managed because the game largely doesn't leverage material property blocks.

Due to this, it explains why the game wants to do such intensive and potentially stutter inducing tasks every 5 seconds. It's essentially a worst case scenario workaround that can't properly be addressed without significantly changing the design of several systems that manages assets so they are more optimised. Ideally, the game should let Unity figure out when to do these procedures, but because the game puts huge preasure on the RAM constantly, it was decided to constantly spam these calls.
