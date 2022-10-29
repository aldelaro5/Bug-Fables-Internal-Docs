# Skills list type

Display the list of skills that a party member has access to.

## Options generation

`listvar` will be the skills array field of the corresponding playerdata (-1, -2 and -3 means 0, 1 and 2 respectively). They corresponds the skills id of skilldata.

`listredirect` is overridden to null.

## Option's SetText input string

This behavior differs depending if we were paused or unpaused.

### When paused

A first SetText call is done for the tp cost in non [Dialogue mode](../../SetText/Dialogue%20mode.md) where the input string is `|sort,10||font,0|` followed by the cost padded left to 2 characters by ` ` or `  -` if there is no cost. This is rendered towards the right of the item bar. 

After, if the TP cost isn't 0, a TP icon (or HP icon if Life Cast is equipped) is rendered on the far right of the item bar. 

Finally, the input string is set to `|single|` followed by the skill's name from skilldata. 

The position of the input string is overridden to (-0.75, -0.2) and the bar height used for calculating the y position of the down arrow is overridden to -0.9.

### When unpaused

A first SetText call is done for the tp cost in non [Dialogue mode](../../SetText/Dialogue%20mode.md) where the input string is `|sort,10||size,0.75||font,0|` followed by `|color,1|` if the player doesn't have enough TP for the skill followed by the cost padded left to 2 characters by ` `.  This is rendered towards the right of the item bar. 

After, if the TP cost isn't 0, a TP icon (or HP icon if Life Cast is equipped) is rendered on the far right of the item bar. 

Then, a string is built with `Icon` commands according to some medals being equipped (this is appended in the respective order if the medal is equipped):

* Needle Toss and Needle pincer: ` |size,0.55,0.6|` + `|icon,192|` if Electric Needles, + `|icon,191|` if Sleepy Needles and + `|icon,193|` if Poison Needles + ` |size,0.55,0.6|` + `|icon,194|` if A.D.B.P Enhancer
* Tornado Toss and Hurricane Toss: ` |size,0.55,0.6|` + `|icon,194|` if A.D.B.P Enhancer
* Secret Stash and Sharing Stash: ` |size,0.55,0.6|` + `|icon,218|` if Heal Plus
* Taunt: ` |size,0.55,0.6|` + `|icon,195|` if Deep Taunt

Finally, the input string is set to the skill's name from skilldata prepended with `|color,1|` if the player doesn't have enough TP for the skill and appended with the icons string. 

The x position of the text is overridden to -2.65.

## Description box rendering

It uses the default rendering scheme described in [Description box rendering](../ShowItemList%20Life%20Cycle/Description%20box%20rendering.md) where the text is the skill's description obtained from skilldata using the skill id.

## Confirmation handling (during battle)

The skill cost will then be checked if the player has enough and if they can use the skill in general. The cost is stored in flagvar 0. If the skill can be used and the player can pay the cost, SetItem is called on the battle with the selected skill and DestroyList is called which ends the processing of the list.

If the skill can't be used, the buzzer sound will play. In addition, if the player couldn't pay the cost and the skill wasn't Hard Charge, the appropriate hud sprite will flash red (the TP one if it's TP cost or the party member's HP if we have Life Cast).
