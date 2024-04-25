# LongRandomBar


## `data` array

- `data[0]`: The length of `commandsprites` will be 3 + this value ??? Also, the length of internaldata ???
- `data[1]`: Determines the first internaldata ???

## [DoCommand](../DoCommand.md) Setup phase

- `presskey` is set to 4
- `successfulchain` is set to 0
- `buttons` is set to a new array with one element being the ButtonSprite of a new GameObject named `Button` which has SetUp called on it with the following:
    - buttonid: `presskey`
    - onlytype: -1
    - description: empty
    - position: (-5.2, 2.35, 10.0)
    - iconsise: (0.75, 0.75, 0.75)
    - sortorder: 2
    - parentobj: `GUICamera`
- `commandsprites` is set to a new array with 3 + `data[0]` elements, here are the first 3 elements:
    - 0: The SpriteRenderer of a new UI object named `barbottom` childed to the `GUICamera` with a local position of (-4.45, 2.4, 10.0), scale of Vector3.one using the `guisprites[81]` sprite with a sortorder of 0
    - 1: The SpriteRenderer of a new UI object named `barslide` childed to the `commandsprites[0]` with a local position of Vector3.zero, scale of (0.0, 1.0, 1.0) using the `guisprites[82]` sprite with a sortorder of 1 and a color of pure yellow
    - 2: The SpriteRenderer of a new UI object named `cursor` childed to the `commandsprites[0]` with a local position of Vector3.zero, scale of (0.5, 0.5, 0.5) using the `cursorsprite[0]` sprite with a sortorder of 20 and with angles of (180.0, 0.0, 90.0)
- internaldata is set to a new float array with length of `data[0]`
- For each internaldata elements:
    - The element is initialised to a random number between the previous one + 1.0 and 8.25 / `data[0]` - the index (if it's the first element, it's random between `data[1]` and 2.0625)
    - `commandsprites[3 + X]` (where `X` is the index) is initialised to be the SpriteRenderer of a new UI object named `Button` childed to the `commandsprites[0]` with a local position of (`internaldata[Y]`, 0.0, 0.1) where Y is the index, scale of (0.65, 0.65, 0.65) using the `guisprites[42]` sprite with a sortorder of 2 + index and with a color of 7F7F7F (gray fully opaque)


## [DoCommand](../DoCommand.md) Execution phase

