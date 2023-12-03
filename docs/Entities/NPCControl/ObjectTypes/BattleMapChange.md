# BattleMapChange
A zone that causes a change of the map.`battlemap`.

> NOTE: While the logic for this object type is technically present, it is dead code because it's impossible to reach it. This means this can be considered as UNUSED even if it technically functions. Its usage isn't recommended.

## Data Arrays
- `data[0]`: The battle map id to change to

## Setup
A few adjustements occurs:
- The entity.`alwaysactive` is set to true
- The gameObject's isStatic is set to true
- The entity.`rigid` is placed in kinematic mode without gravity
- `nointeract` is set to true
- The layer is set to 0 (default)
- The `scol` is disabled

## Update
If the `timer` hasn't expired yet, it is decremented by the game's frametime clamped from 0.0 to infinity. Otherwise, if it is 0.0 and the entity isn't `dead`, a [Death](../../EntityControl/Notable%20methods/Death.md) coroutine is started with the entity.

## OnTriggerStay
There is dead code here where it would have changed the map's `battlemap` to the one whose id is `data[0]`, but it's impossible to reach this logic because it always lacks the proper conditions.
