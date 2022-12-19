# Entry point

Calling [SetText](SetText.md) requires running a coroutine called `SetText` that is static to `MainManager`. There are 4 overloads available.

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

The input string to process. Must contains all the [Commands](Commands/Commands.md) inline. All CRLF are replaced by LF to make the line endings consistent. A null value may either as if it was an empty string or cause undefined behaviors.

### [fonttype](fonttype.md)

The font to use. If `BubblegumSans`, overridden to `Uzura` when the [languageid](languageid.md) is `German` and to `ONEMobilePOP` when it is `Korean`.

### `linebreak`

If not null, the input string will be its [OrganiseLines](Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) version of it passing the value as the `maxoffset`.

### `dialogue`

Tells SetText to operate in [Dialogue mode](Dialogue%20mode.md). If false, a `|Spd,0|` is prepended to the input string which will cause all wait times to be disabled by default. Refer to the [Dialogue mode](Dialogue%20mode.md) document for more information.

### `tridimensional`

Seems to indicate if the [textholder](Notable%20local%20variable/textholder.md) can be angled? Used in the [Button](Commands/Individual%20commands/Button.md) command and can influence the letter's layer in the character loop?

### `position`

Indicates the localPosition of the [textholder](Notable%20local%20variable/textholder.md). This is relative to [Parent](Commands/Individual%20commands/Parent.md) in non [Dialogue mode](Dialogue%20mode.md) and relative to the [textbox](Notable%20local%20variable/textbox.md) in [Dialogue mode](Dialogue%20mode.md) which is positioned at 0.0, 3.25, 10.0 relative to the `GUICamera`. If this is effectively a zero vector, the localPosition of the [textholder](Notable%20local%20variable/textholder.md) defaults to (-5.5, 0.9, 0.0).

### `cameraoffset`

The vector to add to MainManager.instance.camoffset (TODO) in [Dialogue mode](Dialogue%20mode.md). This isn't used in non [Dialogue mode](Dialogue%20mode.md).

### [size](Commands/Individual%20commands/size.md)

The size scaling to apply to each letter from the start. This can be overridden during \[\[life cycle#` ` proocessing\]\] with a [size](Commands/Individual%20commands/size.md) command.

### [Parent](Commands/Individual%20commands/Parent.md)

The parent of the [textholder](Notable%20local%20variable/textholder.md) in non [Dialogue mode](Dialogue%20mode.md). In [Dialogue mode](Dialogue%20mode.md), this represents who is talking at the start and will have the [tailtarget](Notable%20local%20variable/tailtarget.md) set to it. This can be overridden with a [Tail](Commands/Individual%20commands/Tail.md) command.

### `caller`

Represent who is the recipient of the dialogue in [Dialogue mode](Dialogue%20mode.md) at the start. This can be overridden with a [Parent](Commands/Individual%20commands/Parent.md) command.

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
