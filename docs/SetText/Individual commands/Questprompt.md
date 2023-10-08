# Questprompt

Sets the last [prompt](Prompt.md) as a quest prompt which needs specific handling.

## Syntax

````
|questprompt|
````

## Parameters

None.

## Remarks

When confirming an [ItemList](../../ItemList/ItemList.md) whose type is [Quest Board List Type](../../ItemList/List%20Types%20Group%20Details/Quest%20Board%20List%20Type.md), this will call SetText with a [prompt](Prompt.md) command to ask if it should be taken. If it is, it will redirect like a normal [prompt](Prompt.md) command, but this isn't enough: the actual [BoardQuests](../../Enums%20and%20IDs/BoardQuests.md) needs to be transferred using [activateselectedquest](Activateselectedquest.md) as well as some special handling logic that are specific to quest prompts such as showing the tutorial if applicable.

This commands allows SetText to know that it is dealing with such prompts. How it works is during the [Quest Board List Type > Confirmation handling](../../ItemList/List%20Types%20Group%20Details/Quest%20Board%20List%20Type.md#confirmation-handling), it will call SetText with the boardcaller's dialogue line, but it will be prepended with this command. This command will be in effect for the rest of the SetText call, but it is typically used only once. The effect will only be taken into account during [Dialogue post-processing](../Life%20Cycle.md#dialogue-post-processing).

If it is in effect and we have confirmed a [prompt](Prompt.md), after obtaining the [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) version of the dialogue text to redirect, that text is appended with a string that handles the quest prompt.

If we confirmed any other option than the first one (which is the Yes one), this string is |[break](Break.md)\||[loadcamera](Loadcamera.md)\||[end](End.md)\|.

If we confirmed the first option (Yes), this string starts with |[activateselectedquest](Activateselectedquest.md)\||[break](Break.md)\|. If [flag](../../Flags%20arrays/flags.md) 64 is false (received the quest board tutorial), this is also added to it: |[tail](Tail.md),null||[blank](Blank.md)\||[boxstyle](Boxstyle.md),4| followed by menutext 170 (the quest board tutorial) followed by |[flag](Flag.md),64,true||[break](Break.md)\|. Finally, this last portion is added in either cases: |[loadcamera](Loadcamera.md)\||[end](End.md)\|.
