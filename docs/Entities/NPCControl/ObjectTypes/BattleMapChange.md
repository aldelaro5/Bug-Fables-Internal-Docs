TODO: There is a high chance this is effectively unused because of its OnTriggerStay being the only useful logic, but it's actually dead code ???

# BattleMapChange
A loading zone for changing the battle map ???

## Data Arrays


## Setup
A few adjustements occurs:
- The entity's `alwaysactive` is set to true
- The gameObject's isStatic is set to true
- The entity's `rigid` is placed in kinematic mode without gravity
- `nointeract` is set to true
- The layer is set to 0 (default)
- The `scol` is disabled

## Update
If the `timer` hasn't expired yet, it is decremented by the game's frametime clamped from 0.0 to infinity. Otherwise, if it is 0.0 and the entity isn't `dead`, a [Death](../../EntityControl/Notable%20methods/Death.md) coroutine is started with the entity.

## OnTriggerStay
There is dead code here where it would have changed the map's `battlemap` to the one whose id is `data[0]`, but it's impossible to reach this logic because it always lacks the proper conditions.
