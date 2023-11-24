# ChargeAndAttack
The same than [ChasePlayer](ChasePlayer.md), but also attacks the player (with `attacking` set to true during the attack) where the actions depends on the [AnimID](../../../Enums%20and%20IDs/AnimIDs.md) when getting closer than a configurable threshold. Only a few animIds have an attack logic defined for this behavior.

## Frequency meaning
The minimum distance to the player at which no attacks will occur. If the distance falls below this number, a ChargeAndAttack coroutine will be started. NOTE: if it is above 10.0 or negative, the value is overriden to 2.0.

## DoBehavior
The logic starts with the same than [ChasePlayer](ChasePlayer.md).

After, if the distance between the NPCControl and the player is less than the frequency (after overriding it as described above), `behaviorroutine` is set to a ChargeAndAttack coroutine that is started.

## ChargeAndAttack
This is a private coroutine specific to this behavior. The call is assigned to `behaviorroutine` which allows the NPCControl to stop it if needed.

- [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity
- entity.`overrideanim` is set to true
- From there, the charge and attack is done depending on entity.[animid](../../../Enums%20and%20IDs/AnimIDs.md#animids). See the section below for more details (it is possible we exit the coroutine early due to a battle being started).
- If we are in a `pause`, `minipause`, `inevent` or we are in a battle and the battle is `inevent`, all frames are yielded until that is no longer the case
- If we aren't `inrange`, [Emoticon](../../EntityControl/EntityControl%20Methods.md#emoticon) is called on the entity with id 1 (? mark) for 60 frames
- entity.`overrideanim` is set to false
- `attacking` is set to false
- [StopForceBehavior](../Notable%20methods/StopForceBehavior.md) is called

### Charge and attack logic
These are the different attacks an NPCControl can do with this behavior, but only 4 [animid](../../../Enums%20and%20IDs/AnimIDs.md) are supported.

#### `LeafbugClubber`
- The entity.[animstate](../../EntityControl/Animations/animstate.md) is set to 100 (preparing to attack with his club)
- The `Toss3` sound is played on the entity with normal volume and 1.1 pitch
- 0.2 seconds are yielded
- All frames are yielded as long as we are in a `pause`, `minipause`, `inevent` or we are in a `battle` that is `inevent`
- entity.`animstate` is set to 102 (lowering his club suddenly)
- `attacking` is set to true
- The `Toss7` sound is played on the entity
- 0.1 seconds are yielded
- All frames are yielded as long as we are in a `pause`, `minipause`, `inevent` or we are in a `battle` that is `inevent`
- If the distance between the NPCControl and the player is less than 2.0:
  - NPCControl.[StartBattle](../Notable%20methods/StartBattle.md) is called
  - entity.`overrideanim` is set to false
  - [StopForceBehavior](../Notable%20methods/StopForceBehavior.md) is called
  - The coroutine ends early with a yield break
- Otherwise, 0.5 seconds are yielded
- entity.`animstate` is set to 0 (`Idle`)

#### `FlyTrap`
The same than `LeafbugClubber`, but with the difference that instead of the `Toss3` and `Toss7` sounds being used, they are `Chew` and `Bite` respectively at normal volume and pitch. The 100 animstate for this animid is opening his mouth and the 102 one is closing it.

#### `Sandworm`
- entity.`digging` is set to false
- entity.`digtime` is set to 0.0
- entity.`overridejump` is set to false
- entity.`animstate` is set to 100 (opening his jaw)
- A frame is yielded
- entity.`sprite` scale is set to entity.`startscale`
- If entity.`digpart[1]` exists (the digging particles), they are moved offscreen at (0.0, -9999.0, 0.0)
- entity.`overrideflip` is set to true
- If `dirtcd` expired, `DirtExplodeLight` are played at the NPCControl position for 1 seconds with 0.75 uniform scaling
- `dirtcd` is set to 30.0
- [FlipSpriteAngleAt](../../EntityControl/EntityControl%20Methods.md#flipspriteangleat) is called on the entity with the player at the position and (0.0, 90.0, 0.0) as the offset
- From there, there is a loop that goes on for up to 40.0 frames, but counted by a local variable which only gets incremented towards the end of the loop by the game's frametime (meaning there may be more frames yielded during the loop, but it will stop after 40.0 frames are counted using the local variable):
  - The position is set to be on a quadratic Bezier curve with the following:
    - The start is the NPCControl position before this attack started
    - The end is where the player is, but undershot by 1.5 from the direction of the NPCControl. This point is limited to be in a radius where the center is entity.`startpos` and the radius being `radiuslimit`
    - The y of the midpoint is 4.0
    - The t factor is the ratio of the current amount of frames ellpsed since the start of the loop over 40.0. This essentially means the curve will last for a total of 40.0 counted frames and each position is interpolated over the course of those 40.0 frames
  - `attacking` is set to true
  - If the distance between the player and this NPCControl is less than 1.5 and we aren't in a `minipause`:
    - NPCControl.[StartBattle](../Notable%20methods/StartBattle.md) is called
    - entity.`overrideanim` is set to false
    - [StopForceBehavior](../Notable%20methods/StopForceBehavior.md) is called
    - entity.`overrideflip` is set to false
    - This coroutines ends early with a yield break
  - [DetectDirection](../../EntityControl/EntityControl%20Methods.md#detectdirection) is called with the NPCControl position - the normalised direction from the player to this NPCControl
  - If entity.`hitwall` the loop is exited early (but the coroutine still continues). Essentially, it prevents movement when the entity.`detect` goes off
  - Otherwise, a frame is yielded
  - This is where the local variable controling this loop is incremented by the game's frametime. Only this increment counts (no other yields counts)
  - All frames are yielded as long as we are in a `pause`, `minipause`, `inevent` or we are in a `battle` that is `inevent`
- When the loop is done, if `dirtcd` expired, `DirtExplodeLight` are played at the NPCControl position for 1 seconds with 0.75 uniform scaling
- `dirtcd` is set to 30.0
- entity.`overrideflip` is set to false
- If we are `inrange`, entity.`digging` is set to true and entity.`digtime` is set to 100.0

#### `Underling`
The same than `Sandworm`, but just before `attacking` is set to true in the movement loop, entity.`sprite` z angle is set to a lerp between 0.0 and 180.0 where the factor is the ratio of the amount of frames the loop counter over 40.0. This makes it rotate with a roll along the movement. Animstate 100 is instead its head pointing upward.
