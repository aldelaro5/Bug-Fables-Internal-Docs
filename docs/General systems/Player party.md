## Player party
The player party is a BattleData array field called `playerdata`. It's one of the most important field in the game because it contains overworld and battle party information that is used throughout the game in several ways. This includes stats, current battle states, and the dual entities backing it (`entity` for the overworld, `battleentity` for the battles).

## `playerdata` addressing
The most complex part about `playerdata` is how the game addresses it. There's multiple ways to do it, each have their unique properties and they completely change depending on if we want to address the party in the overworld or in battle. There are 3 distinct ordering of the party:

- `trueid` order
- Overworld party order
- Battle formation order 

Since the addressing part is so complex, let's talk about it before going over the methods involved in managing or querying the party.

### Universal `playerdata` addressing
Here are all the ways to address `playerdata` that works regardless if we're addressing in the overworld or battles:

- Regular array index: Doesn't guarantee anything except if all party member are in order and the first one has a `trueid` of 0 (`Bee`) where in such case, the array indexes will always be the same as the `trueid`
- Finding the member with a specific BattleData's `trueid`: The animid associated with this party member at the time of the last ChangeParty which never changes until the next ChangeParty so it doesn't take into considerations overworld switching
- Finding the member with a specific `entity` or `battleentity`'s `animid`: The rendered animid of the party member taking into account all overworld switches (`entity` for the overworld, `battleentity` for battles)

The above remains true in all cases. While it's straight forward to understand why the entity's animid allows to address by rendered animid, what's less straight forward is why the array index doesn't guarantee anything and how the `trueid` works.

#### Addressing by array indexes directly
Essentially, the player party system supports to have any combination of members in any order. The formation can even change which is done by a method called ChangeParty (more details about it in a section below). It receives an int array parameter called ids that tells what animid should be in the party and in what order. It doesn't have to include the first 3 animids, it could have any combinations of them and any amount.

While for most of the game, the party is composed of `Bee`, `Beetle` and `Moth` which happens to match with the array index (0, 1 and 2), the game allows for different party formations and to even excluse memebers. It is very well possible to have a party of `Moth` and `Beetle`, but exclude `Bee`. It's even possible to have a party of a single member that isn't `Bee`. All these cases would lead into a situation where `playerdata[0]` may not actually be the `Bee` member. It might be someone else entirely simply because `Bee` wasn't the first party member (or they weren't present in the party at all).

Because of this, addressing `playerdata` by a direct index is the least useful way to address the party. It's only used when performing operations where not knowing whose member is who is acceptable or when looping through the `playerdata` array to do something on all members. It's more useful to enumerate the party in a certain way with the 3 different ordering that are detailed in the sections below.

#### `trueid` order
Due to this problem, this is where the `trueid` comes in. When the party gets created by ChangeParty, all `playerdata` will have their `trueid` set to the matching one sent in the ids parameter and it will never change until another ChangeParty.

Effectively, this allows to find out the animid each `playerdata` had at the moment of the last ChangeParty. This ordering is refered to as the `trueid` order and it is useful because it tells the canonical party order without considering any overworld switches. This order is also stored in `partyorder` which is also set by ChangeParty. However, since it doesn't consider any overworld switches, it means it's not as informative in the overworld because it doesn't tell who each `playerdata` is AT THE CURRENT MOMENT. It only tells who they were on the last ChangeParty. This method of addressing however becomes much more useful during battle which will be detailed in a section below.

### Addressing `playerdata` in the overworld
In the overworld, the party can be switched using SwitchParty(false). Doing this process renders the `trueid` less useful because the order of the party since the last ChangeParty is no longer the same then the current one. Moreover, addressing by `entity`.`animid` isn't sufficient because this only gets updated on the next RefreshEntities at the very end of a switch animation. It would be far more useful to have a way to know who's `playerdata` element is whom that follows the exact moment SwitchParty(false) is called.

#### Overworld party order
This is where the BattleData's `animid` comes in. Not to be confused with the `entity`.`animid` (which is the actual rendered animid), the BattleData's `animid` is a working value of `entity`.`animid`. It's basically the value `entity`.`animid` will be on the next RefreshEntities.

