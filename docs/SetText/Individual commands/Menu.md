# Menu

Alias to [string](String.md) if processed by SetText directly where `flagstring` is a menu text line id instead, but it can also be processed before by [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) with different behaviours.

## Syntax

(1)

````
|menu,menutext|
````

(2)

````
|menu,menutext,clamp,maxwidth|
````

(3)

````
|menu,menutext,clamp,maxwidth,widthscaleclamp|
````

(3)

````
|menu,menutext,true|
````

## Parameters

The same as [string](String.md) except `flagstring` is now a menu text line id:

### `menutext`:  int

The menu text line id whose value will be appended at the end of the input string. This must corresponds to an integer of a valid menu text id or an exception will be thrown.

## Remarks

This command can get processed by [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md)'s ReplaceFunctions's own char loop if it is called at least once with the input string before the char loop reaches this command.

### [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md)'s processing

Syntax (1) is fully supported and behaves the same way, but the other ones are not. This is because [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) also adds a different syntax involving the second parameter:

````
|menu,menutext,formatting|
````

The `formatting` parameter adds some formatting to the menu text and it can be any of the following values (any other values is ignored):

* `1`: The text of the command is replaced by the first character of the menu text followed by a `-` followed by the entire menu text.
* `2`: The text of the command is replaced by an uppercased version of the menu text
* `3`: The same as `1`, but the portion after the `-` is uppercased.

### About processing order

Since this command has the unique property that it is processed very differently between SetText and [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md), it is important to be mindful who will run the command first. 

It should be noted that if SetText runs it first using syntax (3), it will not cause interference because [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) will be run after the command text was replaced which means it will not process the command a second time, but it means it is no longer possible to process it using the special [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) syntax. 

If [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) runs it first, then SetText can no longer process it because the command will disappear from the input string and in such case, only syntax (1) and the special syntax of [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) are supported.
