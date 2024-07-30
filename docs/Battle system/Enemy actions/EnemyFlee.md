# EnemyFlee
A coroutine mode for enemy actions to have an enemy party member flee the battle. This will set their `fled` to true and `enemyfled` to true alongside an animation to move them offscreen.

The coroutine should be stored in `tryenemyheal` as it will be set to null when completed so the caller can yield on it.

```cs
private IEnumerator EnemyFlee(int walkstate, bool afterimage, int enemyid)
```

## Parameters

- `walkstate`: The animstate to use when the enemy party member
- `afterimage`: Whether or not to enable the enemy party member's `trail` during the move
- `enemyid`: The `enemydata` index of the enemy party member that will flee

## Procedure

- `Scuttle2` sound plays on loop using `sounds[9]`
- animstate set to the sent walkstate
- `flip` set to true
- [Jump](../../Entities/EntityControl/EntityControl%20Methods.md#jump) called
- Yield for 0.4 seconds
- `sounds[9]` pitch set to 1.25
- Yield all frames until the enemy party member's `onground` is true
- The enemy party member's `trail` set to afterimage
- `Flee` sound plays
- `sounds[9]` stop with 0.001 delay
- Over the course of 50.0 frames, the enemy party member's position changes to offscreen at (15.0, 0.0, 0.0) via a lerp. Before each frame yield, the animstate is set to the sent walkstate
- The enemy party member's `fled` set to true
- `tryenemyheal` set to null informing the caller that the coroutine completed
- `enemyfled` set to true
- `sounds[9]` stop with 1.0 delay
