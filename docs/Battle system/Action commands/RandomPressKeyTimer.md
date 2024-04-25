# RandomPressKeyTimer


## `data` array

- `data[0]`: Decides the value of `presskey` depending on its value (it is left unchanged if it isn't in the list):
    - 0.0: Random between 4 and 6 inclusive
    - 1.0: Random between 0 and 3 inclusive
    - 2.0: Random between 0 and 6 inclusive
- `data[1]`: The length of `commandsprites` will be 2 + this value ???

## [DoCommand](../DoCommand.md) Setup phase

- `presskey` is set to a value that depends on `data[0]` (check the `data` array section above for details)
- `commandsprites` is set to a new array with 2 + `data[1]` elements where the first one is the SpriteRenderer of a new UI object named `barholder` childed to the `GUICamera` with a local position of (-7.0, 1.0, 10.0), scale of Vector3.one using the `guisprites[62]` sprite with a sortorder of 0
- CreatePips(2) is called which initialises each elements of `commandsprites` after the second one (if any exists) to be the SpriteRenderer of a new UI object named `pointX` where `X` is the index of the element childed to `commandsprites[0]` with a local position of (x, 0.0, 0.0) where x is a lerp from 0.0 to 4.5 with the factor being the ratio of element's index - 2 over `commandsprites`.length - 3 (TODO: division by 0 if `data[1]` is exactly 1 ???), scale of (0.5, 0.5, 1.0) (or Vector3.one if it's the last element) using the `guisprites[59]` sprite (`guisprites[42]` if it's the last element) with a sortorder of 1 + the element's index and a color of 7F7F7F (fully opaque gray)
- letters is set to a new array of one element childed to the last `commandsprites` which has the following initialisation:
    - SetFont is called on it with [fontid](../../SetText/Notable%20states.md#font-id-table) 0 (`BubblegumSans`)
    - text of `?`
    - local position of Vector3.zero
    - scale of (0.1, 0.1, 1.0)
    - layer of 5 (`UI`)
    - anchor of `MiddleCenter`
    - MeshRenderer has a sortingOrder of 30
- `buttons` is set to a new array with one element being the ButtonSprite of a new GameObject named `Button` (disabled for now) which has SetUp called on it with the following:
    - buttonid: `presskey`
    - onlytype: -1
    - description: empty
    - position: (0.0, 0.0, 0.0)
    - iconsise: (0.75, 0.75, 0.75)
    - sortorder: 50
    - parentobj: The last `commandsprites`

## [DoCommand](../DoCommand.md) Execution phase

