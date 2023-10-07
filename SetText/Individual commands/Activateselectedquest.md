# Activateselectedquest

Mark a specific [BoardQuest](../../Enums%20and%20IDs/BoardQuests.md) or the one whose id is in [flagvar](../../Flags%20arrays/flagvar.md) 0 as taken which removes it from the open board and adds it to the taken board.

## Syntax

(1)

````
|activateselectedquest|
````

(2)

````
|activateselectedquest,quest|
````

## Parameters

### `quest`: int | `v`int | `var`int | `money`

The [BoardQuest](../../Enums%20and%20IDs/BoardQuests.md) id to mark as taken. The int form specifies the id directly while either string forms specifies the [flagvar](../../Flags%20arrays/flagvar.md) slot used to get the id. The int form must be a valid [BoardQuest](../../Enums%20and%20IDs/BoardQuests.md) or an exception will be thrown. 

Either string forms are equivalent, but they must contain `v` or `var` once or they will be interpreted as the int form which will cause an exception to be thrown. Additionally, the value inside of them must be a valid [flagvar](../../Flags%20arrays/flagvar.md) slot or an exception will be thrown. Finally, that [flagvar](../../Flags%20arrays/flagvar.md) slot must also contain a valid [BoardQuest](../../Enums%20and%20IDs/BoardQuests.md) id or an exception will be thrown.

Specifying `money` is technically possible, but is not recommended because it will interpret the current berry count as a [BoardQuest](../../Enums%20and%20IDs/BoardQuests.md) id which will likely result in an exception being thrown.

## Remarks

Syntax (1) is meant to be used in conjunction with the [Quest Board List Type](../../ItemList/List%20Types%20Group%20Details/Quest%20Board%20List%20Type.md) because [flagvar](../../Flags%20arrays/flagvar.md) 0 is where the selected [BoardQuests](../../Enums%20and%20IDs/BoardQuests.md) is located upon confirmation. For more details, check [Quest Board List Type > Confirmation handling](../../ItemList/List%20Types%20Group%20Details/Quest%20Board%20List%20Type.md#confirmation-handling). This syntax is actually sent by SetText itself when a [questprompt](Questprompt.md) is confirmed as part of the normal quest prompting process.

This command also makes sure that the open board has the `NoQuest` element if there are no more open quests after the transfer.

This also changes some flags:

* The taken [flag](../../Flags%20arrays/flags.md) slot of the [BoardQuest](../../Enums%20and%20IDs/BoardQuests.md) from boardquestdata is set to true if it exists (-1 denotes that)
* [flagstring](../../Flags%20arrays/flagstring.md) 11 is set to all open [BoardQuests](../../Enums%20and%20IDs/BoardQuests.md) id after the transfer separated by `,`
* [flags](../../Flags%20arrays/flags.md) 2 is set to true