The reason this is useful is because overworld switching cycles all BattleData's `animid`. Effectively, they visually cycle their animid, but they don't actually change anything else. Finding a `playerdata` that has an `animid` of a specific value is the most reliable to find a member by its animid because it's essentially the `trueid`, but it follows overworld switches. This is refered to as the overworld party order: it allows to enumerate the party by their order in the overworld from front to back. For example, `playerdata[0].animid` always gives the logical animid of the leader in the overworld.

The only downside of the BattleData's `animid` is it doesn't reflect the ACTUALLY RENDERED animid. This is because like explained above, between the moment SwitchParty(false) is called and the moment the next RefreshEntities happen, there's a gap of time (15.0 frames) where the `entity`.`animid` won't be the same as the BattleData's `animid` and the `entity`.`animid` will lag behind.

To address by rendered animid, finding the member with a specific `entity`.`animid` is required. This can be useful for performing specific animations, but it's oftentime more recommended to address by BattleData's `animid` because it will always be logically accurate in at most 15.0 frames while the `entity`.`animid` might become outdated in at most 15.0 frames.

### Addressing `playerdata` in battles
Battles changes a lot how `playerdata` is addressed because switching the party works very differently using SwitchParty(true) and the starting order of the party in `playerdata` always matches the `trueid` order. To understand why, let's talk about what SwitchParty(true) does differently.

#### Battle formation order
Unlike SwitchParty(false), SwitchParty(true) doesn't actually changes anything in any `playerdata`. What it does instead is cycle `battle.partypointer`'s elements which was first assigned during StartBattle. `battle.partypointer` is a mapper that maps logical formation position to `playerdata` indexes. A logical formation number is a number starting from 0 that identifies members from front to back (so 0 is frontmost, 1 is one position back and 2 is 2 positions back). On its own, this doesn't do anything, but the caller (in this case, BattleControl's SwitchPos or SwitchParty) will go further and physically move each `playerdata` to their party position using `battle.partypointer` as the mapper.

So for example, to move the front member to its position after calling SwitchParty(true), BattleControl will set `playerdata[partypointer[0]]`.position = `partypos[0]`. This will work regardless of how the switch was done and when this is done on all `playerdata` elements using matching `playerdata` indexes, it will move all party member to their respective spot because `battle.partypointer` is the mapper that dictates everything here. Additionally, it also updates the BattleData's `pointer` to the matching `partypointer`, but this always result in being the same as the `playerdata` index.

What's important to mention here is that unlike overworld switching, the party members actually moves between each other, but the internal `playerdata` ordering will always remain in `trueid` order no matter what overworld order the party was in when StartBattle was called and no matter how many switches or swaps of positions happens during the battle.

This property is enforced by StartBattle: it will arrange the party in `trueid` order initially, but then performs a series of SwitchParty(true) until the party is aligned with the overworld party order. This implies that `battle.partypointer` will map to the same overworld party order in battle when enumerating `playerdata[battle.partypointer[i]]` elements sequentially, but further switches will only cause changes to `battle.partypointer`.

This way of enumerating the party gives us the battle formation order: it allows to enumerate the party by their formation position from front to back (right to left). For example, `playerdata[partypointer[0]]` always refer to the party member in the front.

This mechanism is also what allows SwitchPos to work: instead of cycling `battle.partypointer`, all that's needed to swap 2 members is to simply swap the 2 elements in `battle.partypointer` and refresh everyone's position in battle formation order.

Additionally, there's an interesting side effect to all of this: in battles, any BattleData's `trueid` will match their `battleentity`.`animid`. This is because StartBattle initially composed the battle entities in `trueid` order and only did SwitchParty(true) to make it align with the overworld party order, but that doesn't change any animid, only their physical positions and the `battle.partypointer` mapper. This means that in battles specifically, addressing by `trueid` can be used to address by animid and it is the most reliable way to address a specific member.

However, this method of party ordering has a downside: it now means that BattleData's `animid` are no longer valid during a battle. This is because battle switching has no animid cycling so it doesn't need to use it and their values only reflect the overworld which the battle system disociated on StartBattle. It means that it's not valid to address the party in overworld party order during a battle. This is usually fine because `trueid` order and battle formation order are plenty to address everyone by animid or by formation position.

## ChangeParty
Now that we're established how the party can be addressed, let's talk about the methods involved in managing the party and their stats.

