# Notable states
[SetText](SetText.md) contains some key fields and local variables that are important to understand. They especially have to do with [Dialogue mode](Dialogue%20mode.md). 

## message

A global flag on MainManager behaving like a lock that enforces the restriction that only one Dialogue mode call can execute at a given time. It is used to prevent behaviors to take place until its value is set to false. In Dialogue mode, this lock is set to true on [Dialogue setup](Life%20Cycle.md#dialogue%20setup) and released on [Dialogue Cleanup](Life%20Cycle.md##dialogue-cleanup).

White it is possible to bypass it this restriction by ignoring the value of message, doing so has undefined behaviors.

## waitinput

A global flag on MainManager that acts as a lock to wait for a confirmation input (confirm or cancel). This is used with the [Text advance](Related%20Systems/Text%20advance.md) system in Dialogue mode by yielding during [Dialogue post-processing](Life%20Cycle.md#dialogue-post-processing).

## tailtarget

The [entity](../Entities/Entity.md) pointed by the textbox. This is meant to show who is talking. This can be overridden by [tail](Individual%20commands/Tail.md).

## textbox

The box that holds the text in Dialogue mode. This contains the textholder. It can be hidden with a shrink animation or revealed with a grow animation.

## textholder

The GameObject that holds all the TextMesh [SetText](SetText.md) renders. Its parent is set to the `parent` parameter with a local position of the `position` parameter. Its localEulerAngles is set to zero if the `tridimensional` parameter is false and its scales is set to Vector3.one. Its name will be `Text: ` followed by the input string and its tag will be set to `Text`.

# fonttype

This is a parameter sent to [SetText](SetText.md) that determines the font to render.

## Font id table

|Name|ID|
|----|--|
|`BubblegumSans`|0|
|`D3Streetism`|1|
|UNUSED (gets overriden to `D3Streetism` by MainManager.FontID)|2|
|`Uzura`|3|
|`BalsamiqSans`|4|
|`ONEMobilePOP`|5|

