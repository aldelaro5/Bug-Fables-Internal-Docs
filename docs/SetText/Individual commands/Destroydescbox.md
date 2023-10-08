# Destroydescbox

Destroy the caller's description window if it exist no matter what or only do it if the caller's interaction type isn't Shop or CaravanBadge.

## Syntax

(1)

````
|destroydescbox,soundname|
````

(2)

````
|destroydescbox,1|
````

## Parameters

### `1`

Whether to ignore the caller's interact type and attempt to destroy the description window anyway. Any other value is ignored and treated like syntax (1). The caller must not be null when sending this parameter or an exception will be thrown.

## Remarks

Syntax (1) will destroy the caller's description window in 0.5 seconds with a shrink animation if it exists and its interact type is anything other than Shop or CaravanBadge. Syntax (2) ignores all of this and tries to destroy it anyway, even if the caller is null which will throw an exception.

In the case where the caller has no description window or syntax (1) is used without meeting the conditions, this command does nothing.

This command is only used in very specific cases such as grabbing an item, during the course of [giveitem](Giveitem.md) and when regenerating the list of medals available at Merab's shop or Shades's shop.
