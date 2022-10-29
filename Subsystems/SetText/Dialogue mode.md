# Dialogue mode

SetText has 2 main mode of operation: dialogue mode and non dialogue mode. The flag to use dialogue mode or not is sent as a parameter from the [Entry point](Entry%20point.md). 

## Non dialogue mode

This mode restrict accesses to features dialogue mode would have and is in general a simpler mode of operation. The only notable exclusive behavior it has is in [life cycle > Setup](life%20cycle.md#setup), a `|spd,0|` is prepended to the input string which forces wait times to be disabled. Other than that, many commands becomes not usable in non dialogue mode.

One big advantage of non dialogue mode is it allows multiple SetText calls running at the same time and it supports recursion. This means that any SetText calls (even in dialogue mode) can itself calls SetText in non dialogue mode however it wants. Since non dialogue mode doesn't have accesses to the features dialogue mode offers, it prevents conflicts with the MainManager fields that dialogue mode uses. Additionally, this means any caller can call SetText in non dialogue mode however many times is needed.

Generally, this mode is recommended for UI text because it greatly simplify the processing and in UI, most of the feature given by dialogue mode gets in the way as well as the inability to be called multiple times at once.

## Dialogue mode

Dialogue mode is a superset of non dialogue mode. It allows access to many features such as [Text advance](Related%20Systems/Text%20advance.md), [Backtracking](Related%20Systems/Backtracking.md), [Prompt](Commands/Individual%20commands/Prompt.md), [Pickitem](Commands/Individual%20commands/Pickitem.md), auto bleep on every letter and in general, anything that the game needs to do to present text that scrolls. It also allows to control a textbox with the [tailtarget](Notable%20local%20variable/tailtarget.md) set to the [Parent](Commands/Individual%20commands/Parent.md) and it gives access to the full set of commands features.

To accomplish this, dialogue mode has additional phases in the [life cycle](life%20cycle.md):

* [life cycle > Dialogue setup](life%20cycle.md#dialogue-setup)
* [life cycle > Dialogue pre-processing](life%20cycle.md#dialogue-pre-processing)
* [life cycle > Dialogue post-processing](life%20cycle.md#dialogue-post-processing)
* [life cycle > Dialogue Cleanup](life%20cycle.md#dialogue-cleanup)

A huge disadvantage of this mode is it is not possible to have 2 SetText calls in dialogue mode running at the same time. This not only prevents recursion, but it also means that any caller cannot calls SetText in dialogue mode more than once. Failure to respect this rule will cause undefined behaviors.

Generally, anything that needs these features should operate in dialogue mode as long as that call can safely be the only one in this mode at a time. Only if none of these features are needed should non dialogue mode be used such as UI text.
