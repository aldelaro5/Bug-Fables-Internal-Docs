# Checkanim

Redirect to a different dialogue line if an [Entity](../../../Entities/Entity.md)'s [animstate](../../../Entities/EntityControl/Animations/animstate.md) is a specific value.

## Syntax

````
|checkanim,entity,animstate,redirect|
````

## Parameters

### `entity`: int | `this` | `caller` | string

The [Entity id](../Entity%20id.md) or designator to check the [animstate](../../../Entities/EntityControl/Animations/animstate.md). The int form represents an [Entity id](../Entity%20id.md) and it must be a valid int or an exception will be thrown. If the entity resolves to null, an exception will be thrown. This also supports other values:

* `this`: Refers to the [tailtarget](../../Notable%20local%20variable/tailtarget.md).
* `caller`: Refers to the caller.
* Anything else: Refers to a [Define](Define.md) if it exists, otherwise, this is interpreted as a regular [Entity id](../Entity%20id.md) which will cause an exception to be thrown.

### `animstate`: int | `!`int

The [animstate](../../../Entities/EntityControl/Animations/animstate.md) to check for equality that `entity` has. The int portion must be a valid int or an exception will be thrown. The `!` prefix indicates to always redirect (NOTE: it is likely a bug because it is safe to assume it was supposed to invert the condition, but it instead always redirect due to the command's implementation).

### `redirect`: int

The [Dialogue line id](../Dialogue%20line%20id.md) to redirect if applicable. This must be a valid [Dialogue line id](../Dialogue%20line%20id.md) or an exception will be thrown.

## Remarks

This command does nothing if the caller is null.

If a redirect happens, the input string is overwritten with an [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) version of the `redirect` line and processing resumes at the start of the input string to accommodate its replacement.
