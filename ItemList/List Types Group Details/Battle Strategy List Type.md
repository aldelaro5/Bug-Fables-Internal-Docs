# Battle Strategy list type

Display the battle strategy options during battle.

## Options generation

`listvar` will be the following: { 0, 271, 1, 2, 3 }. These are used as offset to gather the different MenuText line id when rendering the options.

`listredirect` is overridden to null.

## Option's [SetText](../../SetText/SetText.md) input string

The main text corresponds to a certain line from MenuText depending on the options (it is 65 + the option with the exception of anything larger than 3 which is the line id itself):

* Id 66 (Rotate Party)
* Id 271 (Swap Positions)
* Id 67 (Spy)
* Id 65 (Do Nothing)
* Id 68 (Flee)

If the option is Spy while it is disabled or Flee while it isn't possible, the text is prepended with |[color](../../SetText/Commands/Individual%20commands/Color.md),1| and that will be the final string.

Otherwise, if the option is Rotate Party, ` ` + |[size](../../SetText/Commands/Individual%20commands/size.md),0.55,0.6||[button](../../SetText/Commands/Individual%20commands/Button.md),6| is appended to the MenuText line and this is the final string.

Otherwise, if the option is Do Nothing, |[single](../../SetText/Commands/Individual%20commands/Single.md)\| is prepended to the main text and ` ` + |[size](../../SetText/Commands/Individual%20commands/size.md),0.55,0.6| is appended to it. There are also [Icon](../../SetText/Commands/Individual%20commands/Icon.md) commands that can be appended if a certain medal is equipped which will be appended in this order if that is the case:

* Prayer, |[Icon](../../SetText/Commands/Individual%20commands/Icon.md),187|
* Meditation, |[Icon](../../SetText/Commands/Individual%20commands/Icon.md),188|
* Reflection, |[Icon](../../SetText/Commands/Individual%20commands/Icon.md),189|

Otherwise, if the option is Flee, |[single](../../SetText/Commands/Individual%20commands/Single.md)\| is prepended to the main text and ` ` + |[size](../../SetText/Commands/Individual%20commands/size.md),0.55,0.6| is appended to it. There are also [Icon](../../SetText/Commands/Individual%20commands/Icon.md) commands that can be appended if a certain medal is equipped which will be appended in this order if that is the case:

* Secure Pouch, |[Icon](../../SetText/Commands/Individual%20commands/Icon.md),221|
* Quick Flea, |[Icon](../../SetText/Commands/Individual%20commands/Icon.md),222|

Otherwise, if the option is Spy and Spy Specs is equipped, |[single](../../SetText/Commands/Individual%20commands/Single.md)\| is prepended to the main text and ` ` + |[size](../../SetText/Commands/Individual%20commands/size.md),0.55,0.6||[Icon](../../SetText/Commands/Individual%20commands/Icon.md),219| is appended to it. 

The x position of the main text is overridden to -2.6.

## Description box rendering

It uses the default rendering scheme described in [Description box rendering](../ShowItemList%20Life%20Cycle/Description%20box%20rendering.md) where the text is a certain line from MenuText depending on the options (it is 69 + the option with the exception of anything larger than 3 which is the line id itself):

* Id 70 (Rotate Party description)
* Id 272 (Swap Positions description)
* Id 71 (Spy description)
* Id 69 (Do Nothing description)
* Id 72 (Flee description)

## Confirmation handling

The confirmation handling only applies when in battle in MainManager's Update which is the only circumstances this list type can be called under normal gameplay. If we weren't in battle, the [Items List Type](Items%20List%20Type.md)'s confirmation handling logic will be used instead which will cause an exception to be thrown.

In that handler, it checks if the option is available to be used and if it is, it calls SetItem on the current battle with the option selected. Otherwise, the buzzer sound plays. The specific logic on what to do with a valid option is entirely within BattleControl's SetItem.
