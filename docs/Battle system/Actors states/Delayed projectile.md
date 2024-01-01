# Delayed projectile
A delayed projectile, also known as a delproj, is a projectile an enemy can launch, but that will only land on a player party member after a predetermined amount of main turns passed. The projecile will cause damage with an optional property attached. The visual and audio effects are all configurable.

The delprojs currently scheduled are tracked by the `delprojs` field which is an array of `DelayedProjectileData`, a struct documented in the section below. The actual logic of the delprojs are defined in [AdvanceMainTurn](../Battle%20flow/Action%20coroutines/AdvanceMainTurn.md#delprojs-advance) with their main turn advances done very early in the coroutine.

## DelayedProjectileData
This struct defines the entirety of a delproj, here are all the fields and their semantics:

|Name|Type|Description|
|----|----|-----------|
|turns|int|Amount of main turns left before the projectile lands|
|damage|int|The amount of damage to deal to the target|
|position|int|The player party member position to target|
|areadamage|int|UNUSED|
|framestep|float|The amount of frames to move the projectile before landing completely|
|obj|GameObject|The projectile object|
|deathparticle|string|A `Resources/prefabs/particles` name that will play when the projectile fully lands|
|deathsound|string|A `Resources/audio/sounds` audio clip's name that will play when the projectile fully lands|
|whilesound|string|A `Resources/audio/sounds` audio clip's name that will be play while the projectile is landing on loop. For the clip to play only once instead of looping, the name needs to be prepended by `@`|
|args|string|`@` separated strings containing optional configurations, see the section below for details|
|calledby|MainManager.BattleData|UNUSED|
|property|AttackProperty?|The property of the damage dealt|

### `args` field
The `args` of the `DelayedProjectileData` are a list of commands which is `@` separated while each command is a list of string that is `,` separated.

Each command's first element is the name of the command while every strings after are parameters. Here is a table that shows all the vailable commands, their parameters meaning if any and the overall effect of the commmand if applied:

|Name|Parameters|Description|
|---:|------|-----------|
|partoff|First 3 params are float forming a Vector3|Offsets the `deathparticle` position to play by the Vector3 in parameters relative to the `obj` position upon landing|
|move|First 3 params are float forming a Vector3|Offsets the position to land on by the Vector3 in parameters relative to the `partypos` of the delproj's `position`|
|noshadow|None|Prevent a ShadowLite to be added to the delproj's `obj`|

## Delprojs methods
To manage delprojs, there are 2 methods that exists that allows to add or remove them:

- AddDelayedProjectile 
- RemoveDelayedProjectile 

Both methods converts the array to list, performs the necessary modifications and then converts the list back to array and assigns it to `delprojs`.

The sections below details the methods's signatures and parameters details.

### AddDelayedProjectile
Adds a delproj to `delprojs` with all the parameters needed to initialise one.

```cs
private void AddDelayedProjectile(GameObject obj, int targetpos, int damage, int turnstohit, int areadamage, AttackProperty? property, float framespeed, MainManager.BattleData summonedby, string hitsound, string hitparticle, string whilesound)
```

#### Parameters
Each parameters directly maps to a field in `DelayedProjectileData` of the delproj to add. Here is the mapping:

- `obj`: `obj`
- `targetpos`: `position`
- `damage`: `damage`
- `turnstohit`: The value + 1 is mapped to `turns` (this is because the main turn that the projectile is fired is excluded from `turns` so the first [AdvanceMainTurn](../Battle%20flow/Action%20coroutines/AdvanceMainTurn.md) will remove it after its launch and the [enemy phase](../Battle%20flow/Update.md#enemies-phase) is done)
- `areadamage`: `areadamage` (this field is UNUSED because it is never read)
- `property`: `property`
- `framespeed`: `framestep`
- `summonedby`: `calledby` (this field is UNUSED because it is never read)
- `hitsound`: `deathsound`
- `hitparticle`: `deathparticle`
- `whilesound`: `whilesound`

### RemoveDelayedProjectile
Removes an existing delproj from `delprojs` by its array index.

```cs
private void RemoveDelayedProjectile(int arraypos)
```

#### Parameters

- `arraypos`: The `delprojs` array index to remove the delproj from the array
