# Button

Renders a ButtonSprite inline of the text and adjust the offset accordingly to resume processing after the sprite.

## Syntax

(1)

````
|button,buttonid|
````

(2) (Not compatible with [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md))

````
|button,buttonid,type|
````

(3) (Not compatible with [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md))

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
  This parameter is only used in the [Key Bindings List Type](../../../ItemList/List%20Types%20Group%20Details/Key%20Bindings%20List%20Type.md) under normal gameplay.

### `description`: string

If no value is specified, this defaults to null. This render some text immediately after the button. This parameter is not used under normal gameplay and the text does not render correctly as its rendering is not considered during offset calculations.

## Remarks

The command will first create a GameObject with a ButtonSprite which will get added to the `buts` list. This ButtonSprite will have the following properties:

* tridimentional: set to tridimensional
* parent: set to the [textholder](../../Notable%20local%20variable/textholder.md)
* localEulerAngles: Vector3.zero
  If tridimensional is false, the layer is set to 5.

The ButtonSprite is initialised like so:

* `id`: `buttonid`
* `onlyone`: `type`
* `labeltext`: `description`
* `tposition`: (current offset + o, current line + 0.225 * [size](size.md).y) where o is 0.25 or 0.55 if it is a longer button than usual
* `size`: ([size](size.md).x / 2.2, [size](size.md).y / 2.2, 1.0)
* `overridesortorder`: [Sort](Sort.md)
* `parent`: null
* `basecolor`: pure white
* `labeloffset`: Vector3.zero

After the button has been fully setup, the current offset will be increased by 0.75 (1.5 if this was a longer button than usual). Finally, if the current speed is above 0 and [Text advance](../../Related%20Systems/Text%20advance.md) is not reporting a text skipping, an additional yield will be performed using the current speed as amount of seconds. This will simulate an additional cycle of the [SetText Life Cycle > Char loop](../../SetText%20Life%20Cycle.md#char-loop).

This command has special logic in [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md). It adds 0.7 to the line accumulator when encountered and another 0.7 if the the keyboard input text is not empty and it doesn't have `Arrow`. Then, it reset the word accumulator to 0.

It should be noted that this command does not work correctly under [Single Letter Rendering](../../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md) because the offset calculations are off which means the ButtonSprite may not render on the screen or it may render at the incorrect position.

This command is accumulated for the [Backtracking](../../Related%20Systems/Backtracking.md) system.

## Known issue with spacing

While it is possible to place this command anywhere in the input string, it is strongly recommended to place it at the start of a line or after a space. This is because a known issue with the button rendering is that the render position is slightly off to the left and if this command is placed directly after some text, it will overlap said text by a slight amount. Putting a space before the command or ensuring the command is placed at the start of a line remedies this problem. Additionally, it also helps [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) because it increases the chance that the button will not overflow the line if it is placed at the end of it due to the method resetting the word width accumulator when encountering this command which would ignore the current word.

## Compatibility with [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md)

Syntax (2) and (3) are not recommended to use because while they technically work, if the linebreak requested isn't null, this will throw an exception during [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md). The auto line breaker assumes that the command will be in syntax (1) as far as parsing the `buttonid` parameter goes. It is still possible to use these syntaxes, but care must be given to make sure [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) is never called on the input string. The game uses syntax (2) only during the [Key Bindings List Type](../../../ItemList/List%20Types%20Group%20Details/Key%20Bindings%20List%20Type.md).