Starting with the most important method that setup the party: ChangeParty:

```cs
public static void ChangeParty(int[] ids, bool fromscratch)
public static void ChangeParty(int[] ids, bool fromscratch, bool destroyoldentity)
```
This method is complex, but in general, it destructively reorganise the party in `playerdata` using `ids` as the animids where their order determines the `trueid` order. This method is the only way to change the `trueid` order. This is a desructive operation: it completely invalidates previous `playerdata` references and should only be done in a controlled manner such as specific events or when loading the save.

The `fromscratch` should always be true. If it's false, the behavior can be considered broken because the method won't actually change the party and it will instead result in `playerdata` being reset to a new list, but leave `partyorder` to `ids` which wouldn't be correct since the method didn't actually change the party. The only time the game sends false to this parameter is when loading the save file, but it doesn't actually do anything useful because the loading process sets `playerdata` to a new list anyway.

Finally, the `destroyoldentity` parameter will cause the older `playerdata`'s overworld entities that aren't present in the new party to be destroyed instead of being appended to `map.entities`. NOTE: This functionality of determining removed party members is broken, see the section below for details.

The overload without a `destroyoldentity` parameter has its value default to true.

Here's how the method process goes (`fromscratch` is assumed to be true since a false value leads to a broken party change):

- If `playerdata` isn't null or empty, all of the `entity` are saved in an array for reference later
- `partyorder` is set to `ids`
- `playerdata` is set to a new array containing the return values of SetDefaultStats using each `ids` elements in order (the method is detailed more below, but it notably sets the `trueid` and BattleData's `animid` to the matching `ids` element)
- Each entity in list saved earlier that aren't null are checked to see if they are to be removed from the party or stayed. NOTE: The determination logic is incorrect, check the section below for details. What happens depends if it was found the entity is removed from the new party or not:
    - If it's not removed, the matching `playerdata`'s `entity` is set to it. Their tag is set to `Player` with a layer of 10 () for `playerdata[0]` or `PFollower` with a layer of 9 () for other `playerdata`
    - If it is removed, what happens depends on `destroyoldentity`:
        - If it's true, the entity is destroyed
        - If it's false and the map isn't null (nothing happens if it is null), then the entity gets childed to the `map` with a tag of `Untagged` and it's appended to the end of `map.entities` after reassigning it to a new arrays
- If the older party wasn't empty and the new party isn't empty, but `playerdata[0].entity` doesn't have a PlayerControl, one will be added to it followed by a destruction of `player` (it will get automatically reasigned by PlayerControl's Start)
- ApplyBadges is called
- ApplyStatBonus is called
- All `playerdata`'s `hp` is set to their `maxhp`
- RebuildHUD is called

### Problem with party members removal
As explained above, ChangeParty should normally determine which party members is getting removed compared to the older party so it can either destroy the entity or move it out of the way. The problem is the logic to determine this is bogus and can't be relied upon.

What should be happening is figuring out if the older party contains all elements in `ids` and if it doesn't, the entities of the members that the new party has in excess should be destroyed / moved.

What is instead happening is only `playerdata[0].animid` is checked for this against the older entities's animids. It means that for any older party members whose `entity.animid` isn't `playerdata[0].animid`, it is incorrectly deemed as a removed party member even though they might appear in the party in a further `playerdata` element.

Even more confusingly, because it's based on animid, this is checked in older overworld party order. This means that even overworld switches before the ChangeParty call can influence this behavior. As a general rule however, this issue only has a bad effect on a ChangeParty where `ids` has more than one element.

A workaround to this issue is to not rely on the proper member's entities composition and just call SetPlayers after the ChangeParty. This will recreate every `entity` of every BattleData with the proper animid derived from the BattleData one. It is considered undefined to not do this after a ChangeParty because it may lead to overworld entities being incorrectly destroyed and left to null.

## SwitchParty
This method rotates the party where each member cycles to the position in front of them except for the front member which cycles to the backmost position. Here is its signature:

```cs
public static void SwitchParty(bool battle)
```
The `battle` parameter determines the kind of switching: true means a battle switch while false means an overworld switch. They behave very differently with each other with a high level explanation mentioned in a section above. It should never be called with a value of true while MainManager.`battle` is null or an exception will be thrown.

