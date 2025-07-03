# InputIO
InputIO is the only component in the game that's not really used as a MonoBehavior, but more used as a general utility static class. Almost all of its members are static and the only reason it is technically a MonoBehavior is because at least on the Steam platform, there is a OnApplicationQuit Unity event. It is never attached to anything and is almost always used as a static utility class.

It mostly handles any tasks related to:

- [Platform specific](PC%20platforms.md) logic (Steam, GOG or Itch.io)
- [Disk files handling](Disk%20files%20handling.md)
- [Inputs](Inputs.md)

That being said, not all the logic of the above is contained in InputIO. InputIO frequently works with several MainManager methods for things like handling file formats or for abstracting inputs further.

It also is notable in the sense it is the only class that was compiled with specific flags to add or remove logic depending on the platform. It also is the only class that doesn't reside in the global namespace, it resides in the `InputIOManager` namespace.

The class isn't very isolated however. A lot of its logic makes use of MainManager and it is generally not really decoupled from the rest of the game.

All the methods are documented in the pages in this directory and all the fields are documented in the [fields page](InputIO%20fields.md).
