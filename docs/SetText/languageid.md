# languageid

MainManager.languageid represents the current language's ID. Bug Fables is playable in 6 languages as of 1.1.2. A language is represented by a number which is also the suffix number to identify its [dialogue folder](../TextAsset%20Data/Dialogue%20data.md). This influences the logic of [SetText](SetText.md) and [OrganiseLines](Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md).

## Language ID table

|Name|ID|
|----|--|
|`English`|0|
|`Spanish`|1|
|`Portuguese`|2|
|`Japanese`|3|
|`German`|4|
|`Korean`|5|

## Definition of a languageid

Here are the components that defines a language:

* A name hardcoded in the `languagenames` array whose index is the languageid
* Its own directory in `Ressources/data` `dialogue` followed by the languageid. It contains language specific data notably [Dialogue data](../TextAsset%20Data/Dialogue%20data.md), names/descriptions of many different elements and others which are detailed further in the TextAsset folder of this documention
* Several logic changes in [SetText](SetText.md) and UI rendering (this is checked using the languageid directly) such as [OrganiseLines](Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md)

The languageid is saved in the config file and the game will prompt for one if it isn't set, but it defaults to English on first boot. It can only be changed using the language selection screen which has its own [listtype](../ItemList/listtype.md) called [Languages list Type](../ItemList/List%20Types%20Group%20Details/Languages%20list%20Type.md).