The method does nothing for an overworld switch while `inevent`.

Here is the detailed process for an overworld switch:

- PlaySound(`Switch`) called
- All `playerdata`'s `animid` are rotated with each other such that `playerdata[X].anim` becomes `playerdata[X + 1].anim` where `X` is any `playerdata` index except for the last one where its animid will become `playerdata[0].anim`
- `switchicon` (if it exist) gets its sprite changed to `guisprites[playerdata[0].animid + 94]` which is the icon sprite whose animid is the frontmost member with a color of full white, but half transparent
- `playerdata[0].entity.emoticonoffset` (the frontmost member's `entity`.`emoticonoffset`) is set to (0.0, 1.8 + 0.25 * their BattleData's `animid`, -0.1)

Here is the detailed process for a battle switch:

- All `battle.partypointer` elements are rotated with each other such that `battle.partypointer[X]` becomes `battle.partypointer[X + 1]` where `X` is any `playerdata` index except for the last one where the number becomes `battle.partypointer[0]`

## Other party utilities
The following are a bunch of potentially useful utilities to work with `playerdata` and its BattleData as a whole. This doesn't include the methods that has to do with the party stats as these are brought up in a section below.

```cs
public static bool HasPlayer(int id)
```
Returns true if any `playerdata` has a `trueid` of `id`, false otherwise.

```cs
public static BattleData GetPlayerData(int id, bool frombattleentity)
public static BattleData GetPlayerData(int id)
```
Returns the first `playerdata` whose `trueid` is `id` or the first `playerdata` if none exists.

If `frombattleentity` is true however, the method will first search for a `playerdata` whose `battleentity` has a `battleid` of id and it will return it if one is found. Otherwise, the regular search is done.

On the overload that doesn't take a `frombattleentity`, the default value is false.

```cs
public static BattleData? GetPlayerDataNullable(int id)
```
Returns the first `playerdata` whose `trueid` is `id` or null if none exists.

```cs
public void RefreshPlayer(bool onlycollider = false)
```
Reset various physical related state to the `player` and `playerdata`:

- `player`.`ceiling` is set to false
- If `onlycollider` is false, every `playerdata` whose `entity`.`noclock` is false gets their entity rooted to the scene and their `entity`.`onground` set to false
- If the game isn't in a `battle` and not `inevent`, ForceHitWall is called on `player`.`entity`

```cs
public static bool FreePlayer(bool getfly)
public static bool FreePlayer()
```
Only returns true if all of the following conditions are fufilled (returns false otherwise):

