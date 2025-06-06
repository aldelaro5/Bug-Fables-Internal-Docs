# Skills list type

Display the list of [skills](../../Enums%20and%20IDs/Skills.md) that a party member has access to.

## Options generation

`listvar` will be the skills array field of the corresponding playerdata (-1, -2 and -3 means 0, 1 and 2 respectively). They corresponds the skills id of skilldata.

The playerdata's skills are expected to be refreshed by using [RefreshSkills](../../Battle%20system/RefreshSkills.md) which takes into account the party's rank and [Medal](../../Enums%20and%20IDs/Medal.md)s equipped.

`listredirect` is overridden to null.

## Option's [SetText](../../SetText/SetText.md) input string

This behavior differs depending if we were paused or unpaused.

### When paused

A first SetText call is done for the tp cost in non [Dialogue mode](../../SetText/Dialogue%20mode.md) where the input string is |[sort](../../SetText/Individual%20commands/Sort.md),10||[Font](../../SetText/Individual%20commands/Font.md),0| followed by the cost padded left to 2 characters by ` ` or `  -` if there is no cost. This is rendered towards the right of the item bar. 

After, if the TP cost isn't 0, a TP icon (or HP icon if Life Cast is equipped) is rendered on the far right of the item bar. 

Finally, the input string is set to |[single](../../SetText/Individual%20commands/Single.md)\| followed by the skill's name from `skilldata`. 

The position of the input string is overridden to (-0.75, -0.2) and the bar height used for calculating the y position of the down arrow is overridden to -0.9.

### When unpaused

A first [SetText](../../SetText/SetText.md) call is done for the tp cost (obtained by calling GetTPCost detailed below) in non [Dialogue mode](../../SetText/Dialogue%20mode.md) where the input string is |[sort](../../SetText/Individual%20commands/Sort.md),10||[size](../../SetText/Individual%20commands/size.md),0.75||[Font](../../SetText/Individual%20commands/Font.md),0| followed by |[color](../../SetText/Individual%20commands/Color.md),1| if the player doesn't have enough TP for the skill (Checked by HasSkillCost detailed below) followed by the cost padded left to 2 characters by ` `.  This is rendered towards the right of the item bar.

After, if the TP cost isn't 0, a TP icon (or HP icon if Life Cast is equipped) is rendered on the far right of the item bar. 

Then, a string is built with [Icon](../../SetText/Individual%20commands/Icon.md) commands according to some medals being equipped (this is appended in the respective order if the medal is equipped):

* Needle Toss and Needle pincer: ` ` + |[size](../../SetText/Individual%20commands/size.md),0.55,0.6| + |[Icon](../../SetText/Individual%20commands/Icon.md),192| if Electric Needles, + |[Icon](../../SetText/Individual%20commands/Icon.md),191| if Sleepy Needles and + |[Icon](../../SetText/Individual%20commands/Icon.md),193| if Poison Needles + ` ` + |[size](../../SetText/Individual%20commands/size.md),0.55,0.6| + |[Icon](../../SetText/Individual%20commands/Icon.md),194| if A.D.B.P Enhancer
* Tornado Toss and Hurricane Toss: ` ` + |[size](../../SetText/Individual%20commands/size.md),0.55,0.6| + |[Icon](../../SetText/Individual%20commands/Icon.md),194| if A.D.B.P Enhancer
* Secret Stash and Sharing Stash: ` ` + |[size](../../SetText/Individual%20commands/size.md),0.55,0.6| + |[Icon](../../SetText/Individual%20commands/Icon.md),218| if Heal Plus
* Taunt: ` ` + |[size](../../SetText/Individual%20commands/size.md),0.55,0.6| + |[Icon](../../SetText/Individual%20commands/Icon.md),195| if Deep Taunt

Finally, the input string is set to the skill's name from skilldata prepended with |[color](../../SetText/Individual%20commands/Color.md),1| if the player doesn't have enough TP for the skill and appended with the icons string. 

