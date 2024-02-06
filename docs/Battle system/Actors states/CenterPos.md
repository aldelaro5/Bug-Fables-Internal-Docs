# CenterPos
A method that returns an intuitive center position to render effects on an actor.

```cs
private Vector3 CenterPos(MainManager.BattleData entity, bool world)
```

## Parameters

- `entity`: The actor to get the center position of
- `world`: Whether or not to add `entity`.battleentity's position to the final result

### Procedure

- If the actor's `position` is `Underground`, return Vector3.up with entity.battleentity's position added if world is true
- Otherwise, return entity.`cursoroffset` + (0.0, entity.battleentity.`height` - 1.0, 0.0) with entity.battleentity's position added if world is true