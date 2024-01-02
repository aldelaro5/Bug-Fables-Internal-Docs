# `playerdata` addressing
The player party members are all stored in `playerdata`, but the way the game addresses it is complex and involves multiple layers depending on if it's during a battle or not.

Let's first explain why is there multiple way to address the player party and how it relates to the array itself.

## `playerdata`'s layout
At its code, `playerdata` is an array. This means that no matter what, a member can only be addressed by its index which starts at 0 and ends at the amount of members - 1.

This by itself is very limiting for varying reasons because the game needs to keep track of the player switching the order and the game also needs to sometime address a the party by using an [animid](../Enums%20and%20IDs/AnimIDs.md) instead of a particular position in the formation.

Due to this, 2 fields exists on each `BattleData` that covers addressing everyone in every way at least in the overworld. Here's the fields and what they do:

- `trueid`: The [animid](../Enums%20and%20IDs/AnimIDs.md) of the player party member since the last time ChangeParty was called which creates or recreates the entire party. This value never changes for a particular `playerdata` element until ChangeParty is called which recreates the whole array.
- `animid`: The visually rendered [animids](../Enums%20and%20IDs/AnimIDs.md) in the overworld. This value initially starts as the same as `trueid`, but if the player switches the order, the animids are rotated between all elements of `playerdata`. This means this value not only allows the switch to work since the visual rendering changes accordingly, but it also allows the game to know who is actually in front when taking into account player switches.

To understand better how these fields work, we need to explain how ChangeParty works.

## A general explanation of ChangeParty
MainManager.ChangeParty is a method that takes an array of [animids](../Enums%20and%20IDs/AnimIDs.md) that forms the party. It will completely recreate `playerdata` with the animids sent in the same order they were sent. This is not a trivial operation: there are heavy restrictions imposed on the moment it is possible (usually during [events](../Enums%20and%20IDs/Events.md) or save loading) and it is potentially destructive because `playerdata` gets completely wiped and reinitialised.

