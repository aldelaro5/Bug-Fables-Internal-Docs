# Samira songs list type

Display the list of the currently available songs from Samira.

## Options generation

`listvar` is the list of music id in samiramusics which contains only the songs available for purchase or have been bought already. If there is at least one song that is available, but not purchased yet, a -1 option is added first before adding all the songs from samiramusics.

## Option's SetText input string

If the option is -1, the text is `|color,1|` followed by CommonDialogue line 152 which is "We'll buy all songs!" in English.

Otherwise, the entry in samiramusics is obtained by using the index of the option (1 is subtracted from it if -1 is in `listvar`). Then, the text becomes the name of the song in musicnames using the music id. This is prepended with `|color,3|` if the song was bought.

The x position of the text is overridden to -2.65.

## Description box rendering

It uses the default rendering scheme described in [Description box rendering](../ShowItemList%20Life%20Cycle/Description%20box%20rendering.md) where the text is empty. It should be noted that under normal gameplay, this list type is not called with `showdescription` set to true.

## Confirmation handling

The list gets destroyed when a confirmation occurs by MainManager's Update which ends this list's processing and resets the [ItemList State Machine](../ItemList%20State%20Machine.md).
