# IsInRadius
This is a method that is primarily used in [DamageWithinPos](DamageWithinPos.md), but also in some [player actions](../Player%20actions/Player%20actions.md) to check if an enemy party member's physical position (or world [CenterPos](../Actors%20states/CenterPos.md)) falls within a radius of a point.

```cs
private bool IsInRadius(Vector3 pos, MainManager.BattleData entity, float radius, bool getair, bool xonly)
```

## Parameters

- `pos`: The point to check if the enemy party member is within a radius of
- `entity`: The enemy party member that must will be checked to be within a radius of pos
- `radius`: The distance to check if the enemy party member falls within pos
- `getair`: If true, it forces to check only for the distance between pos and the world [CenterPos](../Actors%20states/CenterPos.md) of the enemy party member if its [position](../Actors%20states/BattlePosition.md) is `Flying`. If it's false, it will check both the world CenterPos and physical position. NOTE: The game never calls the method where this value is false under normal gameplay
- `xonly`: Only check for the x distance by zeroing out the y and z components of both points before comparising their distance

## Procedure

- Both the physical position and the world [CenterPos](../Actors%20states/CenterPos.md) of the enemy party member are gathered in 2 local Vector3
- If `xonly` is true, both local Vector3 and pos have their y and z components zeroed out
- If getair is true and entity's [position](../Actors%20states/BattlePosition.md) is `Flying`, the result returned will be whether or not the distance between pos (or its x component) and the physical position of the enemy party member (or its x component) is lower than radius
- Otherwise, the result is whether or not either the world [CenterPos](../Actors%20states/CenterPos.md) or physical position of the enemy party member has a distance from pos less than radius (again, if `xonly` is true, only the x components are compared).
