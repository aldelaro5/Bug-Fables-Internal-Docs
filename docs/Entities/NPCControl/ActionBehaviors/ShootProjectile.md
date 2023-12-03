# ShootProjectile
Attacks the player by shooting an OverworldProjectile which can, under the right conditions, start a battle with advantage. The specifics of the projectile logic is dependant on the [animid](../../../Enums%20and%20IDs/AnimIDs.md) and only a few have a logic defined for this behavior.

## Frequency meaning
None.

## An overview of OverworldProjectile
This component is only involved over the course of this behavior. It is meant to be created externally via a static method called NewProjectile defined in that component's MonoBehavior. It takes a lot of parameters to setup the projectile notably a `parent` which will be this NPCControl.

At a basic level, an OverworldProjectile is a sprite with a default trigger BoxCollider that has the ability to call NPCControl.[StartBattle](../Notable%20methods/StartBattle.md) with advantage on its OnTriggerStay when all of the following conditions are fufilled:

- The other collider is the player
- We aren't in a `pause`, `minipause`, `inevent` or `inbattle`
- The player isn't on `shield`
- This NPCControl entity isn't `iskill`
- This NPCControl `dizzytime` expired (this prevents encounters where the projectile hit the player, but the player stunned them before)

The projectile has a limited amount of screen time in frames, but if it touches the player (without encounter) or a `Ground` / `NoDigGround` layer, it gets destroyed with optional special effects.

As for the movement of the OverworldProjectile, it moves along a beizer curve with start / end points alongside the y position of the mid point. However, it is possible to send 0.1 or less for that last parameter to have it lerp in a straight line instead according to its screen time.

As for the rendering of of the projectile, it's a sprite index of the `Sprites/Misc/projectiles` sprites, but it's possible to send a negative number to specify an [item](../../../Enums%20and%20IDs/Items.md) id (the absolute value) to use as the sprite.

Notable traits of an OverworldProjectile GameObject:

- Its name is `proj`
- It is childed to the map
- A GameObject is childed to it with a SpriteRenderer called `tempproj`
- Its layer is 14 (`Sprite`)
- Its tag is `Projectile`

## DoBehavior
If `returntoheight` is true, the entity.`height` is set to a lerp from the existing one to its `initialheight` with a factor of 0.1.

