# Terminal flow
This is one of the flow of [Update](../Update.md) which only happens if the battle is about to end or retried.

This flow happens if `cancelupdate` is true or instance.`pausemenu` exists.

The former is only set to true when some kind of terminal coroutine occurs (such as a game over) where we know the battle will end in some ways (but it may be retried). The latter only happens after selecting the change loadout and retry option after a game over which brings up a cut down pause menu.

In this flow, absolutely no update logic occurs. The flow is completely controlled by whichever coroutine set `cancelupdate` or the PauseMenu. This flow is only entered in terminal cases where we know for sure the battle will end or retried.

It is also possible to enter this flow by setting instance.`inbattle` to false which happens when we know the battle will end without retry, but this alone won't technically be the same without setting `cancelupdate` to true. Doing this will still allow [RefreshEXP](../../Visual%20rendering/RefreshEXP.md) calls that normally happens in the other 2 flows only. However, because they only happen if `oldexp` doesn't match `expreward`, it ends up not mattering because no EXP can be gained in any terminal cases (since the battle is already scheduled to end in some ways). It should be noted that the only terminal case where this happens while `cancelupdate` remains false is a sucessfull flee attempt.
