# [listtype](listtype.md)

There are 34 types of ItemLists available. Each type mainly dictates how `listvar` is generated, how its SetText input string for each options is generated (but also additional rendering on their bar sprite is done) and how its `listdescbox` is rendered. Some can have specific behaviors external to [ShowItemList](ShowItemList.md).

The handling of the ItemList is usually done by MainManager's Update unless in the pause menu (which includes the settings window called from StartMenu and the change loadout option on game over). In these cases, the handling is done by PauseMenu instead.

Sending any other value than the ones mentioned will lead to undefined behaviors.

|Value|Description|
|-----|-----------|
|-3|Third party member [Skills List Type](List%20Types%20Group%20Details/Skills%20List%20Type.md)|
|-2|Second party member [Skills List Type](List%20Types%20Group%20Details/Skills%20List%20Type.md)|
|-1|First party member [Skills List Type](List%20Types%20Group%20Details/Skills%20List%20Type.md)|
|0|Standard [Items List Type](List%20Types%20Group%20Details/Items%20List%20Type.md)|
|1|Key [Items List Type](List%20Types%20Group%20Details/Items%20List%20Type.md)|
|2|Storage [Items List Type](List%20Types%20Group%20Details/Items%20List%20Type.md)|
|3|[Medals List Type](List%20Types%20Group%20Details/Medals%20List%20Type.md)|
|9|[Battle Strategy List Type](List%20Types%20Group%20Details/Battle%20Strategy%20List%20Type.md)|
|10|The ordered discoveries [Library List Type](List%20Types%20Group%20Details/Library%20List%20Type.md)|
|11|The ordered bestiary [Library List Type](List%20Types%20Group%20Details/Library%20List%20Type.md)|
|12|The ordered recipes [Library List Type](List%20Types%20Group%20Details/Library%20List%20Type.md)|
|13|The ordered records [Library List Type](List%20Types%20Group%20Details/Library%20List%20Type.md)|
|14|The open [Quest Board List Type](List%20Types%20Group%20Details/Quest%20Board%20List%20Type.md)|
|15|The taken [Quest Board List Type](List%20Types%20Group%20Details/Quest%20Board%20List%20Type.md)|
|16|The completed [Quest Board List Type](List%20Types%20Group%20Details/Quest%20Board%20List%20Type.md)|
|17|The [Settings List Type](List%20Types%20Group%20Details/Settings%20List%20Type.md)|
|18|The [Samira Songs List Type](List%20Types%20Group%20Details/Samira%20Songs%20List%20Type.md)|
|19|The [Key Bindings List Type](List%20Types%20Group%20Details/Key%20Bindings%20List%20Type.md)|
|20|The [Languages list Type](List%20Types%20Group%20Details/Languages%20list%20Type.md)|
|21|The [Overall Quests List Type](List%20Types%20Group%20Details/Overall%20Quests%20List%20Type.md)|
|22|The [Lore Book List Type](List%20Types%20Group%20Details/Lore%20Book%20List%20Type.md)|
|23|The [Stat Bonuses List Type](List%20Types%20Group%20Details/Stat%20Bonuses%20List%20Type.md)|
|24|The [B.O.S.S Battles List Type](List%20Types%20Group%20Details/B.O.S.S%20Battles%20List%20Type.md)|
|25|UNUSED (`listvar` is set to `multilist`)|
|26|The [Termacade Prizes List Type](List%20Types%20Group%20Details/Termacade%20Prizes%20List%20Type.md)|
|27|All items [Enum List Type](List%20Types%20Group%20Details/Enum%20List%20Type.md)|
|28|All medals [Enum List Type](List%20Types%20Group%20Details/Enum%20List%20Type.md)|
|29|All skills [Enum List Type](List%20Types%20Group%20Details/Enum%20List%20Type.md)|
|30|UNUSED (the [Controller Bindings List Type](List%20Types%20Group%20Details/Controller%20Bindings%20List%20Type.md))|
|31|All maps [Enum List Type](List%20Types%20Group%20Details/Enum%20List%20Type.md)|
|32|The [Equipped Medals List Type](List%20Types%20Group%20Details/Equipped%20Medals%20List%20Type.md)|
|33|The [Spy Cards List Type](List%20Types%20Group%20Details/Spy%20Cards%20List%20Type.md)|
|34|The [Caravan Prize Medals List Type](List%20Types%20Group%20Details/Caravan%20Prize%20Medals%20List%20Type.md)|
|35|The [Chompy Ribbons List Type](List%20Types%20Group%20Details/Chompy%20Ribbons%20List%20Type.md)|
