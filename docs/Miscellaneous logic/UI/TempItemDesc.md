# TempItemDesc
A component meant to be attached on any GameObject via Unity.

This component is only used in the `TermiteIndustrial` map to implement the flow of purchasing the `BombPlus` medal. Some explanation is in order for why such a component is needed.

In order for the game to render a description box near a shelved item or medal, a shop system needs to be defined. This will handle everything from creation to the purchasing flow and it includes rendering description boxes when in the interact range of the SemiNPC shelved items / medals. So in theory, it could be possible to define a medal shop with only 1 medal, but in practice, it would be much more complicated. This is because defining a medal shop has great implications due to the game making assumpions on how they work. For example, it would have an impact on the save file because it contains data about the medals shops's stock and it would .

Since it's not possible to add a medal shop with 1 medal, this component was made to compensate. It "fakes" the presence of a description box when going near the GameObject with configurable fields even if it is only used for a single occasion:

- `requireEntity`: A map entity entity id that needs to be present (not null) for the component to remain enabled
- `insideID`: The inside id that the GameObject is in (used to see if the player gets close enough as the inside id needs to match too). If this is -1, the GameObject isn't in any insides
- `itemType`: The general type of the item/medal refered to by the GameObject. 0 means standard item, 1 means key item and 2 means medal
- `itemID`: The item or medal id refered to by the GameObject
- `itemValue`: A buying price in berries to show in the description box
- `range`: The maximum Vector3.Distance that the MainManager.`player` can be away from the GameObject while still having the description box rendered
- `itemSprite`: The sprite to show as the item/medal to buy. If `itemType` is 2 (medal) and flags 681 is true (MYSTERY? is active), this is overriden to MainManager.`guisprites[190]` (the ? gray medal sprite)

This component only handles rendering the description box, it doesn't handle the actual purchase, but this can be done using an Event interaction on an NPCControl.

The component has a Start that will enforce the MYSTERY? sprite mentioned above, but also disable the component so no description box gets rendered on MYSTERY? (this is inconsistent with the rest of the game). It also has an OnDestroy that will destroy the description box UI.

Finally, it has an Update where most of the logic happens including the `range` and `insideID` checks.
