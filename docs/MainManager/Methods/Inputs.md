# Inputs
These methods deals with Inputs processing frequently involving InputIO. TODO: Document InputIO is needed to document most of these.

## Inputs

```cs
private static bool AnyJoyKey(bool hold)
```
TODO: Involves InputIO logic that's not documented yet

```cs
private static bool AnyJoyStick()
```
TODO: Involves InputIO logic that's not documented yet

```cs
private void GetJoystick()
```
TODO: Involves InputIO logic that's not documented yet

```cs
public static bool KeyHold(Directions direction)
```
TODO: Involves InputIO logic that's not documented yet

```cs
public static void ResetKeyHold()
```
TODO: Involves InputIO logic that's not documented yet

```cs
public static bool GetKey(int id)
public static bool GetKey(int id, bool hold)
```
TODO: Involves InputIO logic that's not documented yet

```cs
public static int MultipleKeys(bool hold)
```
TODO: Involves InputIO logic that's not documented yet

```cs
public static string[] Controllers()
```
Return Input.GetJoystickNames(), but each element that has a length of 1 or lower are removed.

```cs
public static bool AnyKeyButThis(int id, bool hold)
```
TODO: Involves InputIO logic that's not documented yet
