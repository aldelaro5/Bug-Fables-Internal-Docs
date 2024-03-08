# StartBattle - Pre `haltbattleload`
This is the first of 3 phases of [StartBattle](../StartBattle.md).

This phase is the first that ever happens. It ends by yielding as long as `haltbattleload` is true which can be set by the caller.

## instance.`battle` init
This part does basic initialisation and creates the instance.`battle` if needed:

- The current MainManager.`screenshake` is zeroed out
- MainManager.`battleenemyfled` is set to false
- instance.`lastdefeated` is reset to a new list
- All [OverWorldProjectile](../../Entities/NPCControl/ActionBehaviors/ShootProjectile.md#an-overview-of-overworldprojectile) are destroyed
- [ItemList](../../ItemList/ItemList%20State%20Machine.md)'s `inlist` is set to false
- The `camanglespeed` is set to 0.1 and `camanglechange` to false
- MainManager.`battleresult` is set to true
- MainManager.`battlenoexp` is reset to false
- The discovery HUD element is hidden by setting instance.`discoveryhud` to 0.0
- CancelAction is called on the player
- If it's not an existing battle, MainManager.`battle` is set to a new [BattleControl](../BattleControl.md) component added to the instance's gameObject. From now on, all BattleControl fields mentioned comes from that instance

## Start data init
This part only occurs when `saveddata` is false which can only happen when the battle was started for the first time because this field is only set to true at the very end of StartBattle.

This part initialises the `sdata` which is a [StartUpData](../StartUpData.md) that persists informations about the battle and restored when it's retried. A [retry](../Battle%20flow/Retry.md) involves calling StartBattle when the current one still exists which doesn't need this part's logic to occur.

### EnemyCheck
If calledfrom is null or its `eventid` is 0 or below EnemyCheck is called which can modify the enemyids using 2 special overrides.

The first one involves a potential override to `GoldenSeedling` if instance.`inevent` is false and [flags](../../Flags%20arrays/flags.md) 41 is true (chapter 1 ended). This is only applicable for the following [enemies](../../Enums%20and%20IDs/Enemies.md):

- `Seedling`
- `FlyingSeedling`
- `Acornling`
- `Underling`
- `Plumpling`
- `Flowering`

Any of these have a 1% chance to changed to a `GoldenSeedling`. However, there are 2 cases where these odds increases applied in order (Not mutually exclusive):

- If the current [map](../../Enums%20and%20IDs/Maps.md) is `SeedlingHaven`, the odds are 7.5%
- If the `Seedling` [medal](../../Enums%20and%20IDs/Medal.md) is equipped, it multiplies the base odds by 2.5 (meaning 2.5% or 18.75% at `SeedlingHaven`)

This results in the following odds:

|[map](../../Enums%20and%20IDs/Maps.md) is `SeedlingHaven`?|`Seedling` [medal](../../Enums%20and%20IDs/Medal.md) is equipped|Chances of `GoldenSeedling` (per applicable enemy party member)|
|------------|-------------|---------|
|No|No|1%|
|No|Yes|2.5%|
|Yes|No|7.5%|
|Yes|Yes|18.75%|

The second override applies if any [enemy](../../Enums%20and%20IDs/Enemies.md) is a `SandWyrm` or a `SandWyrmTail`. If this is the case, the entire enemyids array is overriden to be 2 elements with the first being `SandWyrm` and the second being `SandWyrmTail`

### StartData
After, StartData is called:

- If the player is `dashing`, StopDash is called without hitting a wall
- `sdata` is first reset to default values
- All `sdata` fields except `psc`, `esc` and `enemyweakness` are initialised to their respective values (see the [StartUpDate](../StartUpData.md) documentation to learn more)
- From there, GetStartConditions is invoked in 2.5 seconds which completes this saving process by setting `sdata.psc`, `sdata.esc` and `sdata.enemyweakness` to their respective values (the whole thing is enclosed in a try catch block whose catch will invoke it again in 0.5 seconds).

### Battle fought counter increment
Finally, [flagvar](../../Flags%20arrays/flagvar.md) 40 (number of battles fought) is incremented

## Essential setups
This part will do further initialisation:

|Field|Initial value|Notes|
|----:|------------|-----|
|`reservedata`|New empty list||
|`hideenemyhp`|false||
|`overworldmusic`|MainManager.`music[0]`.clip|Only set if MainManager.map.`musicid` is not negative and MainManager.map.`music` isn't empty<sup>1</sup>|
|`overmusic`|MainManager.`music[0]`.time|Only set if `overworldmusic` was and MainManager.`keepmusicafterbattle` is true|
|`canflee`|canescape|Sent as parameter|
|`currentaction`|`BaseAction` (the main vine action menu)||
|`currentchoice`|`Attack`||
|`option`|0||
|`action`|false||
|`alreadyending`|false||
|`expreward`|0||
|`moneyreward`|0||
|`partypointer`|New array with {0, 1, 2}|See [battle party addressing](../playerdata%20addressing.md#methods-of-addressing-durring-battle)|
|`longcancel`|The return value of InputIO.LongButton with input 5 (Cancel)|Determines if the input rendering should be a wide one or not|

1: Additionally, FadeMusic is called with a speed of 0.025 if all of the following are true:

- `overworldmusic` isn't null
- `overworldmusic`.name doesn't match the sent music 
- MainManager.map.`nobattlemusic` is false 

---

After, the following procedure is done:

- We enter a `pause`
- MainManager.[ApplyBadges](../ApplyBadges.md) is called
- MainManager.[ApplyStatBonus](../ApplyStatBonus.md) is called
- StopMovingEntities is called on the map

## Battle map 
This section determins the [battle map](../../Enums%20and%20IDs/BattleMaps.md) and the `extraenemies` if needed:

- If the sent stage is -1, it is overriden to the instance.`battlestage` (the value got set on the last MapControl.Start which sets it to the MainManager.map.`battlemap`)
- If the stageid is `SandCastle` or `SandCastleDark` while calledfrom's entity was `inice`, the stageid is incremented which sets it to `SandCastleIce` or `SandCastleDarkIce` respectively

## `extraenemies` setup
If the sent enemyids has more than 4 elements, all the elements after the 4th one are placed in order in `extraenemies` (after clearing it) and enemyids is overriden to only have the first 4 elements

## Battle in transition
This section setup the battle in transition:

- A `BattleStart0` sound is played (`BattleStart3` if the MainManager.map.`battleleaftype` is `Bee`) at 0.65 volume
- If the sent adv is 3 (enemy advantage) while we aren't `inevent`:
    - A `Hurt` sound is played followed by a `AtkFail` one
    - HurtParticle is called on the player position + Vector3.up with the `Hurt` sound (meaning the same sound will play twice on that frame)
    - All player party members's battleentity have their [animstate](../../Entities/EntityControl/Animations/animstate.md) set to 11 (`Hurt`) with a SetAnimForce call which will force a [SetAnim](../../Entities/EntityControl/Animations/SetAnim.md) call with force which renders the animation immediately
- If the player.entity.`sound` exists, it is stopped
- PlayTransition is called with id 2 (the battle in transition), the data being the map.`battleleaftype` at 0.075 speed and the color being the map.`battleleafcolor` unless adv was 3 (enemy advantage) when we aren't `inevent` where the color is overriden to pure red
- 1.5 seconds are yieled which lets the transition renders
- If MainManager.`haltbattleload` is true, all frames are yielded until it goes to false
