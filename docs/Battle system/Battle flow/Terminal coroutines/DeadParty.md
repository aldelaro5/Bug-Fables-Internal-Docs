# DeadParty
This is a wrapper terminal coroutine that will call [GameOver](GameOver.md) or [ReturnToOverworld](ReturnToOverworld.md) depending on MainManager.`battlelossevent`. It is called when the game finds out the party is fully dead.

- `cancelupdate` is set to true changing to a [terminal flow](../Update%20flows/Terminal%20flow.md)
- `currentturn` is set to `partypointer[0]` (the front member)
- A second is yielded
- If MainManager.`battlelossevent` is false, `gameover` is set to a new [GameOver](GameOver.md) call without skipsetup if it wasn't running already
- Otherwise, this means the game requested the battle to not game over if lost:
    - MainManager.`battleresult` is set to false (the battle was still lost)
    - A [ReturnToOverworld](ReturnToOverworld.md) coroutine starts
- `forceattack` is reset to -1 (there are no valid player party members target anyway)
- `mainturn` is set to null
