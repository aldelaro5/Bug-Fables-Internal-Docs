# SetText Entry Points

Calling [SetText](SetText.md) requires running a Coroutine called `SetText` that is static to MainManager. There are 4 overloads available, but all of them ends up at the (4) one.

# Signatures

(1)

````cs
public static IEnumerator SetText(string text, Vector3 position, Transform parent)
````

(2)

````cs
public static IEnumerator SetText(string text, Transform parent, NPCControl caller)
````

(3)

````cs
public static IEnumerator SetText(string text, bool dialogue, Vector3 position, Transform parent, NPCControl caller)
````

(4)

````cs
public static IEnumerator SetText(string text, int fonttype, float? linebreak, bool dialogue, bool tridimensional, Vector3 position, Vector3 cameraoffset, Vector2 size, Transform parent, NPCControl caller)
````

## Parameters

### `text`

The starting input string to process. Must contains all the [Commands](Commands.md) inline. All CRLF (`\r\n`) are replaced by LF (`\n`) to make the line endings consistent. A null value may act either as if it was an empty string or cause undefined behaviors.

This may not be the only text that gets processed if the [Commands](Commands.md) in it replaces their text or if one of them perform a redirection which replaces the whole string.

### `fonttype`

The starting font to use. This can be overridden by the [Setup](Life%20Cycle.md#setup) and [Letter processing](Life%20Cycle.md#letter-processing) phases or the [font](Individual%20commands/Font.md) command.

### `linebreak`

The max width of a line enforced by [OrganiseLines](Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md). If not null, the starting input string will be replaced by its [OrganiseLines](Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) version as well as any redirected strings passing this parameter as the `maxoffset`. If it's null, this will not run [OrganiseLines](Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) on the starting string or redirected strings.

### `dialogue`

Tells SetText to operate in [Dialogue mode](Dialogue%20mode.md). If false, an |[spd](Individual%20commands/Spd.md),0| is prepended to the input string which will cause all wait times to be disabled by default. Refer to the [Dialogue mode](Dialogue%20mode.md) document for more information.

### `tridimensional`

Render the text using layer 0 (Default) instead of layer 5 (UI) which renders the text using the main camera instead of GUICamera. This is only supported in [Regular Letter Rendering](Letter%20Rendering%20Methods/Regular%20Letter%20Rendering.md), this doesn't do anything meaningful in [Single Letter Rendering](Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md).

This is only used in narrow cases, for most cases, [triui](Individual%20commands/Triui.md) is a better option.

### `position`

Indicates the localPosition of the [textholder](Notable%20states.md#textholder). This is relative to the `parent` parameter in [non dialogue mode](Dialogue%20mode.md#non-dialogue-mode) and relative to the [textbox](Notable%20states.md#textbox) in [Dialogue mode](Dialogue%20mode.md) which is positioned at 0.0, 3.25, 10.0 relative to the GUICamera. If this is effectively a zero vector, the localPosition of the [textholder](Notable%20states.md#textholder) defaults to (-5.5, 0.9, 0.0).

### `cameraoffset`

The vector to add to `camoffset` in [Dialogue mode](Dialogue%20mode.md) which offsets the camera's position. This is ignored  in [non dialogue mode](Dialogue%20mode.md#non-dialogue-mode).

### `size`

The starting size scaling to apply to each letter from the start. This can be overridden with a [size](Individual%20commands/size.md) command.

### `Parent`

The parent of the [textholder](Notable%20states.md#textholder) in [non dialogue mode](Dialogue%20mode.md#non-dialogue-mode). In [Dialogue mode](Dialogue%20mode.md), this represents who is talking at the start and will have the [tailtarget](Notable%20states.md#tailtarget) set to it. This can be overridden with a [tail](Individual%20commands/Tail.md) command.

### `caller`

Represent who is the recipient of the dialogue in [Dialogue mode](Dialogue%20mode.md) at the start. This can be overridden with a [parent](Individual%20commands/Parent.md) command.

## Remarks

Every SetText call ends up at (4) with the other 3 being a simplified call to (4). Specifically:

(1): starts (4) with

````cs
SetText(text, 0, null, false, false, position, Vector3.zero, Vector2.one, parent, null)
````

Then yields for a frame.

(2): starts (4) with

````cs
SetText(text, 0, messagebreak, true, false, Vector3.zero, Vector3.zero, Vector2.one, parent, caller)
````

Then yields for a frame.

(3): Sets `ll` to MainManger.messagebreak if MainManager.dialogue, `null` otherwise. Then, starts (4) with

````cs
SetText(text, 0, ll, dialogue, false, position, Vector3.zero, Vector2.one, parent, caller)
````

Then yields for a frame.

(4): Does essentially all the work of the text processing. This is the main SetText coroutine.
