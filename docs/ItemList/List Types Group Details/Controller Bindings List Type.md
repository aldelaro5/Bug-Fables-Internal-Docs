# Controller Bindings list type

Display the list of inputs of the game and their bound controller inputs. This is an unfinished version of an older implementation of the controller input rebinder that ended up unused and inaccessible under normal gameplay.

While there are several functionality left in the game in regards to rendering, a lot of it is incomplete and doesn't work properly even when forcing the ItemList to render. It was primarily intended as an older version of the controller input rebinder which used this list type to offer a way to rebind controller inputs. 

In 1.0.5, this list type was technically working, but it required the Use Controller settings from the [Settings List Type](Settings%20List%20Type.md) to be set to CUSTOM BINDINGS which was prevented by the input handling of PauseMenu. 1.1.0 replaced the window intended for this list type entirely by the new input rebinder which does not involve any list type and just UI code that acts like a wizard in PauseMenu.

Since the implementation is unfinished, its documentation is too spurious to define and it is recommended to view it as not working even outside of normal gameplay.

It did involved an UNUSED method called GetButtonSpriteD:

```cs
private static Sprite GetButtonSpriteD(int id)
```
This is semantically UNUSED because it's only called by the controller bindings listtype which is UNUSED. It's also likely broken.

Returns null. The `id` value does nothing.
