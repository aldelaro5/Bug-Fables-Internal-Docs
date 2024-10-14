# MapControl
MapControl is a data driven component that defines a game map. It is meant to be attached to the root of a prefab and has most of its fields defined from the prefab's asset data (meaning they were assigned in the Unity editor before the game was built).

## General archtecture
MapControl can be thought as a substitute to a Unity's scene because it offers a wide variety of features just like a scene, but it's not a scene, it's a component. It means it doesn't just handle defining the map, it also handles anything surrounding the map such as its music, rendering, entities, the follower system or other specific construct that might be included in the map.

Most of MapControl is meant to be data driven and it exposes only a portion of methods to perform some procedures (such as managing insides which are interiors portion of the map). This means that most usage of MapControl comes from using its fields outside of MapControl itself. Most fields are initialised by MapControl itself on its Start / the first LateUpdate or have their value assigned from Unity and read by other systems when needed. Essentially, the logic of MapControl is much more spread out outside of it and it is a very data centric component. There's still some public methods available, but they are more specific in what they offer.

As a result, most of MapControl's influence is spread out throughout the rest of the game. Every systems is free to check the fields of the MapControl and react accordingly which tends to be what happens.

In practice, the main way the game interacts with MapControl is the current map which is accessed via instance.`map`. Due to this, while MapControl represents an instance of a map, most usage is done in a static and global context because only the current map's fields are checked. The only time a MapControl is created is when a map prefab containing one is instantiated. This is why that all MapControl in the game must be at the root GameObject of a prefab.

## MapControl's fields
MapControl is very fields centric because most of its fields drives how the component behaves. There are 2 categories of fields:

- Configuration fields: These are all public and they are meant to have a value defined in a prefab from Unity. Some of them are required for normal function and some of them are optional with a default value enforced somewhere in the init process. Due to them being public, it is possible they are also used in other systems
- Tracking fields: These fields may or may not be public and they aren't meant to be driven by the Unity prefab, but by itself or the rest of the game. Even if a tracking field is public, while a value CAN be given in the prefab, it either will be ignored or cause malfunction. These fields act more like regular class fields that tracks information and are effectively part of the MapControl API

Configuration fields can be thought as what drives MapControl and can be overriden by the game while the tracking fields are what MapControl keeps track of. 

## Init process
The initialisation process of MapControl is a multi step process mainly contained in Start and the first LateUpdate (otherwise known as `latestart`). It also includes some methods invoked shortly after Start or the first LateUpdate.

After this process, all configuration fields should have coherent values (meaning the value got set to default if it was optional). There's also a couple of features that are setup specifically in this init process.

## Update lifecycle and features
MapControl only has 2 Unity events that contain logic after the init process called the update process:

- LateUpdate (excluding the `latestart` portion of the first call since that portion is in the init process)
- FixedUpdate

While this lifecycle is simple, the real complex part of MapControl comes down to the wide variety of features it offers. Many can be configured via configuration fields.

They are documented in separate pages each:

- Identification: Describes a map and how the game logic changes depending on it
- Camera: Describes the camera system and how MapControl can configure it further
- Graphics: Describes how the rendering of the map can be configured
- Battles: Describes how battles on the map can be configured
- Music: Describes how the map music system works and how it can be configured
- Insides: Describes how MapControl's interiors works which are called the MapControl's `insides`
- SetText: Describe the role MapControl has with SetText and how it can be configured from the map
- Followers: Describes the follower system and how MapControl influences it
- Entities: Describes how MapControl's entities behave and how map entities (otherwise known as NPCControl) are defined
- General configuration: This contains everything that doesn't fit in the above categories

However, not all of them can be configured. Some features are dependant only on the GameObjects inside the map having a certain form:

- digwall: A GameObject whose collider disappears when the `player` is `digging`
- Merged mesh: A feature where GameObjects with a certain tag can have their meshes merged
- Particles level: A feature where the rendering of certain particles can be tuned down or turned off completely via a game's setting
- EntityOnly colliders: A feature where GameObjects with a certain tag can have their colliders be ignored for the player entities and their followers, but remain active for map entities
