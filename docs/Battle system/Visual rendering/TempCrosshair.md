# TempCrossfair
A method that renders a crosshair's ring that periodically spins over an enemy party member's physical position + (0.0, 1.0, 0.0) or its world [CenterPos](../Actors%20states/CenterPos.md) and returns its transform.

```cs
private Transform TempCrosshair(MainManager.BattleData target, bool groundpos)
```

## Parameters

- `target`: The enemy party member to render the crosshair on
- `groundpos`: If it's false, it will use the world [CenterPos](../Actors%20states/CenterPos.md) to determine where to render the crosshair and if it's true, it will use its physical position + (0.0, 1.0, 0.0) instead

## Procedure

- A new sprite object is created with name `tempsprite` chiled to the `battlemap` at the position determined by `groundpos` using the sprite `guisprites[93]` (a crosshair's ring) on layer 15 (`3DUI`) with the `spritedefaultunity` material
- A SpinAround is added to the sprite object with an `itself` of (0.0, 0.0, 7.5)
- The transform of the sprite object is returned
