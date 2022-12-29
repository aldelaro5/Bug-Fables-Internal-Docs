# Overall [BoardQuests](../../Enums%20and%20IDs/BoardQuests.md) list type

Display a sanitized version of the taken and completed quests. This is not to be confused with [Quest Board List Type](Quest%20Board%20List%20Type.md) which renders all the [BoardQuests](../../Enums%20and%20IDs/BoardQuests.md) of a given state.

## Options generation

`listvar` is all the [BoardQuests](../../Enums%20and%20IDs/BoardQuests.md) from the taken board followed by the quests from the completed board * -1 with the exception of quest 0 (No Quest).

`listredirect` is overridden to null.

## Option's SetText input string

The text is the name of the name of the quest corresponding to the absolute value of the option. If the language is set to `German`, |[Sizemulti](../../SetText/Commands/Individual%20commands/Sizemulti.md),0.7,1| is prepended to the string.

If the quest is a story quest (id between 11 and 17), the icon of the corresponding quest is rendered using psprite and the index as the option on the right side of the bar. The y position is 0.2 when completed and 0.25 when taken and if it is taken, the sprite is rendered with a black color.

Otherwise, if the quest isn't a story quest, but it is completed, a green checkmark is rendered instead of the icon.

In all cases where a sprite is rendered, `\t` is prepended to the string and |[Tab](../../SetText/Commands/Individual%20commands/Tab.md),tabsize| is appended where tabsize is 6.0 except in `Japanese` where it is 3.0 (also multiplied by 0.85 if the sprite is the checkmark).

If the language is set to `Japanese` and we are paused, |[size](../../SetText/Commands/Individual%20commands/size.md),0.75,y,[lock](../../SetText/Commands/Individual%20commands/Lock.md)\| is prepended to the string.

Finally, |[single](../../SetText/Commands/Individual%20commands/Single.md)\| is prepended to the string which will serve as the final string.

The x position of the text is overridden to -0.5 and the size to Vector2.one.

## Description box rendering

It uses the default rendering scheme described in [Description box rendering](../ShowItemList%20Life%20Cycle/Description%20box%20rendering.md) where the text is empty. It should be noted that under normal gameplay, this list type is not called with `showdescription` set to true.

## Input handling

The Input handling of the list is done by PauseMenu's Update instead of MainManager's.
