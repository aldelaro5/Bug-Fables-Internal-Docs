# JumpTo
This is a coroutine that makes all `playerdata` and `map`.`tempfollowers` entities go to a certain position starting from the player position using a scripted jump animation procedure using a height and multiplier parameters. It is meant to be stored in `jumproutine` as it is set to null when completed so the caller can yield on it.

There are 2 overloads available:

This one performs the whole logic:

```cs
public IEnumerator JumpTo(Vector3 position, float height, float multiplier)
```

This one calls the above where multiplier is 1.0, but the call is stored in `jumproutine` so the caller doesn't need to set it and after the call, a frame is yielded:

```cs
public IEnumerator JumpTo(Vector3 position, float height)
```

## Parameters

- `position`: The position to place the player by the end of the jump animation
- `height`: The ymax of the BeizierCurve3 used for the jump animation
- `multiplier`: A factor that can tune how long the animation takes. See below for details on how its value changes the amount of frames as it's complex

## Procedure
This will be described by sections.

### Setup

- [CancelAction](Actions/CancelAction.md) called
- We enter an instance.`minipause`
- instance.`overridefollower` set to true
- The player position, instance.`camoffset`, `MainCam` position and instance.`camspeed` are saved as they will be used later
- If we're not instance.`inevent`, the camera will be fixed in place:
    - instance.`camspeed` set to 0.075
    - instance.`camtarget` set to null
    - instance.`camtargetpos` set to null
- All `playerdata` have their entity.`rigid` velocity zeroed out and their position set to the player + (0.0, 0.0, 0.1 * their index). The z offset is to prevent z fighting
- All `map`.`tempfollowers` have their entity.`rigid` velocity zeroed out and their position set to the player + (0.0, 0.0, 0.1 * their index). The z offset is to prevent z fighting
- All `playerdata` and `map`.`tempfollowers` are selected the be the entities moved during the animations and are stored in a local list
- All selected entities have their `rigid` gravity disabled and placed in kinematic mode

### Jumping animation
This section contains the animation logic. However, it should be noted that it contains a significant amount of arbitrary and illogical math.

First, a number is calculated that will be used to scale the animation through time:

(45.0 * `X` - (`Y`)) / multiplier

Where:

- `X`: The amount selected entities
- `Y`: Depends on `X`:
    - \> 2: The value is `Y` ^ 3.0
    - <= 2: The value is -20.0
- multiplier: The parameter sent to the coroutine

We'll call the result "scaler". The total amount of frames the animation will take is scaler - 10.0. Each frames is yielded and counted locally. We'll call this frame counter "t".

Before each frame yield, this is what happens:

- If we're not instance.`inevent`:
    - `MainCam` position is lerped from the value before this coroutine to the sent position parameter with a factor of t / (scaler / 1.5)
    - instance.`camoffset` is set to the value before this coroutine + -`CamDir`.forward.normalized * `X` + (0.0, `X`, 0.0) where `X` is (0.5 - Abs(t / scaler - 0.5)) * 2.0 clamped from 0.0 to 0.75
- For each selected entity (their index in the local list will be refered to as "i"):
    - local position set to a BeizierCurve3:
        - From: The player position before this coroutine
        - To: The position parameter + `CamDir`.forward * (i / 10.0) which prevents z fighting
        - ymax: The height parameter
        - Factor: t / (scaler / 2.0) - i * (1.0 / the amount of selected entities) clamped from 0.0 to 1.0
    - An AnimationClip plays on the entity's `anim` depending if they are `onground`:
        - If they are, the clip is `Idle`
        - If they aren't, the clip is `Fall`

### Cleanup

- All selected entities have the following happen to their `rigid`:
    - velocity zeroed out
    - gravity enabled
    - placed out of kinematic mode
- Yield for a frame
- If we're not instance.`inevent`, the camera and game state is restored:
    - instance.`camoffset` set to the value before this coroutine
    - instance.`camtarget` set to the player
    - instance.`camtargetpos` set to null
    - instance.`camspeed` set to the value before this coroutine
    - We exit the instance.`minipause`
    - instance.`overridefollower` set to false
- `jumproutine` is set to null
