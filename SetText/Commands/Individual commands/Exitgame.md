# Exitgame

Immediately stops the game's execution if called from the StartMenu's settings or reset the game's scene after a transition if called from the pause menu after loading a file.

## Syntax

````
|exitgame|
````

## Parameters

None.

## Remarks

This command also sets [End](End.md) to true.

The way the game determines if it should immediately stop the execution or reset is by checking the pause menu is present and that calledfrommain is true. This is the case on the StartMenu's settings because the settings menu actually requires the pausemenu to be present and the flag is set when going to this ItemList from the StartMenu. If it was called from the StartMenu, this will stop the execution. If it wasn't, it will first fade the music by a rate of 0.15, play a [FadeIn](FadeIn.md) transition with 0.1 speed and yield for 0.1 seconds before finally resetting the game's scene.

This command is unused under normal gameplay. The game has more controlled ways to do the same task and it's all handled externally of SetText now. This command was presumably made obsolete.
