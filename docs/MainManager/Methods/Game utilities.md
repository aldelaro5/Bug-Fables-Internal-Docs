# General game utilities
These methods deals with the general game state and game runtime.

## Framerate
These methods deals with the framerate of the game.

```cs
public static float TieFramerate(float value)
```
Returns `value` * Time.smoothDeltaTime * 60.0. This is intended to be used to count game frames or to scale events happening over time. A `value` of 1.0 returns the actual smoothed out frametime while a `value` lower than 1.0 returns a fraction of it.

NOTE: The multiplication by 60.0 implies that a return value of 1.0 means that exactly 1/60th of a second passed since the last couple of frames on average. It strongly assumes that the target FPS is 60 so anything higher will result in the value being lower. The game was NOT designed to handle this and will result in a lot of events that happens over time being much slower than normal due to this strong FPS dependency.

```cs
public static bool FrameDifference(int frames)
```
Returns true if Time.frameCount % the game's framerate / 60.0 ceiled is 0. The `frames` parameter does nothing.

In practice, this will always return true with vsync set to OFF. If vsync is set to ON, the amount of time true is returned varies and it depends on the monitor's refresh rate:

- 60hz and lower: true is always returned
- Between 60hz and 120hz exclusive: true is returned every 2 frames, false otherwise
- Between 120hz and 180hz exclusive: true is returned every 3 frames, false otherwise
- etc... (keep adding 60hz to the bounds to decrease the frequency at which true is returned)

## Game state
These methods deals with the general state of the game.

```cs
public static Maps CurrentMap()
```
Returns `map`.`mapid`.

```cs
public static bool IsPaused()
```
Returns true if any of the following conditions is true (false is returned otherwise):

- The game is in a `minipause`
- The game is in a `pause`
- The game is `inevent`
- A `battle` is in progress and it is `inevent`

```cs
public static int CrystalBerryAmmount()
```
Returns the amount of `crystalbflags` that are true.

## Condition checking
The following method is a general purpose condition checker commonly used to check if something may exists or not.

```cs
public static bool CheckIfCanExist(int[] requires, int[] limit, int regionalflag)
```
Returns false if all of the following conditions are satisfied  (true otherwise indicating the conditions are NOT satisifed):

- Any [flag](../../Flags%20arrays/flags.md) slots that matches the absolute value of any `limit` elements of -2 or lower must be false
- If `requires` is considered not empty, all flags slots specified in `requires` must be true. The array is considered not empty if it is not null, not empty and its first element isn't negative
- If `limit` is considered not empty, any flags slots specified in `limit` must be false. The array is considered not empty if it is not null, not empty and its first element isn't negative
- If `regionalflag` applies, the [regionalflag](../../Flags%20arrays/Regionalflags.md) slot `regionalflag` must be false. The condition applies only if `regionalflag` isn't negative and `map` isn't null (meaning the map isn't being loaded)

This method implements the backing logic of a ConditionChecker and NPCControl's existence logic. NOTE: The return value is inverted: false if conditions are satisifed, true if conditions are NOT satisfied.
