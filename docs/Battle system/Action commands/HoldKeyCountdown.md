# HoldKeyCountdown


## `data` array

- `data[0]`: The length of `commandsprites` will be 1 + this value ???
- `data[1]`: The `presskey` value after being truncated to int ???

## [DoCommand](../DoCommand.md) Setup phase

- `presskey` is set to `data[1]`
- `buttons` is set to a new array with one element being the ButtonSprite of a new GameObject named `Button` which has SetUp called on it with the following:
    - buttonid: `presskey`
    - onlytype: -1
    - description: empty
    - position: (-0.8, 1.0, 10.0)
    - iconsise: (0.75, 0.75, 0.75)
    - sortorder: 2
    - parentobj: `GUICamera`
- `commandsprites` is set to a new array with 1 + `data[0]` elements where the first one is the SpriteRenderer of a new UI object named `barholder` childed to the `GUICamera` with a local position of (-7.0, 1.0, 10.0), scale of Vector3.one using the `guisprites[62]` sprite with a sortorder of 0
- CreatePips(1) is called which initialises each elements of `commandsprites` after the first one (if any exists) to be the SpriteRenderer of a new UI object named `pointX` where `X` is the index of the element childed to `commandsprites[0]` with a local position of (x, 0.0, 0.0) where x is a lerp from 0.0 to 4.5 with a factor being the ratio of element's index - 1 over `commandsprites`.length - 2 (TODO: division by 0 if `data[0]` is exactly 1 ???), scale of (0.5, 0.5, 1.0) (or Vector3.one if it's the last element) using the `guisprites[59]` sprite (`guisprites[42]` if it's the last element) with a sortorder of 1 + the element's index and a color of 7F7F7F (fully opaque gray)

## [DoCommand](../DoCommand.md) Execution phase

