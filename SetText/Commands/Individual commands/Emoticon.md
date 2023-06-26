# Emoticon

Show an emoticon above an [Entity](../../../Entities/Entity.md) for a given amount of frames.

## Syntax

````
|emoticon,entity,type,time|
````

## Parameters

### `entity`: int | string

The [Entity](../../../Entities/Entity.md) that this command refers to. The int form is a regular [Entity id](../Entity%20id.md) and it must not resolve to null or an exception will be thrown. The string form are only applicable if the caller isn't null and it can be one of the following when not in battle (if in battle, this is interpreted as an int which will cause an exception to be thrown):

* `this`: Refers to the [tailtarget](../../Notable%20local%20variable/tailtarget.md). This must not be null or an exception will be thrown.
* `caller`: Refers to the caller. This must not be null or an exception will be thrown.
* Anything else: if the string is in the [Define](Define.md) list, it will refer to it. Otherwise, the value will be interpreted as the int form.

### `type`: int

The type of emoticon to use. This must be a valid emoticon type or an exception will be thrown. Here are the valid emoticon types:

|Value|Description|
|-----|-----------|
|-1|None|
|0|A talking face|
|1|A ? icon|
|2|A red ! icon|
|3|An animated ellipsis icon|
|4|The Detector medal icon|
|5|A green ! icon|

### `time`: int

The amount of frames to show the emoticon for. This must be a valid int or an exception will be thrown.
