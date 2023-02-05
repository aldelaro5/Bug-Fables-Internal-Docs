# Minibubble

Show some text inside a Minibubble by a [Dialogue line id](../Dialogue%20line%20id.md) or by directly specifying the text which will have its tail set to a specified entity. The Minibubble's lifecycle can then be controlled from the same SetText call using [Waitminibubble](Waitminibubble.md) and [Destroyminibubble](Destroyminibubble.md).

## Syntax

(1) (Only tells SetText that we are in control of an existing Minibubble, see the remarks section for more details)

````
|minibubble|
````

(2)

````
|minibubble,text,entity|
````

(3)

````
|minibubble,text,entity,yposition|
````

(4)

````
|minibubble,text,entity,xposition,yposition|
````

## Parameters

### `text`: int | `@`string

The input string of the SetText call that will show in the Minibubble. The int form specify the [Dialogue line id](../Dialogue%20line%20id.md) to use while the string form specify the text directly. The int form must be a valid [Dialogue line id](../Dialogue%20line%20id.md) or an exception will be thrown. The string form must start with `@` otherwise it will be interpreted as an int which will cause an exception to be thrown.

The string form supports having commands without interfering the main parser. To put commands in it, every `|` must be `{` and every `,` must be `}`. Failure to do so will cause undefined behaviors with the command parser which will likely result in an exception being thrown.

### `entity`: `caller` | `this` | int

The [Entity data](../../../TextAsset%20Data/Entity%20data.md) that the Minibubble's tail will point to. The int form refers to an [Entity id](../Entity%20id.md) which must be a valid int or an exception will be thrown. This form is valid no matter if caller is null or not. If the caller isn't null however, additional values are possible:

* `this`: Refers to [tailtarget](../../Notable%20local%20variable/tailtarget.md)'s entity (This only reliably works in [Dialogue mode](../../Dialogue%20mode.md) otherwise, it will take the last value of [tailtarget](../../Notable%20local%20variable/tailtarget.md) which is potentially undefined behavior) Since it is setting the [tailtarget](../../Notable%20local%20variable/tailtarget.md) to its current entity, this effectively doesn't change it.
* `caller`: Refers to the caller's entity.

Any non int value that isn't in one of the predefined values above will be interpreted as an int which will cause an exception to be thrown. Additionally,  if the caller is null, the int form must be specified as any other value will be interpreted as an int which will cause an exception to be thrown.

If the entity resolves to null, this command will do nothing.

This parameter is completely ignored in [Testdiag](Testdiag.md), but must still be specified to any value in syntax (3) and (4).

### `xposition`: float

The x position of the Minibubble relative to the GUICamera. This must be a valid float or an exception will be thrown. The default value is calculated using  the `entity`'s viewportPoint.x which represents a value between 0 and 1 that indicates where is `entity` rendered on the screen horizontally (0 is leftmost, 1 is rightmost). This is brought to the range between -0.5 and 1.5 and then multiplied by 33.0 all clamped from -5.0 to 5.0.

### `yposition`: float

The y position of the Minibubble relative to the GUICamera. This must be a valid float or an exception will be thrown. The default value is 0.85.

## Remarks

The Minibubble will be setup using the text determined from `text`, the entity determined from `entity`, the position determined from `xposition` and `yposition` with a z component of 10.0, a sortingOrder of 10 + 3 * the amount of active Minibubbles and a wait time of 1.5 seconds.

### Minibubble's SetUp

When the Minibubble is setup, this will create a GameObject that has a Minibubble component and it will be stored in the SetText's bubbles list. This game object will have a name of `MiniBubble-` followed by the final text. The actual text gets prepanded with |minibubble||[Sort](Sort.md),sortingOrder + 1| where sortingOrder is the value calculated from above. The minibubble command uses syntax (1) which is important because it allows SetText to know it is in control of an existing Minibubble. The text also gets appended with |[Fwait](Fwait.md),1.5| where 1.5 is not configurable via this command. The final fields of the component are as follows:

* `basesort`: The sortingOrder
* `text`: The final text determined above
* `pos`: The final position determined above
* `target`: The entity determined above

### Minibubble's Start

The component will setup 3 UI GameObjects:

