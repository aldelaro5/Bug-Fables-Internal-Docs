# Icon

Render a gui sprite with a configurable size multiplier and sort value.

## Syntax

(1)

````
|icon,spriteindex|
````

(2)

````
|icon,spriteindex,size|
````

(3)

````
|icon,spriteindex,size,sort|
````

## Parameters

### `spriteindex`: int

The sprite index of the `Assets/Resources/sprites/gui` file to use. This must be a valid sprite index or an exception will be thrown. 

TODO: Possibly document these?

### `size`: float

The multiplier to use for the size vector of the sprite which is normally set to [size](size.md). This must be a valid float or an exception will be thrown. The default value is 1.0.

### `sort`: int

The sortingOrder value to use when rendering the sprite. This must be a valid int or an exception will be thrown. The default value is [Sort](Sort.md).

## Remarks

This first creates the sprite childed to the [textholder](../../Notable%20local%20variable/textholder.md) named after `spriteindex` prepanded with `icon` at the current offset/line with the resolved size and sort. The local position is adjusted so sprite.pivot / (sprite.pixelsPerUnit * 2.0) is added to it (see issue section below, this isn't correct). If the sprite is index 24 (the hp icon), (-0.1, -0.25) is added to the local position too.

The tag of the sprite is set to `Letter` and it is added to the same list than the one [Button](Button.md) uses. The current offset/line are then adjusted to be after the icon. Then, the font effects are set so [Glitchy](Glitchy.md) and [FadeIn](FadeIn.md) are forced to be disabled. The rotation effect is enabled if the sprite is index 146 (the Mothfly speaking scribble) on top of that.

If in [Dialogue mode](../../Dialogue%20mode.md), the [tailtarget](../../Notable%20local%20variable/tailtarget.md) is set to be talking at the end followed by a bleep play and a yield of the current [Speed](Speed.md) if that speed is higher than 0 and there is no [Text advance](../../Related%20Systems/Text%20advance.md)'s skiptext.

It should be noted that this command does not work correctly under [single letter rendering](../../Life%20Cycle/letter%20rendering/single%20letter%20rendering.md) because the offset calculations are off which means the sprite may not render on the screen or it may render at the incorrect position. There is still a mitigation in place where the text of this command is replaced by `\t` and the char loop is set to resume at it, but the space gap is too small and it doesn't solve the rendering position of the sprite.

This command is accumulated for the [Backtracking](../../Related%20Systems/Backtracking.md) system.

This is one of the command supported by [Testdiag](Testdiag.md).

### Issue with the sprite position adjustment

When rendering to the current offset/line, the sprite is going to get rendered there at its pivot point, but this is wrong because that point is too far left and down for the sprite to render correctly.

The game tries to correct it by adding a vector that is the pivot point (its center) of the sprite divided by double the amount of pixels per unit. The issue is that this math is wrong: the correct math is to add half the width in local space horizontally and a quarter of the height in local space vertically. 

To get half the width/height in local space, all that's needed is to get the x or y of the sprite's pivot point and divide it by the amount of pixels per unit which is something the sprite knows. To get the quarter height/width, simply divide this by 2. However, this isn't enough: this will give the distance between the border and the center AT NO SCALE which is often larger than desired (hence the `size` parameter of this command). To fix this, it's possible to multiply the final factor by the localScale of the sprite.

The result is a sprite that will be offset by half its width to the right and a quarter of its width up which fixes the original offset differences.

This command might still work for some sprites with the broken math, but it is important to keep in mind that not all sprites work due to this.
