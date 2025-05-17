## Player party

```cs
public static void ChangeParty(int[] ids, bool fromscratch)
public static void ChangeParty(int[] ids, bool fromscratch, bool destroyoldentity)
```
TODO: document the party system better and explain better how a change party works because it has wide implications on the playerdata addressing

```cs
private static void ResetStats()
```
Resets every `playerdata`'s `basehp`, `baseatk` and `basedef` to their base value and reset `basetp` to its base value. The base values are the following:

- `basehp`: 7 (9 if the `playerdata`'s `trueid` is 1 which is `Beetle`)
- `baseatk`: 2
- `basedef`: 0
- `basetp`: 10

```cs
public static void ApplyStatBonus()
```
Does the following to apply all of the `statbonus` on top of the base party stats:

- Calls ResetStats
- Apply all of the `statbonus`
- If `statbonus` wasn't empty, calls ApplyBadges

```cs
public static bool HasPlayer(int id)
```
Returns true if any `playerdata` has a `trueid` of `id`, false otherwise.

```cs
public static bool PartyLowHP()
```
UNUSED: No one calls this, but it remains functional.

Returns true if any `playerdata` has an `hp` of 4 or below, false otherwise.

```cs
private static BattleData SetDefaultStats(int id)
```
A part of ChangeParty.

Returns a BattleData with the following fields:

- `animid` and `trueid`: `id`
- `hp`, `maxhp` and `basehp`: 7 (9 if `id` is 1 which is `Beetle`)
- `atk` and `baseatk`: 2
- `skills`: new empty list
- `cursoroffset`: 2.3 in y
- `entityname`: `menutext` at index 46 + `id` (this is basically the language specific party member name of `Bee`, `Beetle` and `Moth`)
- `condition`: new empty list

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
public static bool IsParty(Transform obj)
```
Returns true if `obj` is a `playerdata`'s `entity`'s transform, false otherwise.

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
public static bool PartyIsNotMoving()
```
Returns true if no `playerdata`'s `entity` are in a `forcemove`, false otherwise.

```cs
public static void RefreshSkills()
```
Refresh each `playerdata`'s `skills` list which represents all the skills that are available to each player party member. Check the skills data documentation for more details.

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
public static int GetFreePlayerAmmount()
public static int GetFreePlayerAmmount(bool hponly)
```
Returns the amount of player party members that are considered free in battle. Check the GetFreePlayerAmmount documentation to learn more.

```cs
public static int GetAlivePlayerAmmount()
```
Returns the amount of `playerdata` whose `hp` are above 0 and aren't `eatenby`.

```cs
public static bool AllPartyFree()
```
Returns true if no `playerdata` has a `cantmove` of 1 or higher (no actor turns available on the current main turn), false otherwise.

```cs
private static void ResetPlayerStat(int id)
```
A part of ApplyBadges where `playerdata[id]` has several fields reset to their base value. Check the ApplyBadges documentation to learn more.

```cs
public static void SwitchParty(bool battle)
```
TODO: this is sort of documented, but not organised right so this needs to be organised

```cs
public static IEnumerator LevelUpMessage()
```
Applies any rank up bonuses if any exists and recalculate the party stats. Check the LevelUpMessage documentation to learn more.

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

```cs
public static void AddStatBonus(StatBonus type, int ammount, int to)
```
Adds a new element to the `statbonus` list using the `type`, `ammount` and `to` values provided. `statbonus` contains a list of permanent stats bonuses to apply from the base stats when calculating the party's stats. They are saved to the save file.

```cs
private static bool HasSkillCost(int tpcost, int playerid)
```
Returns true if `tp` is at least `tpcost`, false otherwise.

However, if `playerdata[playerid]` has the `HPFunnel` medal equipped or if `tpcost` is negative, this instead returns true when `playerdata[playerid]`.`hp` - 1 is at least the absolute value of `tpcost`, false otherwise.

This method is intended to check if the player is able to pay the cost of a skill whether it is in TP (`tpcost` isn't negative) or HP (`tpcost` is negative or `playerdata[playerid]` has the `HPFunnel` medal equipped). In the case the cost is in HP, the HP cost is the absolute value of `tpcost` - 1.

```cs
public static bool AddExp(int ammount)
```
Increases `partyexp` by `ammount`. If this resulted in a rank up, true is returned, false otherwise. A rank up is detected if after the increase, `partyexp` becomes at least `neededexp`. When that happens, `partyexp` is decreased by `neededexp`.

Before returning, a PlaySound call happens to play the the `Exp2` sound with a volume of 1.0 and a pitch of 1.5 + a random number between -0.1 and 0.1. The `sounds` element to use is the first among `sounds[4]`, `sounds[5]`, `sounds[6]` and `sounds[7]` in that order that isn't playing (always falsback to `sounds[7]` even if it is playing).
