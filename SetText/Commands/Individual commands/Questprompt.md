# Questprompt

Sets the last [Prompt](Prompt.md) as a quest prompt which needs specific handling.

## Syntax

````
|questprompt|
````

## Parameters

None.

## Remarks

When confirming an [ItemList](../../../ItemList/ItemList.md) whose type is [Quest Board List Type](../../../ItemList/List%20Types%20Group%20Details/Quest%20Board%20List%20Type.md), this will call SetText with a [Prompt](Prompt.md) command to ask if it should be taken. If it is, it will redirect like a normal [Prompt](Prompt.md) command, but this isn't enough: the actual [BoardQuests](../../../Enums%20and%20IDs/BoardQuests.md) needs to be transferred using [Activateselectedquest](Activateselectedquest.md) as well as some special handling logic that are specific to quest prompts such as showing the tutorial if applicable.

This commands allows SetText to know that it is dealing with such prompts. How it works is during the [Quest Board List Type > Confirmation handling](../../../ItemList/List%20Types%20Group%20Details/Quest%20Board%20List%20Type.md#confirmation-handling), it will call SetText with the boardcaller's dialogue line, but it will be prepended with this command. This command will be in effect for the rest of the SetText call, but it typically used only once.

If it is in effect and we have confirmed a [Prompt](Prompt.md), after obtaining the [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) version of the dialogue text to redirect, that text is appended with a string that handles the quest prompt.

If we confirmed any other option than the first one (which is the confirmation one), this string is |[Break](Break.md)\||[Loadcamera](Loadcamera.md)\||[End](End.md)\|.

If we confirmed the first option, this string starts with |[Activateselectedquest](Activateselectedquest.md)\||[Break](Break.md)\|. If [flags](../../../Flags%20arrays/flags.md) 64 is false (received the quest board tutorial), this is also added to it: |[Tail](Tail.md),null||[Blank](Blank.md)\||[Boxstyle](Boxstyle.md),4| followed by menutext 170 (the quest board tutorial) followed by |[Flag](Flag.md),64,true||[Break](Break.md)\|. Finally, this last portion is added in either cases: |[Loadcamera](Loadcamera.md)\||[End](End.md)\|.
