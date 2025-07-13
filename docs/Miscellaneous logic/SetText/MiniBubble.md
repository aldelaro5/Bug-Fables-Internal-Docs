# MiniBubble
A component meant to be programmatically added to a new GameObject using the SetUp static method:

```cs
public static MiniBubble SetUp(string text, EntityControl target, Vector3 pos, int sortingorder)
public static MiniBubble SetUp(string text, EntityControl target, Vector3 pos, int sortingorder, float timer)
```
The default value of `timer` on the first overload is 1.5.

This component is specific to the [minibubble](../../SetText/Individual%20commands/Minibubble.md) SetText command implementation, check its documentation to learn more.

The only public field is the `target` [EntityControl](../../Entities/EntityControl/EntityControl.md) that the minibubble refers to. Outside of the SetUp methods above, the only public method is DestroyThis which cleanly destroys the MiniBubble's GameObject.
