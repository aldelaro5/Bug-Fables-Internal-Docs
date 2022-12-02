# SetText

SetText is Bug Fables's text management system. It is a hybrid between a text renderer, a script processor and a dialogue management system. Its name comes from the name given to its main [SetText entry point](SetText%20entry%20point.md) which is a static Coroutine defined in MainManager.

Its main features are:

* Serve as the only organised method in the game to render any textual information on screen using the different [fonttype](fonttype.md) available.
* Supports 2 distinct modes of operation: [Dialogue mode](Dialogue%20mode.md) or non dialogue mode. The later is a stripped version of the former that offers the advantage of not having any limits on the amount of calls running at a given time which is ideal for UI texts.
* Supports 217 different [Commands](Commands/Commands.md) offering a wide variety of features ranging from [FontEffects](Related%20Systems/FontEffects.md) and rendering to a simplified form of a Turing complete scripting system.
* Provides the different facilities needed to manage dialogues between entities such as a [Text advance](Related%20Systems/Text%20advance.md) system, a [Backtracking](Related%20Systems/Backtracking.md) system and the management of a [textbox](Notable%20local%20variable/textbox.md) that can optionally point to its speaker using the [tailtarget](Notable%20local%20variable/tailtarget.md).
* Include an automatic line breaker called [OrganiseLines](Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) (with some caveats, see [OrganiseLines Known Issues](Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines%20Known%20Issues.md)).

For a more detailed understanding of the inner workings, see [life cycle](life%20cycle.md).
