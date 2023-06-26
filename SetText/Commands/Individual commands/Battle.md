# Battle

Starts an inescapable enemy battle with all or a specific list of [Enemies](../../../Enums%20and%20IDs/Enemies.md) with configurable battle map and music; then yield until the battle is over.

## Syntax

(1)

````
|battle,enemies|
````

(2)

````
|battle,enemies,music|
````

(3)

````
|battle,enemies,music,map|
````

## Parameters

### `enemies`: `all` | array split by `.`: int

The [Enemies](../../../Enums%20and%20IDs/Enemies.md) list to start the battle with. Every element of the array form must by valid [Enemies](../../../Enums%20and%20IDs/Enemies.md) id and the array must be split by `.` or an exception will be thrown. The `all` value is a shorthand to all [Enemies](../../../Enums%20and%20IDs/Enemies.md) in the game ordered by their id (NOTE: due to a quirk with the battle starting process, this will only end up like a standard Tydal Wyrm fight).

### `music`: string | `null`

The name of the music file without extensions to play during the battle from the `audio/music` resources directory. This must be a valid music filename or an exception will be thrown. TODO: document these? The default value is `null` which will determine the one to use automatically based on the [Areas](../../../Enums%20and%20IDs/librarystuff/Areas.md) and [flags](../../../Flags%20arrays/flags.md).

### `map`: int

The battle map to use for the battle. This must be -1 or a valid battle map id or an exception will be thrown. Since there 73 battle maps defined, the value must be between 0 and 72 or -1 which means to automatically select it. TODO: Document these? The default value is -1.

## Remarks

The camera position is saved in the same way then [Savecamera](Savecamera.md) before starting the StartBattle coroutines with the desired parameters. It is however, not restored after the battle, but it can be restored manually later using [Loadcamera](Loadcamera.md).

From there, the message lock value is saved temporarily before forced its release. From there, the textbox is hidden with a shrink animation and a yield of 0.5 secnds. Finally, the yield until the battle is over starts. When it is over, an additional yield is done of 0.5 seconds, the message lock is restored to its original value and the current textbox is set to reveal with a grow animation.

This temporarily violates the rule that only one SetText call in [Dialogue mode](../../Dialogue%20mode.md) may run because it is now possible to have this command process in [Dialogue mode](../../Dialogue%20mode.md) while letting the battle itself call SetText in [Dialogue mode](../../Dialogue%20mode.md), but this is safe because should this happen, this command will stall SetText until the battle is over. This means that the original call wouldn't do anything that can interfere with any calls done during the battle. It is anyway necessary: BattleControl sometime needs [Dialogue mode](../../Dialogue%20mode.md) in some scripted events.
