# PressKeyTimer


## `data` array

- `data[0]`: The `presskey` value after being truncated to int ???

## [DoCommand](../DoCommand.md) Setup phase

- `presskey` is set to `data[0]`
- `commandsprites` is set to a new SpriteRenderer array with 2 elements:
    - 0: The SpriteRenderer of a new UI object named `hexagon` childed to the `GUICamera` with a local position of (-5.5, 1.5, 10.0), scale of (1.5, 1.5, 1.5) using the `guisprites[42]` sprite with a sortorder of 0 and a color of pure yellow
    - 1: The SpriteRenderer of a new UI object named `hexagon outline` childed to the `commandsprites[0]` with a local position of Vector3.zero, scale of (2.0, 2.0, 2.0) using the `guisprites[80]` sprite with a sortorder of 1
- `buttons` is set to a new array with one element being the ButtonSprite of a new GameObject named `Button` which has SetUp called on it with the following:
    - buttonid: `presskey`
    - onlytype: -1
    - description: empty
    - position: (0.0, 0.0, 0.0)
    - iconsise: (0.75, 0.75, 0.75)
    - sortorder: 5
    - parentobj: `commandsprites[0]`

## [DoCommand](../DoCommand.md) Execution phase