- `player` exists (meaning a map isn't being loaded)
- The game isn't in a `minipause`
- The game isn't in a `pause`
- The game isn't `inevent`
- The SetText `message` lock isn't active
- The `player` isn't `digging` or using the submarine's sonar
- If `getfly` is true, the `player` also needs to not be `flying` (otherwise, this condition doesn't apply)

On the overload without `getfly`, the default value is true.

```cs
public static bool IsPlayerInPos(int pos, Transform entity)
```
Returns true if `playerdata[battle.partypointer[pos]]`.`battleentity` is `entity`, false otherwise.

```cs
public static bool IsParty(Transform obj)
```
Returns true if `obj` is a `playerdata`'s `entity`'s transform, false otherwise.

```cs
public static bool PartyIsNotMoving()
```
Returns true if no `playerdata`'s `entity` are in a `forcemove`, false otherwise.

```cs
public static int[] PartyArray()
```
This is UNUSED. Returns an array of a single element being `playerdata[0]`.`trueid`.

```cs
public static void SetPlayers()
public static void SetPlayers(Vector3[] newentitypos)
```
A part of the map loading process. Recreate all `playerdata` elements. If `newentitypos` isn't null, the existing `playerdata` gets destroyed before the recreation and after, their position are set to matching `newentitypos` elements (indexed by `playerdata` index).

The parameterless overload acts as if `newentitypos` was null.

The recreation process is documented in the map loading documentation, but the method is also used in certain events that needs to recreate the party.

```cs
public static EntityControl[] GetPartyEntities()
public static EntityControl[] GetPartyEntities(bool idorder)
```
Returns an array of all `playerdata`'s `entity`. If `idorder` is false, the array is ordered by `playerdata` index. If `idorder` is true, the order is done by animid of `Bee`, `Beetle` and `Leif` (more precisely, it's GetEntity(-4), GetEntity(-5) and GetEntity(-6)).

```cs
public static Vector3[] GetPartyPos(bool inorder)
```
Return the `playerdata` positions in an array. If `inorder` is false, they are ordered by `playerdata` index. If `inorder` is true, they are in the following order: GetEntity(-4), GetEntity(-5), GetEntity(-6).

```cs
public static void DestroyPlayers(bool remake, bool inorder)
```
This is UNUSED, but remains functional. Destroys all `playerdata`'s `entity`.gameObject. If `remake` is true, SetPlayers(GetPartyPos(`inorder`)) is called after.

```cs
public static void StopDig()
public static void StopDig(bool delayed)
```
Does the following on each `playerdata`'s `entity` which ends any digging animations that were ongoing:

- Call LockRigid(false) on the entity
- Reset `spin` to Vector3.zero
- Reset `startscale` to Vector3.one
- Reset the `sprite`'s scale to Vector3.one
- Reset `overrideanim` to false
- Enables the `ccol`

The overload with a `delayed` parameter is UNUSED, but remains functional and it will start a DigStop coroutine instead, but all it does is yield for 0.1 seconds before calling the other overload so it's effectively doing the same after waiting for 0.1 seconds.

```cs
private static IEnumerator DigStop()
```
Part of the UNUSED StopDig overload where it will yield for 0.1 seconds before calling the parameterless overload of StopDig.

## Party stats
The way the party stats are calculated is complex because it's a manual process that involves multiple components being added together.

As explained in ChangeParty above, there's 3 methods it ends up calling that messes with the new BattleData and these 3 methods when called together allows to calculate the effective stats of the party:

- SetDefaultStats (needs to be called on all `trueid` of the party which ChangeParty normally does). This NEEDS to be called to initialise some fields
- ApplyStatBonus (operates on the whole party). It applies all `statbonus`
- ApplyBadges (operates on the whole party). It applies all BadgeEffects

Some of these methods ends up doing similar steps than the others and some even calls the other methods, but it doesn't have negative effects to do so because they are built in a way that ensure the correct stats numbers are set by the end. There is one requirement: both ApplyBadges and ApplyStatBonus needs to be called to guarantee a full recalculation of the party stats (order doesn't matter). Even if it results in a double call, these methods are resilient to this because they reset their numbers before proceeding.

To summarise the best way to call these methods:

- Calling ChangeParty (with `fromscratch` set to true) guarantees the party stats are fully calculated so calling any of these methods after isn't necessary
- Once medals equipment changes occurs that implicate changes to stats, ApplyBadges should be called
- Once `statbonus` changes happen, ApplyStatBonus should be called

### SetDefaultStats
Mentioned in the ChangeParty section is a method used to build the basic fields of the new BattleData. That method is SetDefaultStats:

```cs
private static BattleData SetDefaultStats(int id)
```
It returns a BattleData with the following fields:

- `animid` and `trueid`: `id`
- `hp`, `maxhp` and `basehp`: 7 (9 if `id` is 1 which is `Beetle`)
- `atk` and `baseatk`: 2
- `skills`: new empty list
- `cursoroffset`: 2.3 in y
- `entityname`: `menutext` at index 46 + `id` (this is basically the language specific party member name of `Bee`, `Beetle` and `Moth`)
- `condition`: new empty list

This essentially initialises the basic fields so the BattleData is ready for use. Notably, it is the only method that sets `trueid`.

ChangeParty should always call this. The only time it doesn't is when the game setup the party when loading the save file, but it ends up doing the same important steps.

### ApplyStatBonus
This method is what ends up applying the `statbonus` which are permanent stats upgrades the party accumulated through various means. They are saved on the save file with a specific format documented in the save file format documentation.

Here is its signature:

```cs
public static void ApplyStatBonus()
```
It does the following to apply all of the `statbonus` on top of the base party stats:

- Calls ResetStats
- Apply all of the `statbonus` (check the save file documentation for the formatting)
- If `statbonus` wasn't empty, calls ApplyBadges

The way it works is it only messes with the following fields:

- All `playerdata`'s `basehp`
- All `playerdata`'s `baseatk`
- All `playerdata`'s `basedef`
- The party's `basetp`

