# Musics data

[Musics](../Enums%20and%20IDs/Musics.md) data are split in 2 TextAssets in the game:

* Loaded by LoopPoint which will only run on the first FixedUpdate after LoadEverything:
  * `Ressources/data/LoopPoints`
* Loaded on boot by SetVariables
  * `MusicList` from the corresponding dialogue directory of the current [languageid](../SetText/languageid.md)

## `LoopPoints` data

The asset contains one line per [Musics](../Enums%20and%20IDs/Musics.md) which contains the loop points of each music where the line index corresponds to the music. Each line contains 2 fields separated by `;`:

|Loaded index|Name|Type|Description|
|------------|----|----|-----------|
|0|End boundary|float|The timestamp in seconds to stop playing the audio and resume at the restart boundary|
|1|Restart boundary|float|The timestamp in seconds to continue playing the audio once it reaches the end boundary|

The data will be loaded into `musicloop[id, x]` where `id` is the [Musics](../Enums%20and%20IDs/Musics.md) id and `x` is the loaded index.

## `MusicList` data

The asset contains one line per [Musics](../Enums%20and%20IDs/Musics.md) whose id corresponds to the line index. Each line contains 1 field:

|Name|Type|Description|
|----|----|-----------|
|Name|[SetText](../SetText/SetText.md) string|The name of the [Musics](../Enums%20and%20IDs/Musics.md) as shown by [Samira Songs List Type](../ItemList/List%20Types%20Group%20Details/Samira%20Songs%20List%20Type.md)|

The data will be loaded into `musicnames[i]` where `i` is the line index. The convention is to name any songs not intended to be shown as `.`.

The name of the entry is used by the [Samira Songs List Type](../ItemList/List%20Types%20Group%20Details/Samira%20Songs%20List%20Type.md) for rendering.
