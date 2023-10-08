# Skills list type

Display the list of skills that a party member has access to.

## Options generation

`listvar` will be the skills array field of the corresponding playerdata (-1, -2 and -3 means 0, 1 and 2 respectively). They corresponds the skills id of skilldata.

The playerdata's skills are expected to be refreshed by using RefreshSkills which takes into account the party's rank and [Medal](../../Enums%20and%20IDs/Medal.md)s equipped.

`listredirect` is overridden to null.

## Option's [SetText](../../SetText/SetText.md) input string

This behavior differs depending if we were paused or unpaused.

### When paused

A first SetText call is done for the tp cost in non [Dialogue mode](../../SetText/Dialogue%20mode.md) where the input string is |[sort](../../SetText/Individual%20commands/Sort.md),10||[Font](../../SetText/Individual%20commands/Font.md),0| followed by the cost padded left to 2 characters by ` ` or `  -` if there is no cost. This is rendered towards the right of the item bar. 

After, if the TP cost isn't 0, a TP icon (or HP icon if Life Cast is equipped) is rendered on the far right of the item bar. 

Finally, the input string is set to |[single](../../SetText/Individual%20commands/Single.md)\| followed by the skill's name from `skilldata`. 

The position of the input string is overridden to (-0.75, -0.2) and the bar height used for calculating the y position of the down arrow is overridden to -0.9.

### When unpaused

A first [SetText](../../SetText/SetText.md) call is done for the tp cost in non [Dialogue mode](../../SetText/Dialogue%20mode.md) where the input string is |[sort](../../SetText/Individual%20commands/Sort.md),10||[size](../../SetText/Individual%20commands/size.md),0.75||[Font](../../SetText/Individual%20commands/Font.md),0| followed by |[color](../../SetText/Individual%20commands/Color.md),1| if the player doesn't have enough TP for the skill followed by the cost padded left to 2 characters by ` `.  This is rendered towards the right of the item bar. 

After, if the TP cost isn't 0, a TP icon (or HP icon if Life Cast is equipped) is rendered on the far right of the item bar. 

Then, a string is built with [Icon](../../SetText/Individual%20commands/Icon.md) commands according to some medals being equipped (this is appended in the respective order if the medal is equipped):

* Needle Toss and Needle pincer: ` ` + |[size](../../SetText/Individual%20commands/size.md),0.55,0.6| + |[Icon](../../SetText/Individual%20commands/Icon.md),192| if Electric Needles, + |[Icon](../../SetText/Individual%20commands/Icon.md),191| if Sleepy Needles and + |[Icon](../../SetText/Individual%20commands/Icon.md),193| if Poison Needles + ` ` + |[size](../../SetText/Individual%20commands/size.md),0.55,0.6| + |[Icon](../../SetText/Individual%20commands/Icon.md),194| if A.D.B.P Enhancer
* Tornado Toss and Hurricane Toss: ` ` + |[size](../../SetText/Individual%20commands/size.md),0.55,0.6| + |[Icon](../../SetText/Individual%20commands/Icon.md),194| if A.D.B.P Enhancer
* Secret Stash and Sharing Stash: ` ` + |[size](../../SetText/Individual%20commands/size.md),0.55,0.6| + |[Icon](../../SetText/Individual%20commands/Icon.md),218| if Heal Plus
* Taunt: ` ` + |[size](../../SetText/Individual%20commands/size.md),0.55,0.6| + |[Icon](../../SetText/Individual%20commands/Icon.md),195| if Deep Taunt

Finally, the input string is set to the skill's name from skilldata prepended with |[color](../../SetText/Individual%20commands/Color.md),1| if the player doesn't have enough TP for the skill and appended with the icons string. 

The x position of the text is overridden to -2.65.

## Description box rendering

It uses the default rendering scheme described in [Description box rendering](../ShowItemList%20Life%20Cycle/Description%20box%20rendering.md) where the text is the skill's description obtained from skilldata using the skill id.

## Confirmation handling (during battle)

The skill cost will then be checked if the player has enough and if they can use the skill in general. The cost is stored in [flagvar](../../Flags%20arrays/flagvar.md) 0. If the skill can be used and the player can pay the cost, SetItem is called on the battle with the selected skill and DestroyList is called which ends the processing of the list.

If the skill can't be used, the buzzer sound will play. In addition, if the player couldn't pay the cost and the skill wasn't Hard Charge, the appropriate hud sprite will flash red (the TP one if it's TP cost or the party member's HP if we have Life Cast).
