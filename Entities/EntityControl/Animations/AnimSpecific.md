# AnimSpecific

AnimSpecific is a system that allows the game to go beyond its standard conventions of the [animstate](animstate.md) and to perform more complex animation setup. The system is made of multiple methods that are called at specific points:

* UpdateAnimSpecific
* AnimSpecificQuirks

## UpdateAnimSpecific

This is a void returning parameterless method on EntityControl that is ran at specific points involving the [animstate](animstate.md) updating. It can contain logic specific to the entity's [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) which allows to use non standard animations and setup. This method doesn't apply to `item`.

It's a more frequently called method because not only it gets called everytime the `animid`, the [animstate](animstate.md) changes or during a [Freeze handling](../Notable%20methods/Freeze%20handling.md) event, it can be called externally if needed.

First, all objects in `animspecific` are destroyed if any were present. Then, the [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) specific part occurs (only high level details are provided as most of these involve detailed rendering):

* EverlastingKing: The `shadowsize` is set to 5.0 if the [animstate](animstate.md) is 115
* MimicSpider: The `OutOfBattle` animation parameter is set depending on `battle`. The `shadowsize` is set to 3.0 unless the [animstate](animstate.md) is `Idle` and `Walk` in which case, the value is set to 1.75 instead
* Bee: if the [animstate](animstate.md) is `BattleIdle`, `animspecific` is initialised with one element being a `Prefabs/AnimSpecific/BeeBIdle` from the root of the asset tree. This is her Beemerang animation when idling during battle.
* Watcher: The same than Moth, but before, the second `extras` is destroyed if present (unless he was underground during the battle) with the `bobspeed` and `bobrange` set to their start field counterpart.
* Moth: if the [animstate](animstate.md) is `BattleIdle` or `PickAction`, `animspecific` is initialised with one element being a `Prefabs/AnimSpecific/mothbattlesphere`. Either of the 2 [animstate](animstate.md) determines how it will be positioned (fixed if `BattleIdle` or with a SpinAround for `PickAction`)
* Eremi: If the [animstate](animstate.md) is `Happy`, the `walkstate` becomes `Chase` and the `basestate` becomes `Happy`
* Tanjerin: The `walkstate` is overriden to 101 if the [animstate](animstate.md) was `Angry` or `Surprised` and to `Walk` if it was `Idle` or `Sad`
* JumpingSpider: Same than Spuder, but before, the bubble's sprite is enabled or disabled depending on the battle's `enemydata` in `battleid`'s `holditem` is higher than -1. This just manage if the item sprite of the bubble should be enabled or not.
* Ruffian: This only applies if the `extras` have not been initialised. This initialises 4 of them which is his chains (the first 3) and ball (the 4th one). It will then setup them in a specific way such that the first 3 are linked together to the ball. The ball gets a FollowerLite and a ShadowLite. It also considers `hologram` for the materials.
* Strider: Initialises the `extraanims` if they weren't already to 4 new elements and the `extras` to being the second child of the model and have all the children of that be the `extraanims`. This simply setups the legs and their animator. Finally the only `extras` gets its scale adjusted on the Z axis to 1.0 unless the [animstate](animstate.md) is `Walk` or `Chase` in which case, the value is 0.5. This creates a constricting effects on his legs when needed
* DeadLanderC: `extraanims` is initialised to 4 new elements being the second, third, fourth and fifth child of the `model`. Then, specific animations are played on them depending of the [animstate](animstate.md). This essentially manages the legs.
* Scorpion: Same then Spuder, but before, the `basestate` and `walkstate` are overridden to 150 and 151 respectively if the current [Area](../../../SetText/Commands/Individual%20commands/Area.md) is Giant's Lair. This is what makes them look different in that area. 
* Spuder, DivingSpider and PeacockSpider: This is a very complex management of the legs. Notably, `overridejump` is set to false all the time and `extraanims` is initialised with the legs's animators. This also manages special animation clips to play on the legs.
* Venus: Setup 6 `extraanims` being all the children of the second child of the `model` with their speed being set to 1 unless the [animstate](animstate.md) is 100 or 101 in which case, the value is 0. This simply manages her legs animations
* BeeBot: Ensure its wings's StaticModelAnim are enabled when the `icecube` isn't present or disabled if it is with fixed angles.
* MotherChomper: `extras` is initialised if it wasn't already to 1 element containing the second child of the second child of the `model` which is the ground sprite
* VenusGuardian: This is where the arms information are initialised if `extraanims` wasn't initialised yet. The `extraanims` are both arm's animator. It also initializes 4 `extras` with the first 2 being her leaf wings, the third being the ground and the 4th being the flower canon on her left arm. The ground and flower canon will be enabled or disabled depending on her `height` and [animstate](animstate.md). Finally, this also plays the proper animations on the `extraanims` depending on her `height`, [animstate](animstate.md) and other conditions.
* AngryPlant: The only `spinextra` is initialised to zero or (0.0, -20.0, 0.0) if the `height` is higher than 0.1. This also sets the only `extras` scale depending on the `height`.
* Seedling: If the [animstate](animstate.md) is `Hurt` when it had a copter with the `height` above 0.1, it will be deparented while the object will fly offscreen at the top using a LerpObject coroutine

## AnimSpecificQuirks

