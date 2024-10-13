# MapControl
MapControl is a data driven component that defines a game map. It is meant to be attached to the root of a prefab and has most of its fields defined from the prefab's asset data (meaning they were assigned in the Unity editor before the game was built).

Most of MapControl is meant to be data driven and it exposes only a portion of methods to perform some procedures (such as managing insides which are interiors portion of the map). This means that most usage of MapControl comes from using its fields outside of MapControl itself. Most fields are initialised by MapControl itself on its Start or have their value assigned from Unity and read by other systems when needed. Essentially, the logic of MapControl is much more spread out outside of it and it is a very data centric component. There's still some public methods available, but they are more specific in what they offer.

As a result, most of MapControl's influence is spread out throughout the rest of the game. Every systems is free to check the fields of the MapControl and react accordingly which tends to be what happens.

In practice, the main way the game interacts with MapControl is the current map which is accessed via MainManager.instance.map. Due to this, while MapControl represents an instance of a map, most usage is done in a static and global context because only the current map's fields are checked. The only time a MapControl is created is when a map prefab containing one is instantiated. This is why that all MapControl in the game must be at the root GameObject of a prefab.