* `Bubble`: The Minibubble sprite. This is childed to the Minibubble component with a position of Vector3.zero and a size of (0.8, 0.65, 1.0). The sortingOrder of the SpriteRenderer is set to basesort - 1 with a color of E5D8BF (a similar color to the main textbox color).
* `Tail`: The tail sprite that will point to the target. This is childed to the Minibubble component with a position of Vector3.zero and a size of (0.75, 0.75, 0.75). The sortingOrder of the SpriteRenderer is set to basesort with a color of E5D8BF (a similar color to the main textbox color). This object is stored in the `tail` field of the component.
* `Tailback`: The background of the `Tail` sprite. This is childed to the `tail` with a position of Vector3.zero and a size of (1.2, 1.2, 1.2). The sortingOrder of the SpriteRenderer is set to basesort - 2 with a color of pure black.

The parent of the transform is then set to the GUICamera with a localPosition of `pos` and without any scaling. A dialogueAnim is added to the component in the `anim` field with a shrinkspeed of 0.3 which effectively makes the shrink animation of the Minibubble take 30% of the time of the default time that textbox uses.

Finally, the SetText call is done in non [Dialogue mode](../../Dialogue%20mode.md) with the `text` field as the input string:

* `fonttype`: `BubblegumSans`
* `linebreak`: 3.5 in `English` or 3.25 on any other language (this is to workaround the broken auto line breaker, see [OrganiseLines Known Issues > Not counting a whole word's width after the first line](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines%20Known%20Issues.md#not-counting-a-whole-word-s-width-after-the-first-line) for more details on why)
* No `tridimensional`
* `position`: (-1.55, 0.15)
* No `cameraoffset`
* `size`: (0.75, 0.75)
* `parent`: The Minibubble's transform
* `caller`: null

### Minibubble's LateUpdate

While the Minibubble is active, the `tail` will track the position of `target` by dynamically changing its angle (max 45 degrees) and localPosition depending on its horizontal location on the screen. Additionally, each lateUpdate will have the `target`'s entity set to be talking. If the `target` goes null, the `tail` will be disabled.

### Minibubble's destruction

Syntax (1) is a special syntax that doesn't create a Minibubble, but instead tells SetText that we are in control of an existing one created by the parent. This is how the inner SetText call can track the Minibubble created by the outer SetText call. This means that this command should not be used in syntax (1) unless the parent has a Minibubble component.

Once the inner SetText call is done and it has reached the [SetText Life Cycle > Cleanup](../../SetText%20Life%20Cycle.md#cleanup) phase, the Minibubble if it exists will get automatically destroyed due to the parameterless command specified above being prepended to the input string. This destruction not only destroys the GameObject (in 1 second to let the shrink animation play) and its Minibubble component, but it also sets the `target` (if it exists) to no longer be talking.

### Managing Minibbubles from the outer SetText call

It is possible however to preemptively destroy a Minibubble using the [Destroyminibubble](Destroyminibubble.md) command. It is meant to be used in the outer SetText call, but while it can be used in the inner call, this is not recommended as it can lead to undefined behaviors.

It is also possible to wait for wait for a Minibubble to complete from the outer call using a [Waitminibubble](Waitminibubble.md) command. This will yield until the Minibubble goes null which only happens shortly after its destruction.

### A note about [Halt](Halt.md)

The [Halt](Halt.md) command has a special use here: it can be used to let the outer call decide when the destruction of the Minibubble should occur, even after the 1.5 seconds default time. This can be done by placing a [Halt](Halt.md) command at the end of `text` which will cause the inner call to stall indefinitely and thus prevents the destruction of the Minibubble. While it prevents the inner call to end, it can be ended by force from the outer call by using the [Destroyminibubble](Destroyminibubble.md) command mentioned above. Since the inner called is owned by the Minibubble GameObject, destroying it will stop the coroutine with it. This is safe to do because the inner call isn't in [Dialogue mode](../../Dialogue%20mode.md) which would have softlocked the game if it was.

### Special behaviors when in control of a Minibubble

As syntax (1) of the inner call is done, this also changes the [SetText Life Cycle > Letter processing](../../SetText%20Life%20Cycle.md#letter-processing) behavior: it will no longer consider [Text advance](../../Related%20Systems/Text%20advance.md) state and [Dialogue mode](../../Dialogue%20mode.md) to decide whether to yield between each iteration of the [SetText Life Cycle > Char loop](../../SetText%20Life%20Cycle.md#char-loop). Even if the [Speed](Speed.md) is set to 0, it will still do a yield of 0 seconds. Additionally, the same happens with the [Wait](Wait.md) command: it will always cause the yield to occur regardless of the [Text advance](../../Related%20Systems/Text%20advance.md) state.

### Additional remarks

This is one of the command supported by [Testdiag](Testdiag.md) and it has special behaviors in that mode where `entity` is ignored and the player's entity is used instead.
