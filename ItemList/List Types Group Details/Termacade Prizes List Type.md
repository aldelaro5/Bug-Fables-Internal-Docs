# Termacade prizes list type

Display the list of the Termacade prizes.

## Options generation

`listvar` is the integers from 0 to the length of termacadeprize -1.

## Option's SetText input string

The text is the name of the item from itemdata or medal from badgedata unless it is a medal and [flags](../../Flags%20arrays/flags.md) 681 (MYSTERY? activation) is true where the text is MenuText 59 which is "?????".

The sprite of the item or medal will be rendered on the leftmost side of the bar. This sprite is a gray hexagon with a ? in it if it is a medal and [flags](../../Flags%20arrays/flags.md) 681 (MYSTERY? activation) is true. The localScale of the sprite is (0.55, 0.6, 1).

If there is only one instance of the item available and its corresponding purchased flags is true, [SetText](../../SetText/SetText.md) is called where the parent is the bar, the position is (1.0, -0.15) and the input string is |[sort](../../SetText/Commands/Individual%20commands/Sort.md),10||`size`,0.6,0.75||[color](../../SetText/Commands/Individual%20commands/Color.md),1| followed by MenuText 190 which is "Sold Out!".

Otherwise, [SetText](../../SetText/SetText.md) is called where the parent is the bar, the position is (1.4, -0.15) and the input string is |[sort](../../SetText/Commands/Individual%20commands/Sort.md),10||`size`,0.6,0.75| followed by the price of the items in tokens from termacadeprize. Additionally, a token sprite is rendering on the rightmost part of the bar. 

## Description box rendering

It uses the default rendering scheme described in [Description box rendering](../ShowItemList%20Life%20Cycle/Description%20box%20rendering.md) where the text is the item's description from itemdata or the medal description from badgedata if [flags](../../Flags%20arrays/flags.md) 681 (MYSTERY? activation) is false (if it is, it's MenuText 59 which is "?????").

## Confirmation handling

Confirmation is handled by MainManager's Update. First, the flagvar of the `storeid` is set to the selected option (bug?). Then, the list gets destroyed which ends this list's processing and resets the [ItemList State Machine](../ItemList%20State%20Machine.md).
