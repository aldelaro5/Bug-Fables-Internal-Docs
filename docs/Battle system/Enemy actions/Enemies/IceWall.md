# `IceWall`

## Move selection
1 moves is possible:

1. Decrements their `hp` and dies if it reaches 0 after

Move 1 is always used.

## Move 1 - `hp` decrements
Decrements their `hp` and dies if it reaches 0 after. No damages are dealt.

### Logic sequence

- `hp` is decremented
- If `hp` is 0 or below, StartDeath is called on this enemy which sets their `deathroutine` to a new [Death](../../../Entities/EntityControl/Notable%20methods/Death.md) calle with activatekill
