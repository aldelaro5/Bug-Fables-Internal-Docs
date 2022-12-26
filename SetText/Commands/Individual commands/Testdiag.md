# Testdiag

Enables the dialogue test mode until the end of SetText processing.

## Syntax

````
|testdiag|
````

## Parameters

None.

## Remarks

The dialogue test mode will restrict what commands will be processed from this point and will ignore any commands when in non dialogue mode. In dialogue mode, command processing is restricted to the following commands:

* [Next](Next.md)
* [Blank](Blank.md)
* [Speed](Speed.md)
* [Icon](Icon.md)
* [Menu](Menu.md)
* [Color](Color.md)
* [Shaky](Shaky.md)
* `Faketail`
* [Rainbow](Rainbow.md)
* [Glitchy](Glitchy.md)
* [Wavy](Wavy.md)
* [Goto](Goto.md)
* [Call](Call.md)
* [Minibubble](Minibubble.md)
* [Breakend](Breakend.md)
* Any commands that contains `line` in their name as specified in the input string (this is case sensitive). This means the following list are also processed if the command is written to satisfy this condition:
  * [Line](Line.md)
  * [Halfline](Halfline.md)
  * [Quarterline](Quarterline.md)
  * [Pauseline](Pauseline.md)
  * [Libraryline](Libraryline.md)
  * [Shopline](Shopline.md)
  * [Backline](Backline.md)
  * `Unpauseline`

Any other commands will not be processed. ([Tail](Tail.md) and [Tailextra](Tailextra.md) will be technically be processed too, but their command processing will break immediately under dialogue test mode).

This does not impact `Ignorenext` functionality which takes priority over deciding to process the commands or not. Additionally, there is no way to disable test mode meaning when processing this command, the rest of the input string will be processed.

This command is unused under normal gameplay, but it remains functional.

### About line commands case sensitivity

It is possible to have a line command not be processed in test mode by writing `line` differently such as `Line`. Since the comparison is done with case sensitivity, these line commands can be selectively processed in test mode or not depending on the syntax specified in the input string. This means if the intent is to process all of them, it is important to make sure they are specified in lowercase. While the command parsing is case insensitive, the line comparison in test mode is not. 
