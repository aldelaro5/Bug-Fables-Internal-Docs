# Caravan prize medals list type

Display the list of available prize medals to purchase from the Caravan shop.

## Options generation

`listvar` will be set to caravanorder which contains all the medals ids that the caravan has available from the prize medals pool.

`showdescription` is overridden to false if flags 681 is true (MYSTERY? is active) or to true if that flags is false.

## Option's [SetText](../../SetText/SetText.md) input string

If the option is -1 or lower, the text won't be changed and nothing will happen in this phase.

Otherwise, the text is the medal's name from badgedata or MenuText line 59 (?????) if [flags](../../Flags%20arrays/flags.md) 681 is true (MYSTERY? is active). Additionally a sprite of the medal (or a ? in a grey hexagon if MYSTERY? is active) will be rendered on the leftmost portion of the bar with a localScale of (0.55, 0.6, 1.0). Finally, the y position of the text is overridden to -0.2.

## Description box rendering

It uses the default rendering scheme described in [Description box rendering](../ShowItemList%20Life%20Cycle/Description%20box%20rendering.md) where the text is the medal's description obtained from badgedata using the id of the medal the player has located at the index of the option. If this is a `sell` list, the secondary string is the medal's name obtained in a similar manner.