This is a void returning parameterless method on EntityControl that is ran on [LateUpdate](../Update%20process/Unity%20events/LateUpdate.md) before [UpdateSprite](../Update%20process/UpdateSprite.md) and on [UpdateSprite](../Update%20process/UpdateSprite.md) itself if the [animstate](animstate.md) or the `inice` changed. This is more a way to adjust the animation for a specific [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) so it can have custom logic to it. It can be seen as an update method because it handles the adjustements of the extra fields such as `extras`, `spinextra`, `extralines` and `extrasprites`, but it also allows specific [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) to have their own update logic. Unlike the other anim specific update method, this is only called internally during the normal update cycle.

Here are all the specific update logic defined (only described at a high level since most of it is rendering specific):

* ChompyChan: `extrasprites` is ensured to have one SpriteRender and it is enabled depending on the value of [flags](../../../Flags%20arrays/flags.md) 404 (Chompy has a ribbon on her). Then, `ChompyRibbon` is called which only sets the proper color of the sprite depending on which ribbon is in [flagvar](../../../Flags%20arrays/flagvar.md) 56 (The [Items](../../../Enums%20and%20IDs/Items.md) id of the ribbon equipped on her).
* JumpingSpider: If the [animstate](animstate.md) is `Chase`, the `walktype` is set to Jump and to Normal otherwise
* SeedlingKing: Unless the `subentity` weren't inialised yet, override each's [animstate](animstate.md) to `Hurt` if the entity's [animstate](animstate.md) is `Hurt` or to `Chase` otherwise.
* Watcher: `overrideshadow` is set to `digging` and the `shadow`'s enabled to the inverse of `digging`. Then, nothing happens if the `extras` (containing his book) weren't initialized yet, but if they are, the local position of the only one is updated on the y with a Lerp between the current y and a vector determined by the `spritetransform` local position, the [animstate](animstate.md) and if `digging` was true or not. This lerp is done with a factor of 0.1. The whole thing essentially just renders the book with a floating effect unless he is `digging`.
* BeeBot: This only applies if the entity is `dead` or in the process of dying (`iskill` or `deatcoroutine` not being null). When it does, the animation `Idle2` or `Hurt2` is played depending on the current form of the entity which is determined either by the current battle data if it's a `battle` entity or the [animstate](animstate.md) if it's not.
* Krawler and CursedSkull: Only applies when `inice` has changed (tracked by `lastice`) and the `npcdata` if present doesn't have an enabled `disguiseobj`. This will play the `IceShatter` particle and also control whether the first extras's ParticleSystem should play when `inice` is true or stop otherwise.
* MotherChomper: the first `extras` if it exist has its angle set to zero which is the ground object
* VenusGuardian: Only applies when `extras` has been initialised. If it was, the third `extras`'s angles (which is the ground object) is set to zero and the first 2 (her leaf wings) have their angles adjusted to a sin curve on the x axis if the `height` is higher than 0.1 and there's no `icecube`.
* Scarlet: The `shadowsize` is set to 2.5 unless the [animstate](animstate.md) is among specific values which will cause the value to be set to 1.25 instead.
* Zasp: The `shadowsize` is set to 1.0 unless the [animstate](animstate.md) is among specific values which will cause the value to be set to 1.5 or 1.75 instead depending on the [animstate](animstate.md)
* ShielderAnt: The `shadowsize` is set to 1.0 unless the [animstate](animstate.md) is `KO` which causes the value to be set to 2.0 instead.
* Seedling: This manages the `copter` sprite if applicable which is where the `height` is above 0.1 among other conditions. It initialises the `extras` if it wasn't already with one element being that sprite and it also initialises `spinextra` to 20.0 units on the z axis. This finally adjusts the `bobspeed` and `bobrange` to 2.0 and 4.0 respectively if they were lower than 0.1 (this also assign the start field counterpart to the new value)
* Flowering: Same then Seedling, but before, the only `extras` is activated if `flyinganim` and the [animstate](animstate.md) is a [animstate > Extra animations](animstate.md#extra-animations), it is disabled otherwise
* UltimaxTank: The `extras` initialised first if they weren't already. The first 3 extras are the wheels, the 4th is the body object (the one above the wheels) and the 5th is the missile launchers. Then, their position and angles are adjusted depending on the [animstate](animstate.md)
* MidgeBroodmother: This only applies when `model` is not null. The `extras` (the wings) are initialised if they weren't already to the children of the first child of the `model`. From there, their angles are adjusted.
* DeadLanderB: If the `extrasprites` weren't initialised, this method will initialise them, `extralines` and `spinextra`. This essentially manages the balloons and the lines connecting them by setting various properties about them (also considers `hologram` and `cotunknown`).
* KeyR and KeyL: The angles are adjusted on the Z axis depending on the [animstate](animstate.md) and `inice`.
* Tablet: In battle, the sprite is set depending on if the HP. If the HP is below 3, the `startscale` is set to Vector3.one * 0.75
* Pitcher and PitcherSummon: This only applies if the `extras` were intialised. It adjusts the second `extras`'s position (the end of the next) to the first's position (the head) + the first `spinextra`.
* Midge: Only applies if the `extras` were initialised. It manages the position and angles of the wings in `extras` depending on the specific [animstate](animstate.md).

For all entity except `DeadLanderB`, `Pitcher` or `PitcherSummon`, every `extras`'s angles gets incremented by the `spinextra` of the corresponding index if it exist.
