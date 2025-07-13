# Audience
A component meant to be added programmatically or via Unity to a GameObject.

This is a component meant to hold many instances of the `Prefabs/Objects/Audience` prefab located closely together in a defined area with configurations to render their sprite through different AnimationClips selected more or less randomly. They are all childed to the attached GameObject while being locally positioned differently within a range.

The prefab has an Animator and a SpriteRenderer at its root. The animator controls the sprite to use through the `animationcontrollers/_misc/audience` controller which features clips named in specific way. Their naming format is `X_Y` where `X` is a number from 0 to 5 that represents the audience member kind to render and `Y` is the specific sprite number to use (0 is idle, 1 is cheering sprite). All audience member's sprites are gray because they are meant to be colorised at runtime by the component using colors that are randomly selected within a range using a lerp from one color to another with the factor being random. Here are the audience member kind mappings and their color range:

- 0: Moth - from pure yellow to pure red with the factor between 0.2 and 0.8
- 1: Ant - from pure yellow to pure red with the factor between 0.5 and 0.9
- 2: Beetle - from pure green to pure blue with the factor between 0.2 and 0.8
- 3: Bee - from pure yellow to pure red with the factor between 0.2 and 0.45
- 4: Termite - from pure white to pure yellow with the factor between 0.2 and 0.5
- 5: Termite (older looking) - from pure red to pure yellow with the factor between 0.3 and 0.45

Since most of the logic are complex animations, only the global details will be given. Here are all the public fields:

- `animtype`: An enum value that determines how to select the audience member kind:
    - `MothAntBeetle`: Random between 0 and 2
    - `OnlyAnt`: 1
    - `OnlyBee`: 3
    - `OnlyBeetle`: 2
    - `OnlyMoth`: 0
    - `All`: Random between 0 and 3
    - `Termites`: Random between 4 and 5
- `ammount`: The amount of audience members to spawn
- `currentammount`: This field holds `ammount` after Start, but it isn't used for anything
- `spawnarea`: The xz local position range to use for each audio member. More precisely, the range is random between (-(`spawnarea`.x / 2.0), 0.0, -(`spawnarea`.y / 2.0)) and (`spawnarea`.x / 2.0, 0.0, `spawnarea`.y / 2.0)
- `constantjump`: The 2 components of this Vector2 controls how to change the y local position of the audience members on LateUpdate periodically. If this is Vector2.zero, this feature is disabled
- `noflip`: This value does nothing
- `lowfps`: This value does nothing
- `entities` (not configurable via the inspector): The Animators of all the audience members. It is exposed to the game.

Most of the setup is done on Start, but there is an OnEnable that will force all audience members on their idle animation. It's also possible to do the same externally by calling the RefreshAnim public method. The `constantjump` feature is implemented in LateUpdate as well as the ability to periodically change their y angles.

Finally, there's a Jump public method that starts a coroutine that has all audience members temporarilly go to their cheering animation while either jumping, rotating in y or both.
