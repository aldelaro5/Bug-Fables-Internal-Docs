# Chapterinto

Play a chapter into and yield until it is done playing.

## Syntax

````
|chapterinto,ch|
````

## Parameters

### `ch`: int

The chapter intro to play from 1 to 7. This must be a valid chapter id or an exception will be thrown.

## Remarks

This command also saved both the message lock and minipause value before starting the coroutine only to restore them to their original value after the yield is done.

This command is only used for testing purposes in the TestRoom.
