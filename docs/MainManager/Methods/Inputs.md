# Inputs
These methods deals with Inputs processing frequently involving InputIO.

## Checking any [controllers](../../InputIO/Controllers.md) input
The following methods checks that any out of a variety of controllers inputs are pressed / held.

```cs
private static bool AnyJoyKey(bool hold)
```
Returns true if Input.GetKeyDown returns true (Input.GetKey instead if `hold` is true) for any joystick related KeyCode, returns false otherwise. More information in the [Unity Input](../../InputIO/Controllers.md#unitys-input-api) documentation.

```cs
private static bool AnyJoyStick()
```
Returns true if InputIO.JoyStick returns anything other than 0.0 for any axis based [joystick input](../../InputIO/Inputs.md) (id from 0 to 3), returns false otherwise.

## Getting controller names
The following method allows to get all connected controllers's name:

```cs
public static string[] Controllers()
```
Return Input.GetJoystickNames(), but each element that has a length of 1 or lower are removed.

## GetJoystick
This method is called at the end of each Update when `basicload` is true ([LoadEverything](../Boot%20and%20reset%20process.md#loadeverything-part-12) got done) and its task is to update the [controllers](../../InputIO/Controllers.md#games-controller-input-pipeline) input pipeline configuration when necessary:

```cs
private void GetJoystick()
```
Most of the logic of this method is explained in the documentation linked above so it won't be repeated here, but there are a few details worthy to mention:

- If InputIO.`IsConsole` is true, the method always assumes controllers are enabled (it won't check if `usejoystick` is 0 which is DISABLED)
- The method updates the value of `joystick` which tells if a controller is in use. Here is a summare of how the value changes in this method:
    - If controllers are disabled (or InputIO.`IsConsole` is true), the value is set to false
    - If Input.anyKeyDown, MainManager.AnyJoyKey(false) and MainManager.AnyJoyStick all return false while InputIO.`IsConsole` is also false, the `joystick` value is set to false
    - If there is a controller bindings update, the value is set to true since it implies a controller was detected
- The method won't always update the MainManager.`joyid` value and it won't always call InputIO.GetJoyButtons. It only does this during a controller update which either requires that `forcecontrollerupdate` is true (can only happen after calling [ApplySettings](../../External%20data%20format/Config%20File.md#applysettings) in some conditions) or all of the following conditions are true:
    - `joystick` is false (controllers weren't being used)
    - InputIO.`IsConsole`, MainManager.AnyJoyKey(false) or MainManager.AnyJoyStick retrurns true
- On any controller update, `forcecontrollerupdate` is reset to false
- At the end of the method, Screen.sleepTimeout is set to NeverSleep if `joystick` is true (a controller is in use) and InputIO.`IsConsole` is false. The value is set to SystemSetting otherwise

## GetKey
The vast majority of the game doesn't check InputIO directly to query for inputs, but rather a method on MainManager called GetKey which features friendly input abstractions to simplify InputIO:

```cs
public static bool GetKey(int id)
public static bool GetKey(int id, bool hold)
```
It returns whether or not [input](../../InputIO/Inputs.md) id `id` was pressed on this frame. If `hold` is true, it also counts if the input has been pressed for more than a frame. The `id` supports [input aliases](../../InputIO/Inputs.md#input-aliases) and it is the only method in the game that supports them.

The overload without a `hold` parameter has its value default to false.

More specifically, the method processes all inputs and inputs aliases as described in the [input](../../InputIO/Inputs.md) documentation, but it uses the following rules:

- If [controllers](../../InputIO/Controllers.md) are enabled and `joystick` is true (a controller was in use by the end of the last Update), controllers inputs are always checked first and they have priority. In that case [Keyboard](../../InputIO/Keyboard.md) inputs are only checked when the input isn't detected on any controllers
- Any button inputs is checked via InputIO.GetKeyDown if `hold` is false or InputIO.GetKey if `hold` is true. However if [controllers](../../InputIO/Controllers.md) are checked, the method receives the `id` - 4 because it is required to allow InputIO to index the bound InputIO`joykeys` element
- Any axis inputs on [Keyboard](../../InputIO/Keyboard.md) are checked the same way than buttons ones (InputIO.GetKeyDown if `hold` is false or InputIO.GetKey if `hold` is true)
- Any axis inputs on [controllers](../../InputIO/Controllers.md) is checked via InputIO.JoyStick using all axises that can provide the input (so up/down checks either vertical axises and left/right checks either horizontal axises). Specifically, it checks that the axis value is towards the input and not be 0.0 However, if `hold` is false, the value must be more than 0.5 towards the input instead and the matching stick hold value (`stickholdx` for horixontal, `stickholdy` for vertical) must be true. These stick hold values are update on the last MainManager.[LateUpdate](../LateUpdate.md) and they are set to true if either axis in the pair has an absolute value lower than 0.5 (indicating the axises are NOT being held already). NOTE: There is a known issue where `stickholdx` and `stickholdy` don't indicate correctly if inputs were held, check the section below for more details

As for how the virtual axises inputs are implemented:

- -4: Calls this method where the `id` is from 0 to 9 using the same `hold` value. The return is true if any of these calls returns true
- -3: Calls this method where the `id` is from 4 to 9 using the same `hold` value. The return is true if any of these calls returns true
- -2: Essentially does the same checks described above for all axises inputs, but for [controllers](../../InputIO/Controllers.md), it only checks that the return of InputIO.JoyStick isn't 0.0 (the rest still applies like the stick hold value needing to be false)
- -1: The return is true if InputIO.anyKeyDown is true (InputIO.anyKey instead if `hold` is true)

### Issue with `stickholdx` and `stickholdy`
The check to know if an horizontal or vertical input is being held is incorrect because it doesn't account that the first 4 game [inputs](../../InputIO/Inputs.md) are the 4 cardinal directions unlike [joystick inputs](../../InputIO/Inputs.md#joystick-inputs) where they are pairs of horizontal and vertical axises.

This discrepancy means that if the player holds left on their joystick or d-pad, it will be seen as if they are holding the ENTIRE horizontal axis instead of specifically knowing that the "left" input is being held. This specific scenario causes problems with the [TappingKey](../../Battle%20system/Action%20commands/TappingKey.md) ActionCommand in the left/right mashing prompt mode. Specifically, if the player manages to flip flop their held direction from left to right or vice versa within a frame (which can be very doable depending on the controller), the game will not accept this as a non held input because no matter if it's right or left, the game combines both inputs as the axis being held. Since it can't distinguish which one is actually held, there's no way to know that the player is actually holding a DIFFERENT input than before.

The same principle applies with up/down and the vertical axis.

## Checking multiple MainManager.GetKey calls
The following methods calls MainManager.GetKey multiple times and aggregates the result.

```cs
public static int MultipleKeys(bool hold)
```
Returns the amount of calls to MainManager.GetKey that returns true using the same `hold` value where the id is each [input](../../InputIO/Inputs.md) id from 0 to 9.

```cs
public static bool AnyKeyButThis(int id, bool hold)
```
Returns if any calls to MainManager.GetKey returns true using the same `hold` value where the id is each [input](../../InputIO/Inputs.md) id from 0 to 9 with the exception of `id` which isn't checked.

## KeyHold system
The game has an implementation of an autorepeat system where an axis [input](../../InputIO/Inputs.md) (id from 0 to 3) is allowed to be automatically repeated every 7.0 frames (enforced by the `keyhold[1]` frame counter) as long as it is first held for 16.0 frames or more (enforced by the `keyhold[0]` frame counter) and keeps beind held after.

To use it, the following method needs to be called on every frame to update its state:

```cs
public static bool KeyHold(Directions direction)
```
It returns true if all of the following conditions are fufilled that indicates the input is allowed to be detected under the autorepeat system:

- MainManager.GetKey((int)`direction`, true) returns true (if it's not, the method just returns false)
- `keyhold[0]` is higher than 15.0 frames. If it's not, it's increased by `framestep` and the method returns false
- `keyhold[1]` is at least 7.0 frames. If this happens, `keyhold[1]` is reset to 0.0 and the method returns true. If this condition isn't fufilled, `keyhold[1]` is increased by `framestep` and the method returns false

To reset the state of KeyHold, ResetKeyHold needs to be called:

```cs
public static void ResetKeyHold()
```
If MainManager.GetKey with a hold of true returns false for any axis [input](../../InputIO/Inputs.md) (id from 0 to 3), both `keyhold[0]` and `keyhold[1]` values are reset to 0.0. This resets the state of the autorepeat system used by the KeyHold method.
