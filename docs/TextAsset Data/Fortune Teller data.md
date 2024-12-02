# Fortune teller data

[Musics](../Enums%20and%20IDs/Musics.md) data are split in 3 TextAssets in the game loaded in [Event](../Enums%20and%20IDs/Events.md) 71 (talking to the Fortune Teller) and located in the corresponding dialogue directory of the current [languageid](../SetText/languageid.md):

* `FortuneTeller0`
* `FortuneTeller1`
* `FortuneTeller2`

## `FortuneTeller0` data

The asset contains one line per [crystalbfflags](../Enums%20and%20IDs/crystalbfflags.md) slot which contains 1 field per line:

|Name|Type|Description|
|----|----|-----------|
|Hint|[SetText](../SetText/SetText.md) string|The hint given for the corresponding [crystalbfflags](../Enums%20and%20IDs/crystalbfflags.md)|

The hint is selected randomly among every [crystalbfflags](../Enums%20and%20IDs/crystalbfflags.md) slots whose value is false. None is given if all of them are true.

## `FortuneTeller1` data

The asset contains one line per Lore Book which contains 1 field per line:

|Name|Type|Description|
|----|----|-----------|
|Hint|[SetText](../SetText/SetText.md) string|The hint given for the corresponding Lore Book|

Each Lore Book is expected to be bound to a specific [flag](../Flags%20arrays/flags.md) slot which are hardcoded. Here are the [flags](../Flags%20arrays/flags.md) slots expected for each line index:

|Line|Flags slot|
|----:|----------|
|0|71|
|1|87|
|2|142|
|3|271|
|4|331|
|5|380|
|6|381|
|7|388|
|8|392|
|9|455|
|10|460|
|11|463|
|12|200|
|13|243|
|14|55|
|15|469|
|16|471|
|17|486|
|18|468|
|19|488|
|20|489|
|21|498|
|22|499|
|23|578|
|24|604|
|25|603|

The hint can only be selected if the [flag](../Flags%20arrays/flags.md) slot is false. No hints will be given if they are all true.

## `FortuneTeller2` data

The asset contains one line per [Medal](../Enums%20and%20IDs/Medal.md) that has a hint which contains 1 field per line:

|Name|Type|Description|
|----|----|-----------|
|Hint|[SetText](../SetText/SetText.md) string|The hint given for the corresponding [Medal](../Enums%20and%20IDs/Medal.md)|

Each of these hints is expected to be a [Medal](../Enums%20and%20IDs/Medal.md) bound to a specific [flag](../Flags%20arrays/flags.md) slot which are hardcoded. Here are the [flags](../Flags%20arrays/flags.md) slots expected for each line index:

|Line|Flags slot|
|----:|----------|
|0|338|
|1|121|
|2|230|
|3|137|
|4|611|
|5|462|
|6|220|
|7|149|
|8|415|
|9|285|
|10|355|
|11|534|
|12|575|
|13|42|
|14|23|
|15|60|
|16|721|
|17|719|

The hint can only be selected if the [flag](../Flags%20arrays/flags.md) slot is false. No hints will be given if they are all true.