Due to this, ChangeParty imposes some rules that lasts until the next time ChangeParty is called (which isn't common as said above):

- The `playerdata` indexes are always in the order the animids was sent on ChangeParty
- The `trueid` fields are always going to have the same value for a given `playerdata` index and they will always match the ones sent to ChangeParty

These 2 rules are always true, no matter what situation the game is in and no mater how many times the player switches whether the player is in the overworld or in battle.

This gives different methods of addressing in the overworld and in battle because the methods in which they allows to switch the formation are very different and mostly not linked.

## Methods of addressing (overworld)
To understand how addressing the party works in the overworld, it must be understood that an overworld party switch only rotates a single field of each `playerdata`: the `animid` field. In a sense, this means this switch is "fake" because no actual data changes besides the `animid` which only changes what's being rendered since it changes the animation controller. As far as the data is concerned, the party hasn't changed since the last ChangeParty, hence why `trueid` is named as such: it is the ACTUAL animid of the member that can never change.

Combined with the rules of ChangeParty mentioned above, it gives 3 ways to address the party in the overworld. Let's explore each of them.

### By `playerdata` index directly
This allows to address the party in formation order. The element 0 for example is always the lead party member while element 1 is always the one following the lead. This happens because the only field that gets changed during a switch is the `animid` field, but the entity itself is unchanged. Since the initial ChangeParty order of the party also never changes, this is what allows the index to refer to party members by their position in the overworld formations.

This method of addressing simply involves indexing the `playerdata` array directly. It is the simplest way to address the party.

### By the player party member's `trueid`
This allows to address the party by [animid](../Enums%20and%20IDs/AnimIDs.md), but in a deterministic order. The order preserved by the `trueid` field is always the animids assigned during ChangeParty. This means the order the `trueid` are relative to their `playerdata` index is fixed, even if the player decides to switch in the overworld or the battle. Incidentally, only the `trueid` are saved on the [save file](../External%20data%20format/Save%20File.md) which means the specific formation isn't saved, only the one of the last ChangeParty.

Addressing this way can take multiple forms since an index needs to be passed after all, but it usually involves looping through the array in index order and fetching the `trueid` for each element which gives this deterministic ChangeParty order.

### By the player party member's `animid`
This allows to address the party by the actual [animid](../Enums%20and%20IDs/AnimIDs.md) that is rendered. This is needed if the game needs to care specifically about the animid AND its position in the formation to determine for example the animid of the member in front taking into account the player switches. Since this field is more volatile, it's not suited for having a deterministic order: this field is used during a player switch and as such, their order relative to player indexes is free to be changed by the player.

Addressing this way can take multiple forms since an index needs to be passed after all, but it usually involves indexing the array which gives a member with a specific formation position and fetch their `animid` which tells which party member it represents.

## Methods of addressing (durring battle)
While the overworld ways of addressing the party are sufficient for the overworld, they aren't a good fit during battle because the game wants to disociate the overworld party order with the battle one. This is required because the actual overworld order must not change after the battle is over for various reasons.

Due to this, 2 fields are introduced to allow to "fake" a different view of the party that is reserved specifically for battles:

- instance.`partyorder`: This array contains the [animids](../Enums%20and%20IDs/AnimIDs.md) ordered by the `animid` fields of `playerdata`'s elements. In other words, it's the rendered overworld party order. It is updated on ChangeParty and everytime the player switches the formation, but is otherwise only used by [StartBattle](StartBattle.md)
- `partypointer`: This array maps battle formation position to `playerdata` indexes and it is always composed of 3 elements (which is the maximum amount of player party members). It is affected during a battle party switch or swap (which occurs as part of [SwitchParty](Battle%20flow/Action%20coroutines/SwitchParty.md) and [SwitchPos](Battle%20flow/Action%20coroutines/SwitchPos.md))

Initially, `partypointer` is initialised to {0, 1, 2} on StartBattle. Later on in that coroutine, it will try to align `partypointer` and `partyorder` so they contain matching arrays with the player party being aligned with this. This alignement is done by calling a fast [SwitchParty](Battle%20flow/Action%20coroutines/SwitchParty.md) repeatedly until they both aligne (no call is done if they already happened to align).

This works because a switch during battle works differently than in the overworld. If the overworld only involved rotating the `animid` fields, in battle, a switch only involves rotating the `partypointer` elements. This alone doesn't switch anything so what [SwitchParty](Battle%20flow/Action%20coroutines/SwitchParty.md) and [SwitchPos](Battle%20flow/Action%20coroutines/SwitchPos.md) do after the switch is to also update the `playerdata` elements with some information, but notably, it changes the position of the `battleentity`. This means that as far as the entities are concerned, a switch truly took place as far as the data are concerned, but as far as the `animid` are concerned, they are left unchanged leaving no visual changes like in the overworld. It just provides a different view of the same `playerdata` array.

This has no effect in the overworld because the `battleentity` isn't the `entity` field for the player party, but it means that the party now aligns with the overworld order without actually changing anything that affects the overworld. This leads to a fundamental rule about `partypointer`: it will always act as a mapper from battle formation position to `playerdata` index.

This works because after [StartBattle](StartBattle.md), `partypointer` will hold the same order as how the party was visually in the overworld, but it's now completely disconnected from the overworld party formation. This means that as long as formation position are addressed using the `partypointer`, it will always match with battle formation order. It also means that BattleControl cannot mess with the `animid` fields for player party members because changing those will affect the overworld.

This leads to different ways to address the party. Let's explore them.

### By `playerdata` index directly
Unlike in the overworld, this addressing method is much less useful because it gives the player party member at a specific overworld position still (they aren't changed), but this is obviously not interesting in battles.

Due to this, this mode of addressing is specific to just address all player party members regarless of anything such as in a standard loop as it's still the simplest way to address the party.

### By the player party member's `trueid`
Addressing by `trueid` becomes much more interesting in battle because since the party order didn't changed any `animid` fields, it means `partypointer` resolves to the `trueid` and it now means it acomplishes a similar goal than what the overworld `animid` addressing means.

In practice, it means the `trueid` can be thought of as the same than the actual `animid` field. In fact, the battleentity.`animid` values originally get set to the `playerdata`'s `trueid` values on [StartBattle](StartBattle.md). Essentially, anything that required a specific animid (such as checking a if a [medal](../Enums%20and%20IDs/Medal.md) is equipped on a member with a particular animid) is best suited for this addressing method.

### By `partypointer`
Another benefit of `partypointer` is this array allows to map battle formation order to `playerdata` order. This means that for example `playerdata[partyorder[0]]` always return the player party member in the front in battle irregardless of the overworld formation while `playerdata[partyorder[playerdata.length - 1]]` always return the one on the back. This addressing method is best suited where the battle formation matters such as targetting. 

A benefit to this is it simplifies the procedure of [SwitchParty](Battle%20flow/Action%20coroutines/SwitchParty.md) and [SwitchPos](Battle%20flow/Action%20coroutines/SwitchPos.md) because all that's needed is to change the `partypointer` elements and update the battleentities accordingly (mainly their position).

One thing to be cautious of: `partypointer` always has 3 elements, but there might not be 3 `playerdata` elements. If there's 2 or 1, care must be taken to check that the `partypointer` index falls within the bounds of `playerdata` first before addressing this way.
