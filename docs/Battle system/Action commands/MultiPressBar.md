# MultiPressBar

TODO: identical setup than RandomPressBar

## `data` array

- `data[0]`: The lower bound of the starting x local position of `barmiddle` in percentage ???
- `data[1]`: The upper bound of the starting x local position of `barmiddle` in percentage ??? 
- `data[2]`: The x scale factor of the `barmiddle` in percentage ???

## [DoCommand](../DoCommand.md) Setup phase

- `presskey` is set to 4
- `buttons` is set to a new array with one element being the ButtonSprite of a new GameObject named `Button` which has SetUp called on it with the following:
    - buttonid: `presskey`
    - onlytype: -1
    - description: empty
    - position: (-0.8, 1.0, 10.0)
    - iconsise: (0.75, 0.75, 0.75)
    - sortorder: 2
    - parentobj: `GUICamera`
- internaldata is set to a new array of 2 elements:
    - 0: A random float between `data[0]` and `data[1]`
    - 1: 0.0
- `commandsprites` is set to a new array with 4 elements:
    - 0: The SpriteRenderer of a new UI object named `barbottom` childed to the `GUICamera` with a local position of (-7.0, 1.0, 10.0), scale of (5.0, 1.0, 1.0) using the `guisprites[58]` sprite with a sortorder of 0 and a color of clear (all transparent black)
    - 1: The SpriteRenderer of a new UI object named `barmiddle` childed to the `commandsprites[0]` with a local position of (`internaldata[0]` / 100.0, 0.0, 0.0), scale of (`data[2]` / 100.0, 1.0, 1.0) using the `guisprites[58]` sprite with a sortorder of 1 and a color of 60E275 (mostly clear green)
    - 2: The SpriteRenderer of a new UI object named `cursor` childed to the `commandsprites[0]` with a local position of (0.0, 0.25, 0.0), scale of (0.45, 0.1, 1.0) using the `cursorsprite[0]` sprite with a sortorder of 2 and with angles of (0.0, 0.0, 270.0)
    - 3: The SpriteRenderer of a new UI object named `barholder` childed to the `GUICamera` with a local position of (-7.0, 1.0, 10.0), scale of Vector3.one using the `guisprites[62]` sprite with a sortorder of 0

## [DoCommand](../DoCommand.md) Execution phase

