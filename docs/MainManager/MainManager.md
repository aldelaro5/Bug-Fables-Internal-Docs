# MainManager
MainManager is by far the most important Unity component in the game. It has a handle on practically almost everything in the game logic from [booting](Boot%20and%20reset%20process.md) to large systems like [SetText](../SetText/SetText.md) and [ItemList](../ItemList/ItemList.md) to smaller logic such as general math utilities methods. It encompasses the most amount of global state in the game and it is best described as a "God object" or "All knowing object". It also references almost every other large systems that aren't in MainManager such as EventControl (manages [events](../Enums%20and%20IDs/Events.md)), [BattleControl](../Battle%20system/BattleControl.md), [MapControl](../MapControl/MapControl.md), etc..

Its design is a simple singleton: its Start Unity event sets its `instance` field to itself early on boot and the rest of the game uses it to access any MainManager instance fields. Due to the extreme globalness of the component, there's very little distinction in practice between a MainManager instance field and a static field because any instance field can be accessed with MainManager.`instance`. The main difference is instance fields don't persists after a [Reset](Boot%20and%20reset%20process.md#reset) while static fields do, but accessibility wise, there's no differences between the two.

It is impossible to have more than 1 MainManager at a given time as this would result in the 2's state fighting with each other resulting in a broken game beyond recovery. The only MainManager allowed to exist is the one that exists in the main scene which is the only scene in the entire game and it is on the `Main Camera` GameObject (which is childed to the `MainCam` GameObject). Since it's basically on the main [camera](../General%20systems/Camera%20system.md#main-camera) that is required to render anything on the `Default` layer (which is most GameObjects), it is never destroyed and doing so would also result in a broken game beyond recovery. The only time the game is allowed to not have a MainManager is during Unity engine boot before the scene gets loaded and during Unity's LoadScene(0) call that happens in a Reset. In other words, there must be 1 and only 1 MainManager available to boot the game and to have the game function in general and it must be accessible via MainManager.`instance` from very early boot time.

A rule of thumb to keep in mind when dealing with MainManager is that when in doubt, any logic is in MainManager. It is the component with the tightest coupling on just about everything in the game.

## How the MainManager documentation are structured
The root folder of the MainManager documentation contains important lifecycle methods each contained in their own pages:

- [Boot process](Boot%20and%20reset%20process.md)
- [DoClock](DoClock.md)
- [Update and FixedUpdate](Update%20and%20FixedUpdate.md)
- [LateUpdate](LateUpdate.md)
- [RefreshEntities](RefreshEntities.md)

Due to the globalness of MainManager, they are effectively part of the entire game's life cycle.

As for the fields, they are documented in 2 pages:

- The [Used fields](MainManager%20fields.md)
- The [Unused fields](Unused%20fields.md)

They contain everything no matter what system they are related to. The entire game's documentation frequently talks about these fields because almost everything ends up consulting the MainManager fields.

The other methods are however documented differently. The "Methods" folder from these documentation contains various pages with categorised methods doing various things. These methods are there because they are used in many situations globally accross many places so there's not really a particular place the method's documentation belong other than MainManager.

However, these methods are far from all the methods present in MainManager. A significant amount of them arguably shouldn't be in MainManager because they specifically concern an already separated part of the game's logic. For example, many [BattleCondition](../Battle%20system/Actors%20states/Conditions.md) related methods don't belong to [BattleControl](../Battle%20system/BattleControl.md) like it probably should, but they belong in MainManager.

To avoid confusion, such methods won't be documented here and they will instead only be mentioned when they apply in the system they concern the most. Only the methods that have wide reaching global usage are documented in the MainManager documentation.

## About general systems
Additionally, MainManager isn't just a global object: it's the home of many very global systems in the game by itself (a good example would be [SetText](../SetText/SetText.md) which is very complex, yet, entirely resides in MainManager). Due to this, there is a "General systems" folder from the root of the game's documentation that has a page dedicated to each systems that mostly belong in MainManager and have very global usage in the game, but can all be explained in one page.
