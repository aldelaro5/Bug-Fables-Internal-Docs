# Delayed condition
A delayed condition is a [condition](Conditions.md) placed on an enemy party member that whose infliction attempt occurs during the [post action](../Battle%20flow/Action%20coroutines/DoAction.md#post-action) phase of [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md) when the actionid isn't -555 (a dummy value to perform a blank call), it's an enemy action that didn't cause them to flee and it didn't cause them to kill themselves (`selfsacrifice` is false).

NOTE: Only [Freeze](BattleCondition/Freeze.md), [Numb](BattleCondition/Numb.md) and [Sleep](BattleCondition/Sleep.md) are supported because they are the only ones that gets processed.

The list of them is tracked in the `delayedcondition` field of the enemy party member and are ephemeral in nature: once processed, they are removed so they are only valid per action.

## AddDelayedCondition
Adding a delayed condition involves calling the AddDelayedCondition method.

```cs
private void AddDelayedCondition(int enid, MainManager.BattleCondition cond)
```

enid is the `enemydata` index and `cond` is the condition to add to the corresponding `delayedcondition` list. The method will ensure the list isn't null and if the condition didn't already exist (duplicates aren't allowed), it will be added.

Specifically, if a [Freeze](BattleCondition/Freeze.md) condition is added via this method, the `frostbitep` of the enemy party member is initialised to be the ParticleSystem of a new instance of the `Prefabs/Particles/Snowflakes` prefab childed to the battleentity and with a local position of the enemy party member's non world [CenterPos](CenterPos.md#centerpos).

## Delayed conditions's processing
As mentioned above, they are processed during the post action phase of DoAction for the enemy party member that was performing its action. Only [Freeze](BattleCondition/Freeze.md), [Numb](BattleCondition/Numb.md) and [Sleep](BattleCondition/Sleep.md) are supported.

### `Freeze` processing

- `frostbitep` position is set offscreen at (0.0, 999.0, 0.0)
- `frostbitep` is destroyed in 3.0 seconds
- The rest depends on the `freezeres`:
    - If it's 100 or above (it's immune), the `IceShatter` particles are played at the enemy party member's world's [CenterPos](CenterPos.md) and the condition isn't inflicted
    - Otherwise (it's not immune):
        - [Freeze](../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#freeze) is called on the entity
        - `mothicenormal` particles are played without sound at the enemy party member's world's [CenterPos](CenterPos.md) with a scale of (1.5, 1.5, 1.5)
        - [SetCondition](Conditions%20methods/SetCondition.md) is called with the `Freeze` condition on the enemy party member for 3 turns

### `Numb` processing
There's only logic if the `numbres` is below 100 (it's not immune). If it is:

- The `Numb` sound is played
- [SetCondition](Conditions%20methods/SetCondition.md) is called with the `Numb` condition on the enemy party member for 2 turns
- `isnumb` is set to true

### `Sleep` processing
There's only logic if the `sleepres` is below 100 (it's not immune). If it is:

- The `Sleep` sound is played
- DeathSmoke particles are played at the world's [CenterPos](CenterPos.md) of the enemy party member with a size of (2.0, 2.0, 2.0)
- [SetCondition](Conditions%20methods/SetCondition.md) is called with the `Numb` condition on the enemy party member for 2 turns
- `isasleep` is set to true
- battleentity.[animstate](../../Entities/EntityControl/Animations/animstate.md) is set to 14 (`Sleep`)
