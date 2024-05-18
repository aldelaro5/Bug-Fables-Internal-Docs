# `ClearBomb`
This item is a toss item and as such, parts of its logic are based off the [PeebleToss](../Skills/PeebleToss.md) action.

## Object movement
Since this targets everyone, the object will move to Vector3.zero with a ymax of 5.0 instead.

## Item logic

- The object is destroyed
- `checkingdead` is set to a new ClearBombEffect coroutine (see the section below for details)
- Yield until `checkingdead` is null

### ClearBombEffect
This is a parameterless coroutine specifically made for this action. It sets `checkingdead` to null once completed:

- `IceBreak` sound plays at 0.6 volume
- `impactsmoke` particles plays at Vector3.zero
- Yield for 0.3 seconds
- `impactsmoke` particles plays at (-3.0, 0.0, 0.0)
- Yield for 0.3 seconds
- `impactsmoke` particles plays at (3.0, 0.0, 0.0)
- Yield for 0.3 seconds
- [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) is called on each enemy party members
- [ClearStatus](../../Actors%20states/Conditions%20methods/ClearStatus.md) is called on each player party members
- `checkingdead` is set to null signaling that the coroutine is done
