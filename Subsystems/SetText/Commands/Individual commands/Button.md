# Button

Renders a ButtonSprite inline of the text and adjust the offset accordingly to resume processing after the sprite.

## Syntax

(1)

````
|button,buttonid|
````

(2)

````
|button,buttonid,type|
````

(3)

````
|button,buttonid,type,description|
````

## Parameters

### `buttonid`: int

The button id to render. This must be a valid integer or an exception will be thrown. If it is not a valid button id, this is undefined behavior which may lead to an exception being thrown. Here are the different button ids available:
id | name
---------- | ---------
0 | Up
1 | Down
2 | Left
3 | Right
4 | Confirm / Jump / Interact
5 | Cancel / Field Action
6 | Scroll Faster / Switch Party
7 | Show / Hide HUD
8 | Pause
9 | Help

### `type`: int

Determines how to render between keyboard input If no value is specified, this defaults to -1. This must be a valid integer or an exception will be thrown. Here are the different behaviors:

* -1: Automatically determine which input to render based on the current control scheme
* 
   > 
   > = 1: Always render the controller input

* Anything else: Always render the keyboard input
  This parameter is not used under normal gameplay.

### `description`: string

If no value is specified, this defaults to null. This render some text immediately after the button. This parameter is not used under normal gameplay and the text does not render correctly as its rendering is not considered during offset calculations.

## Remarks

The command will first create a GameObject with a ButtonSprite which will get added to the `buts` list. This ButtonSprite will have the following properties:

* tridimentional: set to `tridimensional`
* parent: set to the [textholder](../../Notable%20local%20variable/textholder.md)
* localEulerAngles: Vector3.zero
  If `tridimensional` is false, the layer is set to 5.

The ButtonSprite is initialised like so:

* `id`: `buttonid`
* `onlyone`: `type`
* `labeltext`: `description`
* `tposition`: (`currentoffset` + o, `currentline` + 0.225 * `Size`.y) where o is 0.25 or 0.55 if it is a longer button than usual
* `size`: (`Size`.x / 2.2, `Size`.y / 2.2, 1.0)
* `overridesortorder`: [Sort](Sort.md)
* `parent`: null
* `basecolor`: pure white
* `labeloffset`: Vector3.zero

After the button has been fully setup, the `currentoffset` will be increased by 0.75 (1.5 if this was a longer button than usual). Finally, if the current speed is above 0 and [Text advance](../../Related%20Systems/Text%20advance.md) is not reporting a text skipping, an additional yield will be performed using the current speed as amount of seconds. This will simulate an additional cycle of the [life cycle > Char loop](../../life%20cycle.md#char-loop).

It should be noted that this command does not work correctly under [Single](Single.md) because the offset calculations are off which means the ButtonSprite may not render on the screen or it may render at the incorrect position.

This command is accumulated for the [Backtracking](../../Related%20Systems/Backtracking.md) system.
