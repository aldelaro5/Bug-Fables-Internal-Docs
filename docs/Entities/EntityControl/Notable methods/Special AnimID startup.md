# Special animid startup

This is the part of [CheckSpecialID](CheckSpecialID.md) that deals with specific [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md):

* `GoldenSeedling`: setups the `Prefabs/Particles/GoldStars`
* `RizGrandpa`: creates a line to render as a fishing rod's line
* `SeedlingKing`: Adjusts the `height` and `initialheight` in battle and setups 4 `subentity` which will be seedling rendered under him
* `Midge`: Setups 2 wings in the `extra` array with their sprite set to `Sprites/Entities/Midge`
* `Cape`: setups the `Prefabs/Particles/Snowflakes` if it's an ice variant or `Prefabs/Particles/Flame` if it's a fire one to the only `extra`. The fire variant is in effect only if `forefire` is true OR that the current map is in Giant's Lair except for GiantLairFridgeInside. Finally, the fire variant has its `animid` overriden to FireCape
* `Watcher`: setup the `extra` to be 2 elements and setup the first one to be `Prefabs/Objects/WatcherBook`. The materials of the pages of the models is adjusted to the holosprite one if `hologram` is true. Finally, `nodigpart` is overridden to true
* `Krawler`: Very similar to `Cape`, except the `animid` to override on the fire variant is `FireKrawler` and if it's the ice variant AND it is not `inice`, then the particle are stopped with playOnAwake turned off
* `CursedSkull`: Same than `Krawler` except the `animid` overridden on the fire variant is FireWarden
* `Ahoneynation`: The `rotater` gets a SpriteBounce setup with 0.03 as the frequency and 9.0 as the speed
* `WaspTwinA`: `basestate` is set to Sad if they are following an entity
* `None`: the `rigid`'s gravity is disabled on top of its constraint being set to freeze all. Additionally, the `ccol` is disabled
* `CoilyVine`: the same thing than `None` on top of overriding the check to load a prefab model for later
* `BeeBoss`: the `Prefabs/Particles/BeeBossBottomSmoke` from the root of the asset tree is setup as a child to `shadowtransform`. `shadowtransform`'s local position is then set to 0
* `BeeBot`: ensures that the enemydata\[battleid\].data has one element which contains a random number being either 0 or 1. From there [UpdateAnimSpecific](../Animations/AnimSpecific.md#updateanimspecific) is called
* `CrystalBerry`: `Prefabs/Objects/CrystalBerry` from the root of the asset tree is added as model via [AddModel](../Notable%20methods/AddModel.md) with the offset coming from `endata`'s `modeloffset`. The `animid` is overriden to 3 with a `spin` set on the y axis and `item` set to true
* `Seedling`: the speed is set to 2.5 if it was 0.0 or below on top of the `npcdata`'s `speedmultiplier` being set to 1.0 if it is present
* `AngryPlant`: `Prefabs/Objects/LeafSkirt` from the root of the asset tree is setup as the only `extras`. This includes setting all child's shadowCastingMode to off and to set the correct materials if `hologram` and `cotunknown` (the later being changing the color to pure black with 50% transparency)
* `Turret`: `Prefabs/Objects/turretbase` from the root of the asset tree is setup as the only `extras`. `cotunknown` changes its material color to pure black with 50% transparency
* `WaspDriller`: if npcdata.vectordata\[0\].y is -2.0, setup an item sprite as the only `extra` where the [Items](../../../Enums%20and%20IDs/Items.md) id is npcdata.vectordata\[0\].x and its flipX is true
* `Zombeetle`: Same as WaspDriller, but also setup the `extras`'s angles and localscale on top
