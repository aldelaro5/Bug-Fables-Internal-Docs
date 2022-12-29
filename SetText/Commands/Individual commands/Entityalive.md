# Entityalive

Redirect if an [Entity id](../Entity%20id.md) resolves to not null and enabled, otherwise, this will not do anything and continue processing.

## Syntax

````
|entityalive,entity,redirect|
````

## Parameters

### `entity`: int

The [Entity id](../Entity%20id.md) to check for not being null and enabled. This must be a valid [Entity id](../Entity%20id.md) or an exception will be thrown.

### `redirect`: int

The [Dialogue line id](../Dialogue%20line%20id.md) to redirect to if applicable. This must be a valid [Dialogue line id](../Dialogue%20line%20id.md) or an exception will be thrown when redirecting.

## Remarks

The actual check is performed using the activeInHierarchy field of the entity's GameObject. This means that not only the [Entity](../../../Data%20format/Entity.md) needs to be active to redirect, but all of its parent too.

If a redirection happen, the input string is replaced by an [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) version of the resolved `redirect` dialogue line and processing resumes at the start of the input string to accommodate its replacement.
