# [ItemList](ItemList.md) State Machine

There are several static public fields of MainManager that represents the state of an [ItemList](ItemList.md). All of them can be accessed externally, but extra care must be given in doing so since it can lead to undefined behaviors. The fields are generally kept around between [ShowItemList](ShowItemList.md) calls to preserve them between updates and only changed before doing a refresh.

To denote that we are currently in an [ItemList](ItemList.md), `ItemList` gets assigned when the list gets active and it gets set to null when inactive. Additionally, the instance field `inlist` is set to true on the list creation and it is supposed to be set to false when it's handled, but due to the [inlist issue](inlist%20issue.md), this is only guaranteed after a [Pickitem](../SetText/Commands/Individual%20commands/Pickitem.md) command is fully done. This is different than `prompt` which tells if we are in a [Prompt](../SetText/Commands/Individual%20commands/Prompt.md), [NumberPrompt](../SetText/Commands/Individual%20commands/NumberPrompt.md) or [LetterPrompt](../SetText/Commands/Individual%20commands/LetterPrompt.md) and it is managed differently.

All fields are static to MainManager unless specified otherwise.

## Parameters of [ShowItemList](ShowItemList.md)

These are sent to the method and they are kept in these fields so that the method will be called the same way during refreshes:

* [listtype](listtype.md): The original `type` parameter. It does not take into consideration overrides. Each type specify different methods for generation `listvar`, the [SetText](../SetText/SetText.md) input string used for each options and how its `listdescbox` is rendered.
* `listpos`: The original `position` parameter.
* `listdesc`: The original `showdescription` parameter.
* `listsell`: The original `sell` parameter.

## Configuration

These are not sent to [ShowItemList](ShowItemList.md), but should be set externally before to configure aspects of the list prompt:

* `listredirect`: The [Dialogue line id](../SetText/Commands/Dialogue%20line%20id.md) when a confirmation is done (see [Pickitem](../SetText/Commands/Individual%20commands/Pickitem.md) for more info)
* `listcancel`: The [Dialogue line id](../SetText/Commands/Dialogue%20line%20id.md) to redirect when a cancellation is done (see [Pickitem](../SetText/Commands/Individual%20commands/Pickitem.md) for more info)
* `storeid`: The store id if it's one.
* instance.`listammount`: The amount of items shown at once.

## Save and restore

The list prompt system has the ability to save the browsing information on an [ItemList](ItemList.md) after confirmation (but not cancellation) and to restore them the next time a new [ItemList](ItemList.md) is requested via [ShowItemList](ShowItemList.md):

* `savelastlist`: Tell whether or not to save the browsing info upon confirmation to `overridedlist`. Has to be set externally and gets cleared once the list has been saved.
* `overridedlist`: The browsing info of the last saved list by `savelastlist`. Gets cleared on restore to the next new list call. It contains `option`, `listcursor`, `listlow`, `listmax` and `listoption`.

This is only used in CardGame under normal gameplay.

## Options management

These are used to track a list prompt's options and selections:

* `multiselect`: The list of selected items in the [ItemList](ItemList.md)
* `listcanceled`: Whether the [ItemList](ItemList.md) has been cancelled or not
* `listvar`: All the available options, depends on [listtype](listtype.md)
* instance.`maxoptions`: The number of available options in the entire list (same use as in a [Prompt](../SetText/Commands/Individual%20commands/Prompt.md), [NumberPrompt](../SetText/Commands/Individual%20commands/NumberPrompt.md) or [LetterPrompt](../SetText/Commands/Individual%20commands/LetterPrompt.md)).
* instance.`option`: The option index in the entire list (same use as in a [Prompt](../SetText/Commands/Individual%20commands/Prompt.md), [NumberPrompt](../SetText/Commands/Individual%20commands/NumberPrompt.md) or [LetterPrompt](../SetText/Commands/Individual%20commands/LetterPrompt.md)).
* `listoption`: The same then option, but only after the [ItemList](ItemList.md) destruction (which is done after a selection has been done).

## Scrolling and view

These are used to maintain a separation between what the entire list is and what its visible part is located at:

* `listlow`: Lower boundary of the visible part of the [ItemList](ItemList.md) (this is visually the topmost item visible)
* `listmax`: Upper boundary of the visible part of the [ItemList](ItemList.md) (this is visually the bottomost item visible)
* `listy`: The `listlow` or -1. This is how the game detects that an [ItemList](ItemList.md) isn't fully scrolled by putting the value to -1 and then to `listlow` after fully scrolling it
* `listcursor`: The option index of the currently visible part only (from 0 to `listammount` - 1)

## User interface

These are the actual objects that contains all the UI for the [ItemList](ItemList.md):

* [ItemList](ItemList.md): The transform of the current [ItemList](ItemList.md), also tells if an [ItemList](ItemList.md) is active if it's not null
* `listdescbox`: The transform of the description box
