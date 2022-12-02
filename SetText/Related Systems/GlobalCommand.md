This system was an experiment to improve one of the flaws of [SetText](../SetText.md) which was that every commands had to be specified inline with its text. This meant that localisation could create a lot of mishaps from copy pasting the lines and changing the commands around which can lead to problems as bad as softlocks.

The system has to be enabled for each map via useglobalcommand, but the game only has it enabled on the map SnakemouthEmpty which is a map that was only used in the demo and ended up unused in the final version. Nonetheless, the code to support it is still in and in theory, accessible if useglobalcommand is enabled for a map.

Instead of specifying every commands in the same line, a global command SetText line has `@` to denote placeholders. From there, GlobalCommand replaces these with strings contained in the map's commandlines using `;` as separator which ends up like a normal SetText line. This meant that every line in the map could have common commands shared positionally without actually needing to specify them. This is meant to operate on the current map's `currentline`.

In theory, it would only solve the problem partially since it could allow map's lines to have less SetText errors after translation, but considering how SetText is used in many ways outside of the map lines, it's safe to say that this system was incomplete. It doesn't account for the many lines needed for things like descriptions or enemy tattles.

It's safe to speculate that it was left unused as an artifact for its experimentation. It wouldn't have been possible to implement it in any significant ways in the game as this was done too late to make it possible.
