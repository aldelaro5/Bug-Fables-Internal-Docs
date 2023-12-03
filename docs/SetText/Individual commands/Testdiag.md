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

* [next](Next.md)
* [blank](Blank.md)
* [speed](Speed.md)
* [icon](Icon.md)
* [menu](Menu.md)
* [color](Color.md)
* [shaky](Shaky.md)
* [faketail](Faketail.md)
* [rainbow](Rainbow.md)
* [glitchy](Glitchy.md)
* [wavy](Wavy.md)
* [goto](Goto.md)
* [call](Call.md)
* [minibubble](Minibubble.md)
* [breakend](Breakend.md)
* Any commands that contains `line` in their name as specified in the input string (this is case sensitive). This means the following list are also processed if the command is written to satisfy this condition:
    * [line](Line.md)
    * [halfline](Halfline.md)
    * [quarterline](Quarterline.md)
    * [pauseline](Pauseline.md)
    * [libraryline](Libraryline.md)
    * [shopline](Shopline.md)
    * [backline](Backline.md)
    * [unpauseline](Unpauseline.md)

Any other commands will not be processed. ([tail](Tail.md) and [tailextra](Tailextra.md) will be technically be processed too, but their command processing will break immediately under dialogue test mode).

This does not impact [ignorenext](Ignorenext.md) functionality which takes priority over deciding to process the commands or not. Additionally, there is no way to disable test mode meaning when processing this command, the rest of the input string will be processed.

This command is unused under normal gameplay, but it remains functional.

### About line commands case sensitivity

It is possible to have a line command not be processed in test mode by writing `line` differently such as `Line`. Since the comparison is done with case sensitivity, these line commands can be selectively processed in test mode or not depending on the syntax specified in the input string. This means if the intent is to process all of them, it is important to make sure they are specified in lowercase. While the command parsing is case insensitive, the line comparison in test mode is not. 
