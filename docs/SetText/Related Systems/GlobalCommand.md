# GlobalCommands

This system was an experiment to improve one of the flaws of [SetText](../SetText.md) which was that every commands had to be specified inline with its text. This meant that localisation could create a lot of mishaps from copy pasting the lines and changing the commands around which can lead to problems as bad as softlocks.

The system has to be enabled for each [Maps](../../Enums%20and%20IDs/Maps.md) via `useglobalcommand`, but the game only has it enabled on the [Map](../../Enums%20and%20IDs/Maps.md) SnakemouthEmpty which is a map that was only used in the demo and ended up unused in the final version. Nonetheless, the code to support it is still in and in theory, accessible if `useglobalcommand` is enabled for the map. It is through a method called GlobalCommand:

```cs
public static void GlobalCommand(ref string text)
```
Replaces the global commands placeholder text in `text` to their actual commands. This is an unused system.

Instead of specifying every commands in the same line, a global command [SetText](../SetText.md) line has `@` to denote placeholders. From there, GlobalCommand replaces these with strings contained in the map's `commandlines` using `;` as separator which ends up like a normal [SetText](../SetText.md) line. The map's `currentline` is used to resolve the commands to use which is expected to be set before SetText processes the text (if it's -1, it means the system isn't in use and GlobalCommand won't do anything). This meant that every line in the map could have common commands shared positionally without actually needing to specify them. This is meant to operate on the current map's current line.

In theory, it would only solve the problem partially since it could allow map's lines to have less [SetText](../SetText.md) errors after translation, but considering how [SetText](../SetText.md) is used in many ways outside of the map lines, it's safe to say that this system was incomplete. It doesn't account for the many lines needed for things like descriptions or enemy tattles.

It's safe to speculate that it was left unused as an artifact for its experimentation. It wouldn't have been possible to implement it in any significant ways in the game as this was done too late to make it possible.
