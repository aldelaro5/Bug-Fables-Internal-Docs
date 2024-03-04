# ExitBattle
This method is a wrapper around the [ReturnToOverWorld](../Terminal%20coroutines/ReturnToOverworld.md) terminal coroutine. It is primarily used as part of [EventDialogues](../EventDialogue.md) and is also the last step of [AddExperience](../Terminal%20coroutines/AddExperience.md).

- [ReturnToOverWorld](../Terminal%20coroutines/ReturnToOverworld.md) is called
- `cancelupdate` is set to true changing to a [Terminal flow](../Update.md#terminal-flow)
- `enemy` is set to false which normally makes the flow go to the [player phase](../Update.md#player-phase), but we just changed to a terminal flow so nothing will happen
- `action` is set to true, but not only ReturnToOverworld already did that, this doesn't do anything because we are in a terminal flow