If the entity is in a `forcemove`, [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity without smoothing and without changing the [animstate](../../EntityControl/Animations/animstate.md).

Finally, a ShootProjectile coroutine is started and set to `behaviorroutine`.

## ShootProjectile
This is a private coroutine specific to this behavior. It receives the behavior type and the coroutine call is assigned to `behaviorroutine` which allows the NPCControl to stop it if needed.

- [Emoticon](../../EntityControl/EntityControl%20Methods.md#emoticon) is called on the entity with id 2 (a red ! mark) for 60 frames
- If `projectiles` is null, it is initalised to a new List of OverworldProjectile
- All null elements in `projectiles` are removed
- [FaceTowards](../../EntityControl/EntityControl%20Methods.md#facetowards) is called on the entity to face the player
- From there, the projectile is fired depending on the entity.[animid](../../../Enums%20and%20IDs/AnimIDs.md#animids). See the section below for more details. If this is not an animid with a projectile logic, the following is done:
    - entity.[animstate](../../EntityControl/Animations/animstate.md) is set to entity.`basestate`
    - If we aren't `inrange`, entity.`overrideonlyflip` is set to false
    - A frame is yielded
    - [StopForceBehavior](../Notable%20methods/StopForceBehavior.md) is called
    - The coroutine is exited early with a yield break
- entity.[animstate](../../EntityControl/Animations/animstate.md) is set to entity.`basestate`
- If we aren't `inrange`, entity.`overrideonlyflip` is set to false
- A frame is yielded
- [StopForceBehavior](../Notable%20methods/StopForceBehavior.md) is called
- If we still aren't `inrange`, [Emoticon](../../EntityControl/EntityControl%20Methods.md#emoticon) is called on the entity with id 1 (? mark) for 60 frames

### Projectile logic
Here are the different logic available for each [animid](../../../Enums%20and%20IDs/AnimIDs.md).

#### `DeadLanderA`
If there's no `projectiles`:

- entity.[animstate](../../EntityControl/Animations/animstate.md) is set to 103 (shooting)
- A second is yielded
- All frames are yielded as long as we are in a `pause`, `minipause`, `inevent` or we are in a `battle` that is `inevent`
- If either `dizzytime` or `freezecooldown` haven't expired, this projectile logic ends early (but the coroutine continues)
- [FaceTowards](../../EntityControl/EntityControl%20Methods.md#facetowards) is called on the entity to face the player
- entity.`animstate` is set to 104 (going back to idle from shooting)
- A projectile gets added with this NPCControl as the parent:
    - sprite index 10 (honey projectile)
    - starting position at this NPCControl position - its right vector + (0.0, 1.25, 0.0)
    - target position being the player position undershot by 1.5 towards the direction of the player
    - No spin
    - y angles of the entity.`sprite` angle, but no x or z
    - size of uniform 0.75
    - `HoneyExplode` particles on the projectle destruction
    - y midpoint of the bezier curve of 2.0
    - time of 50.0 frames
    - shadow size multiplier of 0.25
- 0.5 seconds are yielded

No matter if there was projectiles or not, all frames are yielded as long as we are in a `pause`, `minipause`, `inevent` or we are in a `battle` that is `inevent` followed by an additional yield which ends this projectile.

#### `WaspBomber`
If there's no `projectiles`:

- entity.`animstate` is set to 101 (throwing)
- A projectile gets added with this NPCControl as the parent:
    - sprite index -30 (`SpicyBomb` [item](../../../Enums%20and%20IDs/Items.md) sprite)
    - starting position at this NPCControl position - its right vector + (0.25, 1.25, -0.1)
    - target position being the player position undershot by 1.5 towards the direction of the player
    - Spin of 20.0 in z (no x or y)
    - y angles of the entity.`sprite` angle, but no x or z
    - size of uniform 1.0
    - `explosionsmall` particles on the projectle destruction (also, if the player is less than 15.0 away on destruction, `Explosion` sound is played at 0.75 pitch followed by a ShakeScreen of 0.1 for 0.5 seconds without reset)
    - y midpoint of the bezier curve of 5.0
    - time of 80.0 frames
    - shadow size multiplier of 0.5
- a second is yielded
- entity.`animstate` is set to 0 (`Idle`)

No matter if there was projectiles or not, all frames are yielded as long as we are in a `pause`, `minipause`, `inevent` or we are in a `battle` that is `inevent` followed by an additional yield which ends this projectile.

#### `ToeBiter`
If there was at least 1 `projectiles`, all frames are yielded as long as we are in a `pause`, `minipause`, `inevent` or we are in a `battle` that is `inevent` followed by an additional yield which ends this projectile logic.

If there's no `projectiles`:

- entity.`animstate` is set to 100 (lowering upper limbs to the ground)
- 0.5 seconds are yielded
- entity.`animstate` is set to 101 (putting upper limbs in the air)
- If `dizzytime` or `freezecooldown` aren't expired, this projectile logic ends early, but the coroutine still continues
- All frames are yielded as long as the player isn't free (ignoring flying) or they are `digging`
- If `internaltransform[0]` doesn't exist, it is initialised to a `Prefabs/Objects/CrackedRock` prefab without its MeshCollider and Fader with a position of this NPCControl + (0.0, -2.0, 0.0), and a scale of 0.5 uniform
- `internaltransform[0]` is childed to the map
- If `dizzytime` or `freezecooldown` aren't expired, this projectile logic ends early, but the coroutine still continues
- From there, there is a loop that goes on for up to 11.0 frames, but counted by a local variable which only gets incremented towards the end of the loop by the game's frametime (meaning there may be more frames yielded during the loop, but it will stop after 11.0 frames are counted using the local variable):
    - `internaltransform[0]` position is set to a lerp from the existing one to (entity.`sprite` position + the normalised right vector of the entity + 0.3 in y) with a factor of the ratio of the amount of frames counted in this loop over 11.0. This basically moves the rock on a straight line to a position above the entity, but slightly towards them
    - If `dizzytime` or `freezecooldown` aren't expired, this projectile logic ends early, but before it is ends, all frames are yielded as long as we are in a `pause`, `minipause`, `inevent` or we are in a `battle` that is `inevent` followed by an additional yield
    - This is where the local frame counter is incremented by the game's frametime, but only if we aren't in a `pause`
    - A frame is yielded
- 0.25 seconds are yielded
- All frames are yielded as long as the player isn't free (ignoring flying)
- entity.`animstate` is set to 28 (TossItem)
- All frames are yielded as long as the player isn't free (ignoring flying)
- If `dizzytime` or `freezecooldown` aren't expired, this projectile logic ends early, but the coroutine still continues
- The `Toss3` sound is played at 0.9 volume
- A projectile gets added with this NPCControl as the parent:
    - sprite index 17 (A brown missile)
    - starting position at this `internaltransform[0]` position
    - target position being the player position undershot by 1.5 towards the direction of the player
    - No spin
    - y angles of the entity.`sprite` angle and z angle of 90.0, no x
    - size of uniform 0.5
    - `Rock` particles on the projectle destruction (will also lead to the rooting of the rock if it still exists followed by CrackRock being called on it. Additionally, if the player is 15.0 away, `RockBreak` sound is played at 0.75 pitch followed by a ShakeScreen of 0.1 for 0.5 seconds without reset)
    - y midpoint of the bezier curve of 3.0
    - time of 40.0 frames
    - shadow size multiplier of 0.25
- The projectile gets childed to `internaltransform[0]`. This is done so because the game hides the fact the rock isn't the projectile, but rather a brown missile is and its sprite gets disabled on the OverworldProjectile's Start because of the `Rock` particle on death parameter sent
- `internaltransform[0]` is set to null (OverworldProjectile takes control of it from now on)
- 0.5 seconds are yielded
- All frames are yielded as long as we are in a `pause`, `minipause`, `inevent` or we are in a `battle` that is `inevent` followed by an additional yield which ends this projectile

#### `SneilEnemy`
If there's no `projectiles`:

- entity.`animstate` is set to 100 (preparing to shoot)
- [FlipSpriteAngleAt](../../EntityControl/EntityControl%20Methods.md#flipspriteangleat) is called on the entity towards the player with offset (0.0, 90.0, 0.0)
- All frames are yielded as long as we are in a `pause`, `minipause`, `inevent` or we are in a `battle` that is `inevent`
- If `dizzytime` or `freezecooldown` aren't expired, this projectile logic ends early, but the coroutine still continues
- The `Lazer` sound is played on the entity
- [FlipSpriteAngleAt](../../EntityControl/EntityControl%20Methods.md#flipspriteangleat) is called on the entity towards the player with offset (0.0, 90.0, 0.0)
- A projectile gets added with this NPCControl as the parent:
    - sprite index 4 (A white lightning bolt)
    - starting position at this NPCControl position - its  right vector + 0.1 in y
    - target position being the player position undershot by 1.5 towards the direction of the player + 2.0 in y
    - No spin
    - y angles of the entity.`sprite` angle and z angle of 90.0, no x
    - size of uniform 0.75
    - No particles on destruction
    - y midpoint of the bezier curve of 0.0
    - time of 60.0 frames
    - shadow size multiplier of 0.25
- 0.75 seconds are yielded
- all frames are yielded as long as we are in a `pause`, `minipause`, `inevent` or we are in a `battle` that is `inevent` followed by an additional yield which ends this projectile.

Otherwise, if there was a projectile:

- entity.`animstate` is set to entity.`basestate`
- entity.`overrideflip` is set to false
- A frame is yielded
- [StopForceBehavior](../Notable%20methods/StopForceBehavior.md) is called
- The coroutine is ended early with a yield break

#### `BeeBot`
If there's less than 2 `projectiles`:

- entity.`animstate` is set to 100 (preparing to shoot)
- [FlipSpriteAngleAt](../../EntityControl/EntityControl%20Methods.md#flipspriteangleat) is called on the entity towards the player with offset (0.0, 90.0, 0.0)
- 0.5 seconds are yielded
- All frames are yielded as long as we are in a `pause`, `minipause`, `inevent` or we are in a `battle` that is `inevent`
- If `dizzytime` or `freezecooldown` aren't expired, this projectile logic ends early, but the coroutine still continues
- [FlipSpriteAngleAt](../../EntityControl/EntityControl%20Methods.md#flipspriteangleat) is called on the entity towards the player with offset (0.0, 90.0, 0.0)
- A projectile gets added with this NPCControl as the parent:
    - sprite index 10 (honey projectile)
    - starting position at this NPCControl position - its right vector + 1.25 in y
    - target position being the player position undershot by 1.5 towards the direction of the player
    - No spin
    - y angles of the entity.`sprite` angle, no x or z
    - size of uniform 0.75
    - `HoneyExplode` particles on destruction
    - y midpoint of the bezier curve of 2.0
    - time of 60.0 frames
    - shadow size multiplier of 0.25
- 0.3 seconds are yielded

No matter if there was less than 2 projectiles or not, all frames are yielded as long as we are in a `pause`, `minipause`, `inevent` or we are in a `battle` that is `inevent` followed by an additional yield which ends this projectile.

#### `Turret`
The same than `BeeBot` with a few changes:

- The first yield is 0.7 seconds instead of 0.5
- Right before the second FlipSpriteAngleAt, entity.`animstate` is set to 102 (shooting)
- The last yield is 0.4 seconds instead of 0.3

#### `WaspScout`
If there's no `projectiles`:

- entity.`animstate` is set to 100 (preparing to throw)
- [FaceTowards](../../EntityControl/EntityControl%20Methods.md#facetowards) is called on the entity to face the player
- 0.25 seconds are yielded
- entity.`animstate` is set to 102 (throwing)
- All frames are yielded as long as we are in a `pause`, `minipause`, `inevent` or we are in a `battle` that is `inevent`
- If `dizzytime` or `freezecooldown` aren't expired, this projectile logic ends early, but the coroutine still continues
- The `Toss` sound is played on the entity
- [FlipSpriteAngleAt](../../EntityControl/EntityControl%20Methods.md#flipspriteangleat) is called on the entity towards the player with offset (0.0, 90.0, 0.0)
- A projectile gets added with this NPCControl as the parent:
    - sprite index 2 (a brown kunai)
    - starting position at this NPCControl position - its right vector + 1.75 in y
    - target position being the player position undershot by 1.5 towards the direction of the player
    - No spin
    - y angles of the entity.`sprite` angle and -60.0 in z, no x
    - size of uniform 0.75
    - No particles on destruction
    - y midpoint of the bezier curve of 0.0
    - time of 45.0 frames
    - shadow size multiplier of 0.25
- 0.75 seconds are yielded

No matter if there was projectiles or not, all frames are yielded as long as we are in a `pause`, `minipause`, `inevent` or we are in a `battle` that is `inevent` followed by an additional yield which ends this projectile.

#### `LeafbugArcher`
The same than `WaspScout`, but with a few changes:

- Before facing towards the entity, the `Rope1` sound is played on the entity
- The first yield is 0.5 seconds instead of 0.25
- The projectile sprite index is 12 (a purple blunt projectile) instead of 2 (a brown kunai)
- The offset to the projectile starting position is (1.0, 1.0, -0.1) instead of just 1.75 in y
- The z angle of the projectile is -90.0 instead of -60.0

#### `Bandit`
If there's no `projectiles`:

- entity.`animstate` is set to 100 (preparing to throw)
- [FaceTowards](../../EntityControl/EntityControl%20Methods.md#facetowards) is called on the entity to face the player
- 0.5 seconds are yielded
- All frames are yielded as long as we are in a `pause`, `minipause`, `inevent` or we are in a `battle` that is `inevent`
- If `dizzytime` or `freezecooldown` aren't expired, this projectile logic ends early, but the coroutine still continues
- The `Spin3` sound is played on the entity
- [FlipSpriteAngleAt](../../EntityControl/EntityControl%20Methods.md#flipspriteangleat) is called on the entity towards the player with offset (0.0, 90.0, 0.0)
- A projectile gets added with this NPCControl as the parent:
    - sprite index 3 (a shuriken shaped object)
    - starting position at this NPCControl position - its right vector + 1.0 in y
    - target position being the player position undershot by 2.0 towards the direction of the player + 1.25 in y
    - z spin of 20.0
    - y angles of the entity.`sprite` angle and 90.0 in z, no x
    - size of uniform 0.75
    - No particles on destruction
    - y midpoint of the bezier curve of 1.25
    - time of 60.0 frames
    - shadow size multiplier of 0.25
- 0.75 seconds are yielded

No matter if there was projectiles or not, all frames are yielded as long as we are in a `pause`, `minipause`, `inevent` or we are in a `battle` that is `inevent` followed by an additional yield which ends this projectile.

#### `ChomperBrute`
The same than `Bandit`, but with a few changes:

- Before the entity.`animstate` is set to 100 (which is opening their mouth), the `Clomp` sound is played on the entity
- After the first yields when in many kinds of pause, entity.`animstate` is set to 102 (biting) followed by a yield of 0.2 seconds
- Before the FlipSpriteAngleAt, the sound played on the entity is `PingShot` instead of `Toss`
- The projectile sprite index is -38 (`ChomperSeed` [item](../../../Enums%20and%20IDs/Items.md))
- The offset to apply to the projectile starting position is 0.2 in y instead of 0.1

#### `WildChomper`
The exact same logic than `ChomperBrute` (animstate 100 is preparing to shoot while animstate 102 is shooting).
