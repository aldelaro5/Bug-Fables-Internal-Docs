# PlayerControl FixedUpdate
What's contained here is miscellaneous updates, but there's one of 3 update types that can happen here (the first one is chosen):

- `startdig` is true while the player isn't in a `submarine` meaning the `digging` state needs to be updated
- The player is in a `submarine` and entity.`model` exists meaning the `submarine` state needs to be updated
- Neither of the cases above applies meaning the entity.`sprite` local position needs to be progressively restored

## `digging` update

- entity.`sprite` local y position is lerped from the existing one to -2.0 with a factor of 0.03
- `lockkeys` is set to true if the player isn't fully `digging` yet and to false if it is. This essentially locks most inputs processing from `startdig` to fully `digging`
- If entity.`sprite` local y position reached -1.5 or lower:
    - `digging` is set to true
- Otherwise (still in `startdig`):
    - [StopMoving](../Entities/EntityControl/EntityControl%20Methods.md#stopmoving) called on the entity with its current [animstate](../Entities/EntityControl/Animations/animstate.md) as the targetstate
- If no sound is playing on entity.`sound`:
    - If the player is fully `digging`, the `Digging` sound is played on loop on the entity with a volume of 0.8
- Otherwise (a sound is already playing on entity.`sound`):
    - entity.`sound`.pitch is set to a value that depends on `delta`.magnitude (meaning if there's enough movement):
        - \> 0.1: 1.1 pitch
        - \<= 0.1: 0.95 pitch

## `submarine` update

- If we're not in instance.`inevent`, entity.`emoticonoffset` is set to Vector3.up
- The entity gets some adjustements:
    - `flip` is set to false
    - `overrideflip` set to true
    - `sprite` angles zeroed out
    - `sprite` scale set to Vector3.zero
    - `model` local y position set to a lerp from the existing one to Sin(Time.time * 3.0) * 0.15 (unless `digging` is true where it's -0.5 instead) with a factor of 0.05
    - `model`.`extra[1]` (the telescope) local position is set to (0.575, 0.0, `Z`) where `Z` is determined by a lerp from the existing local z position to 0.5 (4.0 instead if `digging` is true) with a factor of 0.025 (0.5 is actually the normal position of where the telescope should be)
    - `model`.`extra[0]` (the propeller) is set to rotate in x by 15.0 * entity.`rigid` velocity magnitude
    - `model` angles are updated:
        - x: Always -90.0
        - y: LerpAngle from the existing y angle to entity.`detect` y angle - 90.0 with a factor of 0.1 (1.0 instead if instance.`inevent`). Essentially, it aligns the y angles to where the player is heading with a bit of delay unless we are instance.`inevent` where it instantly aligns
        - z: Sin(Time.time * 3.0) * (entity.`rigid` velocity magnitude / 2.0) which makes the submarine wave slightly periodically

## entity.`sprite` local position restore
The only thing that happens in this update is the entity.`sprite` local position is lerped from the existing one to Vector3.zero with a factor of 0.17.
