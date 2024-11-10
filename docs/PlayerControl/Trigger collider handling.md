# Trigger colliders handing
PlayerControl has an OnTriggerStay and OnTriggerExit. They only react to collision whose GameObject have a specific tag.

## OnTriggerStay
What happens here depends on the collider's tag (if it's not mentioned here, nothing happens).

### `Pusher` or `PPusher`
Only applies if the player isn't `flying` otherwise, nothing happens:

- entity.`onground` is set to false
- If entity.`rigid` y velocity is above 0.0, it is set to 0.0
- MainManager.PushAway is called to push the player away from the collider's transform by 0.05 with the direction being from the collider's transform to the player

### `Respawn`

- If the collider's GameOject has an [NPCControl](../Entities/NPCControl/NPCControl.md) with a `vectordata[0]`.magnitude above 0.1 (meaning this is a [SetPlayerRespawn](../Entities/NPCControl/ObjectTypes/SetPlayerRespawn.md) and its respawn position is defined):
    - `lastpos` is set to the NPCControl's `vectordata[0]` which is the respawn point
- Otherwise (this is a respawner, but not a SetPlayerRespawn), if entity is `onground`:
    - `respawncount` is increased by `framestep`
    - If `respawncount` reaches above 15.0 (so more than 15.0 frames in that collider), it is reset to 0.0 with `lastpos` set to the current player position

### `Conveyor`

- If the `conveyor` is the other collider's transform:
    - If MainManager.FreePlayer is true (not in a `pause`, `minipause`, `inevent`, the [message](../SetText/Notable%20states.md#message) lock is released and the player isn't `digging` or `flying`), the player position is increased by the `conveyor`'s StaticModelAnim's `conveyor` * `framestep`. Basically, the player will move towards the direction set in the StaticModelAnim as long as it keeps colliding with it
- Otherwise (`conveyor` is null):
    - `conveyor` is set to the other collider's GameObject's StaticModelAnim

### `KeepDig`
The only thing that happens is `keepdig` is set to 5.0.

## OnTriggerExit
What happens here depends on the collider's tag (if it's not part of the following, nothing happens):

- `Conveyor`: `conveyor` is set to null
- `KeepDig`: `keepdig` is reset to 0.0
- `Respawn`: `respawncount` is reset to 0.0