The x position of the text is overridden to -2.65.

### GetTPCost
GetTPCost is a method only used for this listtype that will obtain the effective cost of the skill in TP after considering several medals:

```cs
public static int GetTPCost(int player, int id)
public static int GetTPCost(int player, int id, bool matchid)
```
It returns the cost in HP (if it's negative) or TP (if it's not negative) of the skill of `playerdata[player].skills[id]` after processing medals that can change skills cost. The `matchid` parameter should always be false because sending a value of true is considered broken and the game never does that under normal gameplay (it will result in `id` being converted to the matching skill id in the skills list and then incorrectly used to address the skills list itself resulting in an exception being thrown or the wrong skill being addressed).

The overload without the `matchid` parameter has its value default to false.

Here's the exact calculation procedure:

- The [skill](../../Enums%20and%20IDs/Skills.md) id is obtained from `playerdata[player].skills[id]`
- Obtains the cost from the [skillsdata](../../TextAsset%20Data/Skills%20data.md) and get its absolute value (it's the cost in HP or TP as a positive number)
- If the `Beemerang2` [medal](../../Enums%20and%20IDs/Medal.md) is equipped and the skill id is one of the following, the cost increases by 1:
    - `NeedleToss`
    - `BeeRangMultiHit`
    - `HurricaneBeemerang`
    - `NeedlePincer`
- If the `HPFunnel` [medal](../../Enums%20and%20IDs/Medal.md) is equipped and the skill id is one of the following, the cost increases by the amount of `HPFunnel` equipped on `playerdata[player].trueid` (this is done to cancel out the cost decrease later in the method for these 2 skills):
    - `SecretStash`
    - `SharingStash`
- At this point, if the cost is 0, it is immediately returned so the skill costs no TP
- The amount of `TPSaver` [medal](../../Enums%20and%20IDs/Medal.md) equipped on `playerdata[player].trueid` is removed from the cost
- The amount of `HPFunnel` [medal](../../Enums%20and%20IDs/Medal.md) equipped on `playerdata[player].trueid` is removed from the cost
- The cost is clamped from 1 to 99
- If the number obtained from the `skilldata` earlier indicated an HP cost, the cost is negated so it returns in the same format (negative is HP cost, anything else is a TP cost)
- The final cost is returned

### HasSkillCost
HasSkillCost is a method only used for this listtype that will check if the player party member can pay the resources the skill demands:

```cs
private static bool HasSkillCost(int tpcost, int playerid)
```
It returns true if `tp` is at least `tpcost`, false otherwise.

However, if `playerdata[playerid]` has the `HPFunnel` medal equipped or if `tpcost` is negative, this instead returns true when `playerdata[playerid]`.`hp` - 1 is at least the absolute value of `tpcost`, false otherwise.

This method is intended to check if the player is able to pay the cost of a skill whether it is in TP (`tpcost` isn't negative) or HP (`tpcost` is negative or `playerdata[playerid]` has the `HPFunnel` medal equipped). In the case the cost is in HP, the HP cost is the absolute value of `tpcost` - 1.

## Description box rendering

It uses the default rendering scheme described in [Description box rendering](../ShowItemList%20Life%20Cycle/Description%20box%20rendering.md) where the text is the skill's description obtained from skilldata using the skill id.

## Confirmation handling (during battle)

The skill cost will then be checked if the player has enough and if they can use the skill in general (checked using HasSkillCost which is detailed above). The cost is stored in [flagvar](../../Flags%20arrays/flagvar.md) 0. If the skill can be used and the player can pay the cost, [SetItem](../../Battle%20system/Player%20UI/SetItem.md) is called on the battle with the selected skill and DestroyList is called which ends the processing of the list.

If the skill can't be used, the buzzer sound will play. In addition, if the player couldn't pay the cost and the skill wasn't Hard Charge, the appropriate hud sprite will flash red (the TP one if it's TP cost or the party member's HP if we have Life Cast).
