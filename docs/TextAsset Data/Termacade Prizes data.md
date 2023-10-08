# Termacade Prizes data

The game contains a TextAsset located at `Ressources/data/Termacade` that contains data about the purchasable prizes at Termacade. It is loaded on boot in LoadEssentials.

The asset contains a list of prizes in the order they appear in the [Termacade Prizes List Type](../ItemList/List%20Types%20Group%20Details/Termacade%20Prizes%20List%20Type.md), one for each line. Each lines contains field separated by `,`:

|Loaded index|Name|Type|Description|
|------------:|----|----|-----------|
|0|Type|int|The type of the item (0 = standard items, 1 = key items, 2 = medal)|
|1|Item id|int|The [Item](../Enums%20and%20IDs/Items.md) or [Medal](../Enums%20and%20IDs/Medal.md) id of the prize|
|2|Game Tokens cost|int|The cost in Game Tokens to purchase the prize|
|3|Availability|int|How available the prize is (1 = only once as long as the flags is false, always available otherwise)|
|4|Flags|int|The [flag](../Flags%20arrays/flags.md) slot that determines if the prize was purchased if its availability is 1 (doesn't do anything if it isn't)|

The data will be loaded into `termacadeprize[i, x]` where `i` is the line index and `x` the loaded index.

This information is both used in the [Termacade Prizes List Type](../ItemList/List%20Types%20Group%20Details/Termacade%20Prizes%20List%20Type.md) to render each item, but also in [Event](../SetText/Individual%20commands/Event.md) 121 which is expected to be called right after the ItemList of the aforementioned list type is handled. The event essentially acts like a custom ItemList handler which uses the data to check and perform the transaction.
