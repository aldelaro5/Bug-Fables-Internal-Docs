# Flagstring

A `flagstring` is a string which is a global variable the game can access for any purpose. They are saved on the [Save File](../Save%20File.md) and typically have one reserved use, but some can be used temporarily notably in [SetText](../SetText/SetText.md) operations. There are 15 flagstrings available to the game in the `MainManager.instance.flagstring` array.

## Notes about the flagstrings table

A flagstring with the mention "TEMP" means its value in the save is meaningless since it is only used for temporary usage by the game.

## Flagstrings table

|ID|Description|
|--|-----------|
|0|TEMP|
|1|TEMP|
|2|TEMP Only used for letter prompts|
|3|TEMP Only used for the prompt after each battle using the B.O.S.S system|
|4|TEMP Only used when getting a medal from H.B using the B.O.S.S system|
|5|UNUSED|
|6|UNUSED (had a use in 1.0.x inside dead code about the Ant Compass last dug spot)|
|7|TEMP Holds a comma separated list of the card ID (refer to Appendix J) corresponding to the current deck being built|
|8|Holds some informmation about the items, key items and berries in possession before the bandit ambush in Chapter 4. The format is the following: comma separated list of the items ID (refer to Appendix C) followed by a "-", followed by the same list for key items, followed by a "-" followed by the berry count. This is only used when recalling the list once the locked chest at the Bandits Hideout is opened|
|9|Chompy's name|
|10|Filename|
|11|TEMP Holds a comma separated list of the taken quest ID when the command Activateselectedquest is passed to SetText|
|12|Holds a comma separated list of the card ID (refer to Appendix J) corresponding to the saved deck for Spy Cards|
|13|Holds a comma separated list of the ordered medals ID (refer to Appendix A) in MYSTERY?, empty if the code is not active|
|14|UNUSED|