These "base" fields aren't actually used for anything in the game except for stats calculation since they represents the stats after `statbonus` are applied, but before BadgeEffects are applied. Most notably, it's only useful for ApplyBadges to use these numbers as a base, but this is taken into account because if there was any `statbonus`, ApplyBadges gets called. Not having any `statbonus` would still mean that ApplyBadges would need to get called, but it doesn't matter if it was done before or after since ApplyStatBonus wouldn't have done anything in that case.

As for ResetStats, it's a simple method that ensures the "base" fields are reset to their base value before the procedure:

```cs
private static void ResetStats()
```
It resets every `playerdata`'s `basehp`, `baseatk` and `basedef` to their base value and reset `basetp` to its base value. The base values are the following:

- `basehp`: 7 (9 if the `playerdata`'s `trueid` is 1 which is `Beetle`)
- `baseatk`: 2
- `basedef`: 0
- `basetp`: 10

This is why ApplyStatBonus can be called multiple times without negative effects.

### ApplyBadges
ApplyStatBonus isn't enough on its own because it only applies `statbonus`, but it doesn't consider medals which have effects on stats. Those effects are called BadgeEffects which are defined in the medals data, check the medals data documentation to learn more.

ApplyBadges is the method that will apply these effects on top of the base stats:

```cs
public static void ApplyBadges()
```
Here's the exact procedure:

- Set instance.`speedup` (an UNUSED field) to true
- Set instance.`maxtp` to instance.`basetp`.
- CallResetPlayerStats is called on player party member (more details below)
- All the `BadgeEffects` of every applicable medals are applied just as described in the medals data documentation.
- instance.`tp` is clamped from 0 to instance.`maxtp`
- All `playerdata`'s `hp` is clamped from 0 to their `maxhp`

As for ResetPlayerStat, it's a simple method that ensures the stats fields are reset to their initial value before the procedure:

```cs
private static void ResetPlayerStat(int id)
```
More precisely, it sets these following fields:

|Field|Value set|
|----:|-----|
|`lockitems`|false|
|`lockskills`|false|
|`locktri`|false|
|`lockrelayreceive`|false|
|`maxhp`|`basehp`|
|`atk`|`baseatk`|
|`def`|`basedef`|
|`poisonres`|0|
|`sleepres`|0|
|`freezeres`|0|
|`numbres`|0|

## Other party stats utilities
Here are other useful party stats related methods:

```cs
public static void AddStatBonus(StatBonus type, int ammount, int to)
```
Adds a new element to the `statbonus` list using the `type`, `ammount` and `to` values provided. Check the save file documentation for what these values mean.

```cs
public static void Heal()
public static void Heal(bool noparticle, bool nosound)
public static void Heal(Healing[] parameters, int[] partyids, bool noparticle, bool nosound)
```
Heals the `playerdata` whose index is in `partyids` where the healing is determined by `parameters` and perform the associated visual / audio effects for the healing. If `nosound` is true, PlaySound(`Heal`) will not be called. If `noparticle` is true, HealParticle will never be called even if it would have been otherwise. This method also sets `hudcooldown` to 100.0 to show the party HUD temporarilly. If HealParticle gets called, the parent is the `playerdata`'s `entity` or `battleentity` depending if the game is in a `battle`.

For the first 2 overloads, the `parameters` value defaults to a single element being `Full`, `partyids` defaults to `partyorder` (which is every `playerdata`), `noparticle` defaults to false and `nosound` defaults to false.

Here are the healing effects of the different `parameters` value:

- `Full`: The `playerdata`'s `hp` is set their `maxhp` and the party's `tp` is set to `maxtp`. Particles are included if `noparticle` is false
- `TPOnly`: The party `tp` is set to `maxtp`. Particles are always excluded regardless of `noparticle`
- `FullHPOnly`: The `playerdata`'s `hp` is set their `maxhp`. Particles are included if `noparticle` is false

```cs
public static IEnumerator LevelUpMessage()
```
Applies any rank up bonuses if any exists and recalculate the party stats. Check the LevelUpMessage documentation to learn more.

```cs
public static bool PartyLowHP()
```
UNUSED: No one calls this, but it remains functional.

Returns true if any `playerdata` has an `hp` of 4 or below, false otherwise.
