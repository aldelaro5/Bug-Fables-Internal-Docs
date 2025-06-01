# [BoardQuests](../../Enums%20and%20IDs/BoardQuests.md) board list type

Display the list of open, taken or completed [BoardQuests](../../Enums%20and%20IDs/BoardQuests.md) on the quest board. This is not to be confused with [Overall Quests List Type](Overall%20Quests%20List%20Type.md) which renders a subset of all the quests in one list.

## Options generation

`listvar` is the return of GetQuestsBoard(`listtype` - 14) which are all the [BoardQuests](../../Enums%20and%20IDs/BoardQuests.md) id contained in the `boardquests` depending on the board type of the list (open, taken or completed) with some restrictions that apply if the current map is not TestRoom:

* Leif's Request and Vi's Request are excluded (NOTE: Kabbu's Request should have been excluded, but its exclusion was forgotten).
* Any story request (id between 11 and 17) are excluded.
* Any of the following bounty requests are excluded if the current map isn't UndergroundBar and the board type of the list is the open quests:
    * Bounty: Seedling King
    * Bounty: False Monarch
    * Bounty: Devourer
    * Bounty: Tidal Wyrm
    * Bounty: Peacock Spider
    
If the end result has no quests, add quest id 0 (No Quests) as the only quest offered as option.

`listredirect` is overridden to null.

## Option's [SetText](../../SetText/SetText.md) input string

The text is the name of the corresponding quest from [boardquestdata](../../TextAsset%20Data/BoardQuests%20data.md#boardquests-data) prepended with |size,0.5,0.8,lock| in `German` or |[size](../../SetText/Individual%20commands/size.md),0.6,0.8,lock| in `Japanese`.

The x position of the text is overridden to 0.0 and the size to Vector2.one.

## Description box rendering

`listdescbox` is rendered using a custom rendering scheme. If the quest id isn't No Quest, an image of the quest's author is rendered using `librarysprites` from the index obtained in [boardquestdata](../../TextAsset%20Data/BoardQuests%20data.md#boardquests-data) which will have a name of `Image` and a tag of `Text` towards the top of the description box. Additionally [SetText](../../SetText/SetText.md) is called with the following in non [Dialogue mode](../../SetText/Dialogue%20mode.md):

* `text`: |[size](../../SetText/Individual%20commands/size.md),0.75||[sort](../../SetText/Individual%20commands/Sort.md),1| + The `By:` from MenuText line id 104 + ` ` + The author of the quest obtained from boardquestdata + |[line](../../SetText/Individual%20commands/Line.md)\||[halfline](../../SetText/Individual%20commands/Halfline.md)\| + The `Difficulty:` from MenuText line id 105 + ` ` + |[Stars](../../SetText/Individual%20commands/Stars.md), + The amount of filled in stars of the quest obtained from boardquestdata + `|`
* `position`: (12, 0.35)
* `parent`: `listdescbox`

Whether or not the quest was No Quest, SetText is called in non [Dialogue mode](../../SetText/Dialogue%20mode.md) with the following:

* `text`: |[sort](../../SetText/Individual%20commands/Sort.md),1||[single](../../SetText/Individual%20commands/Single.md)\| + |[singlebreak](../../SetText/Individual%20commands/Singlebreak.md),10||[Sizemulti](../../SetText/Individual%20commands/Sizemulti.md),0.8,1| on `German` or  |[singlebreak](../../SetText/Individual%20commands/Singlebreak.md),6| otherwise + the quest's description from boardquestdata before any `}` or `{`
* `fonttype`: `BubblegumSans`
* No linebreak
* No tridimensional
* `position`: (9.9, -1.75)
* No cameraoffset
* `size`: (0.65, 0.75)
* `parent`: `listdescbox`
* No caller

## Input handling

On top of the default input handling, this listtype overrides the handling of the left and right input to handle changing pages in the `questboardobj`.

When pressing left or right, the PageFlip sound is played followed by a reset of the [ItemList State Machine](../ItemList%20State%20Machine.md) where the visible part, cursor and `option` are put back to the topmost item with `listy` set to -1. 

If left was pressed, [listtype](../listtype.md) is decremented, but it will be set to 16 (completed quest board type) if it was set to open quest board type.

If right was pressed, [listtype](../listtype.md) is incremented, but it will be set to 14 (open quest board type) if it was set to completed quest board type.

In either cases, `questboardobj` will be updated to reflect this page change and [ShowItemList](../ShowItemList.md) will be called using the new [listtype](../listtype.md), the existing `listpos` with `showdescription` to true and `sell` to false.

## Confirmation handling

If type is the open quests board and a `boardcaller` exists, the quest must not be No Quest or a buzzer sound will be played and the confirmation rejected.

If it is not No Quest, the `questboardobj` is closed which adds an `actioncooldown` of 20 frames, sets `minipause` to false and `inlist` to false. From there, every player entity is set to face towards the entity of the boardcaller. SetText is then called in [Dialogue mode](../../SetText/Dialogue%20mode.md) with the [dialogue line id](../../SetText/Common%20commands%20id%20schemes/Dialogue%20line%20id.md) of the boardcaller (it's stored in its `data[1]` field, see the [BoardQuest](../../Entities/NPCControl/Interaction/QuestBoard.md) interaction for more details). That test gets prepended with `|`[questprompt](../../SetText/Individual%20commands/Questprompt.md)`|`. The entity of the boardcaller is the parent and its NPCControl is the caller. [Flagvar](../../Flags%20arrays/flagvar.md) 0 is set to the selected quest id and the list is finally destroyed which ends this confirmation handling.

On the other hand, if the type isn't the open quest board and the current map is the TestRoom, the selected quest is removed from the corresponding board, the `questboardobj` is closed (which adds an `actioncooldown` of 20 frames, sets `minipause` to false and `inlist` to false) and the list is destroyed which ends this confirmation handling.

## Other Behaviors

The parent of [ItemList](../ItemList.md) is set to the `questboardobj` instead of the GUICamera for this [listtype](../listtype.md) as it is expected to not be null under normal gameplay. This means the list is rendered on the board rather than on the root of the GUI.
