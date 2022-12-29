# Choicewave

Enable a PromptAnim on the [textholder](../../Notable%20local%20variable/textholder.md) that will highlight the text with a wave visual effect if it is hovered on during a [Prompt](Prompt.md).

## Syntax

(1)

````
|choicewave,optionindex|
````

(2)

````
|choicewave,optionindex,underline|
````

## Parameters

### `optionindex`: int

The index of the concerned [Prompt](Prompt.md) option. If the current [Prompt](Prompt.md) option is this value, the text will be rendered with the highlight effect enabled and disabled otherwise. This must corresponds to a valid `int` value or an exception will be thrown.

### `underline`: var

Whether to render a red `_` underneath the text once it is highlighted instead of highlighting the text in red. Any value is accepted here as the value itself does not matter, only its presence.

## Remarks

This command allows a [Prompt](Prompt.md), [NumberPrompt](NumberPrompt.md) and [LetterPrompt](LetterPrompt.md) to highlight the currently hovered choice by applying a `PromptAnim` which apply a wave effect and a red color to the text or the `_` in (2).
