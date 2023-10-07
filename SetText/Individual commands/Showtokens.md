# Showtokens

Create or destroy a Game Tokens counter HUD element.

## Syntax

(1) (Destroy the existing HUD element if any before creating it)

````
|showtokens|
````

(1) (To destroy the HUD element if it exists)

````
|showtokens,null|
````

## Parameters

### `null`

This parameter indicates to operate in syntax (2) which will destroy the HUD element if it exists (this command does nothing if it doesn't exist). Any other value will cause this command to do nothing.

## Remarks

Unlike [showmoney](Showmoney.md), this doesn't just show or hide the counter, it creates or destroy it inside the command.

The HUD element is creating with a new UI object named `TokenBar` parented to the GUICamera with size (0.55, 0.6, 1.0) using the gui sprite 4, a sort order of 0 and a color of E57F00 (mostly orange). Its position depends on if the berries counter HUD element is shown or not. If it's hidden, the position is (15.0, -4.25, 10.0) and if it's shown, it's (15.0, -3.0, 10.0).

The counter is rendered using a SetText call where the object created earlier is the parent in [non Dialogue mode](../Dialogue%20mode.md#non-dialogue-mode) with the input string being |[sort](Sort.md),10||[color](Color.md),4||[dropshadow](Dropshadow.md),0.1,-0.1|" appended with the value of [flagvar](../../Flags%20arrays/flagvar.md) 27 padded with to 4 characters with 0s:

* [fonttype](../Notable%20states.md#fonttype.md): UNUSED ???
* No `linebreak`
* No `tridimensional`
* `position` of (-0.75, -0.4, 0.0)
* No `camoffset`
* `size` of (1.75, 1.75)
* No `caller`

For the icons, it is a new UI object named `icon` parented to the HUD element created earlier with a position of (-2.25, 0.1, 0.0), a size of (2.0, 2.0, 2.0) using the Game Tokens [Item](../../Enums%20and%20IDs/Items.md) sprite with a sort order of 1.

Finally, a DialogueAnim is added to `TokenBar` with a targetpos of (7.0, `y`, 10.0) where `y` is where the the `TokenBar`'s y position, a targetscale of (0.55, 0.6, 1.0) and a speed of 0.4.
