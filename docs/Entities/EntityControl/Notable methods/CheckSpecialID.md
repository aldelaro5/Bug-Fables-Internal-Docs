# CheckSpecialID

This is a void returning parameterless method on EntityControl that is ran during [Start](../Start.md). It can be considered as part of the [Entity startup](../EntityControl%20Creation.md#entity-startup) process because its name is underselling what the method does as its job is to initialise the fields and structure of any entity based on the [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md). Because of this, it is the method that will assign most fields from [EntityValues](../../../TextAsset%20Data/Entity%20data.md#`EntityValues`%20data) among other things.

Since this method implies extremely specific handling of specific AnimIDs, their handling will only be documented on a surface level, not in detail.

Here is what the method does:

* Unless `overrideshadow` is true, `hasshadow` becomes true if the AnimIDs is not None (which is -1)
* `originalid` is set to the current animid
* From here, there are 2 special cases that can change what data is used depending on the animid:
  * If it's `KeyL`, `KeyR` or `Tablet`, then the following fields gets set and nothing else will get assigned from `endata`:
    * `nomodel`, `overrideanim`, `overrideanimfunc`, `overridemovesmoke` and `overrridejump` are all true
    * `minheight` is pulled from `endata`
    * `startbf` is set to `bobrange`
    * `startbs` is set to `bobspeed`
    * The appropriate sprite from `Sprites/Objects/artifacts` from the root of the asset tree is set as the entity's sprite
  * If it's `Strider`, `overridemovesmoke` is set to true, but the rest goes as it normally would with the `endata` loading
* If the AnimID is defined, the data from `endata` is assigned, otherwise only `overrridejump` is set to true
* [Special AnimID startup](Special%20AnimID%20startup.md)
* The object is setup whenever `endata`'s `Object` is true or it's a `CoilyVine` and we aren't dealing with `KeyL`, `KeyR` or `Tablet`. This is done by setting `notalk` to true, `hasshadow` to false, `startscale` to `endata`'s `startscale` and adding the model of the corresponding AnimID via [AddModel](AddModel.md) that's located in `Prefabs/Objects/` with the offset being `endata`'s `modeloffset`. The `animid` is overridden to -1 (None) unless it's a `SavePoint` or a `FlyTrapPlatform`
* If we aren't dealing with `KeyL`, `KeyR` or `Tablet` and the original animid is not -1 (None), then further startup steps are performed:
  * If the `endata`'s `ismodel` is true and it's not an item
    * The model of the corresponding AnimID that's located in `Prefabs/Objects/` with the offset being `endata`'s `modeloffset` is added via [AddModel](AddModel.md). The offset is overriden to (0.0, 0.0 - `height`, 0.0) if it's a `TrappedMoth`. The `model`'s angles are set to the `endata`'s `modelscale` TODO: why?
    * From there, there are 3 special cases:
      * `JumpingSpider`: Set all child of the model to be rendered with the 3F3F3F color (light gray) fully opaque
      * `Pitcher` or `PictherSummon`: Setup 2 `extras`, one being the object with the tag `PitcherEnd` and the other being the second child of the `model`. The first `extras` gets the `Untagged` tag. For the second one all child's shadowCastingMode is set to TwoSided and the material set accordingly if `hologram` with half transparency. Finally, `spinextra` is set to to one vector being (0.0, 0.0, -0.2)
      * `Submarine`: If there's an `npcdata` and it's not the player's entity, the `ccol` is enabled and adjusted to have a radius of 2.5. The `npcdata`'s `colliderheight` is set to 20.0. NOTE: the enabling happens in 0.2 seconds in the future
    * [UpdateAnimSpecific](../Animations/AnimSpecific.md#updateanimspecific) is called
  * More `endata` fields are loaded:
    * `emoticonoffset` to `freezeflipoffset`
    * `freezesize` (defaults to (2.0, 2.0, 1.0) if this and `freezeoffset` is 0)
    * `freezeoffset` (defaults to (0.0, 1.0, 0.0) if this and `freezesize` is 0)
  * `initialfrezeoffset` is set to `freezeoffset`
  * If there's a `preloaddata` in `endata`, they are all loaded into `preloadedobjects` and `preloadedsprites` by loading the actual prefabs/sprites. They remain null otherwise
  * If the current map is laoded and we aren't in a battle
    * if the current map is MetalLake, `startscale` is multiplied by 0.35 and the `emoticon`'s localscale is set to (0.65, 0.65, 0.65)
    * If the current map's `waterfloat` is present and we are dealing with a `Strider`, `ignorewater` and `alwaysactive` are set to true on top of the ignoring the collider of the map's `waterfloat` with this entity. Finally,  the y position of the entity is set to the one of the `waterfloat`
* If the `animid` is `Scorpion` and we are at Giant's Lair, the `basestate` is overridden to 150 and `walkstate` to 151 which makes them render differently.
