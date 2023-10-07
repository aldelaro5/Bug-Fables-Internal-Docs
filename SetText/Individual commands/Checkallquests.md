# Checkallquests

Redirect if 60 or more [BoardQuests](../../Enums%20and%20IDs/BoardQuests.md) are in the completed board or do nothing otherwise.

## Syntax

````
|checkallquests,redirect|
````

## Parameters

### `redirect`: int

The [Dialogue line id](../Common%20commands%20id%20schemes/Dialogue%20line%20id.md) to redirect if applicable. This must be a valid [Dialogue line id](../Common%20commands%20id%20schemes/Dialogue%20line%20id.md) or an exception will be thrown.

## Remarks

If a redirection happen, the input string is replaced by an [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) version of the resolved `redirect` dialogue line and processing resumes at the start of the input string to accommodate its replacement.
