# EntityControl Creation

EntityControl is a components in the game that defines what an [Entity](../Entity.md) is and what it can do with some minor exceptions that one might think goes outside this scope. It can be considered the building blocks of the game as it represents anything that is created and behaves at runtime.

## CreateNewEntity

While there are multiple ways to create an entity, all of them ends up at the same method:

````cs
public static EntityControl CreateNewEntity(string name)
````

The `name` here has 2 purposes:

* It will be the name of the root GameObject of the entity which can be useful for debugging purposes
* It can contain some special strings called modifiers that changes how the entity behaves.

This method will create an object structure that is shared across every entities. While an entity can modify this structure later on, all of them starts the same way:

* The root GameObject is created with its name being the `name` parameter and an EntityControl component is added to it
* The transform of the root object is cached into the `transform` field
* Another GameObject is created and childed to the root one called "Rotater". This won't be assigned to anything until the entity's Start runs on the next frame where it is assigned to the `rotater` field.
* Another GameObject is created and childed to the Rotater one called "Sprite" with a SpriteRender component. It is assigned to the `sprite` field and its transform into the `spritetransform` field.
* Another GameObject is created and childed to the root one called "MoveRotater". It is assigned to the `moverotater` field.
* A CapsuleCollider is added to the root object with radius 0.5, height of 2.0, center at (0.0. 1.0, 0.0) and assigned to the `ccol` field.
* The layer of the entity is set to 10 which is the Entity layer.

This essentially gives us the following structure:

````
-> root (named after the name parameter)
	-> Rotater
		-> Sprite (has a SpriteRender)
	-> MoveRotater	
````

And the following fields:

* transform: the root's transform
* rotater (on Start): the Rotater
* sprite: the Sprite
* spritetransform: the Sprite's transform
* moverotater: the MoveRotater
* ccol: the CapsuleCollider of the root

However, this is only the most primitive way to create an entity. There are 2 overloads of this method that accepts different parameters, but one is a helper to the other whose signature is the following:

````cs
public static EntityControl CreateNewEntity(string name, int anim_id, Vector3 position, EntityControl follow)
````

The `name` ends up doing the exact same thing than the primitive version as this method is a wrapper around it. What changes is the ability to specify an [AnimIDs](../../Enums%20and%20IDs/AnimIDs.md), a position and a follower:

* `anim_id`: This is the [AnimIDs](../../Enums%20and%20IDs/AnimIDs.md) of the entity and it is set to the `animid` field.
* `position`: This sets the starting position of the entity's root transform.
* `follow`: This parameter is optional because there is an overload that doesn't have it and it sends null to this method. If it is present, a follower is setup on the entity.

The follower is initialized like this when applicable:

* The game tries to find the last entity of the follow chain. This is done by checking the entity's `following` field to go up the chain until it is null.
* The entity found is set to the `following` field of the entity being created which puts it at the end of the chain.
* The `followeroffset` field is set to the position (starting at 1) of the entity in the follow chain multiplied by 0.1.
* The layer of the entity is overridden to 9 which the Follower layer
* The tag of the entity is set to `PFollower`

## OnEnable

The creation process doesn't stop there because EntityControl defines an OnEnable which is run immediately on creation OR whenever it gets enabled. This will do 2 things:

* Do [SetAnim](Animations/SetAnim.md) call with empty `args` and `force` to true which will force the animation to start on "Idle" as it's the default value of [animstate](Animations/animstate.md).
* If the emoticon object exists, it will be set to -1 (none) and its cooldown to 0 frames. Since an entity cannot have one on creation, this only applies when it gets enabled with one present later.

## Map entities preset

On top of this, there is a system the game has that allows to define each entities's starting parameters for a map along with an [NPCControl](../NPCControl/NPCControl.md) stored in `npcdata`. This data comes from \[\[TextAsset Data/Entity data#`entitydata` directory\]\] and is done by MapControl's [CreateEntities](CreateEntities.md) which happens on its Start.
