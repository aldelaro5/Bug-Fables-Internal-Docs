# ItemList

Bug Fables has a way to present different kinds of lists that can be browsed with scrolling and prompt one selection, multiple incontiguous selection or cancellation from the options in the list. Such a list is called an ItemList and the current one can be accessed with the `ItemList` field (it is null when no ItemList is active). This is not to be confused with [Prompt](../SetText/Commands/Individual%20commands/Prompt.md), [NumberPrompt](../SetText/Commands/Individual%20commands/NumberPrompt.md) or [LetterPrompt](../SetText/Commands/Individual%20commands/LetterPrompt.md) which are only usable in [Dialogue mode](../SetText/Dialogue%20mode.md) in [SetText](../SetText/SetText.md) while an ItemList can be used outside of SetText 
via [ShowItemList](ShowItemList.md). Prompts also does not support scrolling as well as having other limitations in regards to rendering.

The ItemList has the ability to have a description box rendered for the currently hovered item and to have special behaviors in the pause menu since the same method is used there, just presented differently. This is all maintained through the [ItemList State Machine](ItemList%20State%20Machine.md) that MainManager is responsible to manage via mostly static fields.

To show one, ShowItemList must be called or use its SetText command counterpart, [Pickitem](../SetText/Commands/Individual%20commands/Pickitem.md). The first call resets the state machine and renders the ItemList for the first time. From this point, MainManager's Update takes over (or PauseMenu's when paused) which has special routines to handle the ItemList after it is done being processed. Whenever the cursor moves or a change of selection with `multiselect` is performed, the ItemList is refreshed which causes another call to ShowItemList, but this time, it will only update the state machine rather than clear it and rerender the ItemList if it needs to be scrolled. This continues until a confirmation or a cancellation is processed which is handled differently depending on the [listtype](listtype.md). If the ItemList came from a Pickitem command, SetText will do the final handling of it.

## Performance concerns

There are known problems with this implementation regarding performance.

The first one is no matter the conditions in which ShowItemList is called (ItemList creation or refresh), it will always regenerate the backend list of options available called `listvar` which is an array generation. While not too expensive by itself, it gets problematic due to the frequency it happens since it happens every time the user moves, scrolls or change the selection.

The second and potentially more concerning one is the method will destroy and rerender the entire visible part of the ItemList every time it is scrolled. This can quickly get expensive because it means having to do a couple of SetText calls and also dealing with the constant deallocation/allocation of objects used to render such as sprites.

These issues are known to cause noticeable lag on console such as the  Switch, but also on PC with low end hardware.

## Implementation limitations

Since this is using a state machine where most of its fields are globally accessible, it is not possible to have multiple ItemLists at once. If attempted, this will result in undefined behaviors.

## Known issues

Additionally, there has been reported issues in the past due to the sensitivity of this state machine. Since most fields are globally accessible and carried over between ShowItemList calls, it can lead to a situation where fields are carried over between different ItemLists which can lead to undefined behaviors. One such case that affects this system globally is the [inlist issue](inlist%20issue.md).
