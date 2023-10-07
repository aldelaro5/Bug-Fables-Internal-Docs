# Cardbattle

Start a Spy Card battle with an opponent or the caller with a configurable map and deck; then yield until the battle is over.

## Syntax

(1)

````
|cardbattle,opponent|
````

(2)

````
|cardbattle,opponent,map|
````

(3)

````
|cardbattle,opponentid,map,opponentdeck|
````

## Parameters

### `oppenentid`: int | `this`

The opponent the player will be against for the battle. The int form corresponds to the [AnimID](../../Enums%20and%20IDs/AnimIDs.md) of the opponent and it must be a valid one or an exception will be thrown.

`this` corresponds to caller's [AnimIDs](../../Enums%20and%20IDs/AnimIDs.md) and the caller must not be null when using this value or an exception will be thrown.

### `map`: int

The battle map to use for the battle. This must be -1 or a valid battle map id or an exception will be thrown. Since there 73 battle maps defined, the value must be between 0 and 72 or -1 which means to automatically select it based on the current map. TODO: Document these? The default value is -1.

### `opponentdeck`: `@`int | array split by `@`: int

The deck the opponent will use for the battle. The `@`int form corresponds to a preset deck and the int part must be a valid preset deck id or an exception will be thrown. Since there are 15 preset decks, only values from 0 to 14 are valid (TODO: document these?) The array form specifies the card ids directly and each int must be a valid card id or an exception may be thrown during the battle.

TODO: document carddata in general so it's clear what is a valid card id.

## Remarks

Before starting the battle, the texttail is saved. After starting the battle, the [message](../Notable%20states.md#message) lock is forced to be released. From there, the textbox is hidden with a shrink animation and a yield of 1 second. Finally, the yield until the battle is over starts. The message lock is grabbed again, the texttail is restored, the blinker is reassigned to the one it had before and the current textbox is set to reveal with a grow animation.

It should be noted that unlike [battle](Battle.md) which restores the message lock using the previous value, this one forces it on after the battle. This means it is potentially very unsafe to process this command in [non Dialogue mode](../Dialogue%20mode.md#non-dialogue-mode) because after the battle, the SetText call will have the message lock without being able to release it. This can result in a softlock.

This temporarily violates the rule that only one SetText call in [Dialogue mode](../Dialogue%20mode.md) may run because it is now possible to have this command process in Dialogue mode while letting the battle itself call SetText in Dialogue mode, but this is safe because should this happen, this command will stall SetText until the battle is over. This means that the original call wouldn't do anything that can interfere with any calls done during the battle. It is anyway necessary: CardBattle needs [Dialogue mode](../Dialogue%20mode.md) in the event of a tutorial being given.
