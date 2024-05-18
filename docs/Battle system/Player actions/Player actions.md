# Player Actions
This directory contains all the player actions logic contained in [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md). They are important and complex enough to warrant its own directory in the BattleControl documentation.

A player action is any piece of logic that is specific to the case where the actor sent to DoAction is a player party member. In that case, the actionid is an id that uniquely identifies an action (it may or may not be backed by a [skill](../../Enums%20and%20IDs/Skills.md)) and the actor's information as well as the battle state (such as `selecteditem` for an item usage) will be used alongside the actionid to determine the specific logic the action should be doing.

Anything else is detailed in the DoAction documentation. For the enemy actions, check the [enemy actions](../Enemy%20actions/Enemy%20actions.md) documentation.

## Player actions table
The following is a list of the action per their actionid. Please note the vast majority (but not all) of the actionid doubles as a [skill](../../Enums%20and%20IDs/Skills.md) id, but some of them (such as 9) have a broader meaning and some actionid aren't backed by a skill (such as regular attacks) and thus, doesn't have a corresponding skill. If a skill backs the action, its name and English name will be provided for convenience alongside some notes about its semantics if anything special is worth to mention.

Here is the player actions table:

|ID|Name|English Name|Notes|
|-:|----|------------|-----|
|-2|-|-|The action for using a `LonglegSummoner` [items](../../Enums%20and%20IDs/Items.md), documentation here TODO|
|-1|-|-|Any player party member's basic attacks performed when confirming an [Attack](../Player%20UI/Confirmation%20handling/SelectEnemy.md#attack) in the player UI:<ul><li>[Bee](Basic%20attacks/Bee.md)</li><li>[Beetle](Basic%20attacks/Beetle.md)</li><li>[Moth](Basic%20attacks/Moth.md)</li></ul>|
|0|RESERVED|RESERVED||
|1|RESERVED2|RESERVED||
|2|BeeRangMultiHit|Tornado Toss||
|3|BeetleTaunt|Taunt||
|4|Icefall|Icefall||
|5|BeeFly|Fly Drop||
|6|BeetleDig|Under Strike||
|7|BubbleShield|Bubble Shield||
|8|Empower|Charge Up||
|9|PeebleToss|Pebble Toss||
|10|HornDash|Dash Through||
|11|SecretStash|Secret Stash||
|12|DefenseBreak1|Break||
|13|DefenseBreakAll|Break+||
|14|AttackUp|Empower||
|15|DefenseUp|Fortify||
|16|NeedleToss|Needle Toss||
|17|BubbleShieldLite|Bubble Shield Lite||
|18|HurricaneBeemerang|Hurricane Toss||
|19|PebbleTossPlus|Boulder Toss||
|20|RevivalMassage|Revival Massage||
|21|FrigidCoffin|Frigid Coffin||
|22|ChargeUpPlus|Charge Up+||
|23|EmpowerPlus|Empower+||
|24|NeedlePincer|Needle Pincer||
|25|IceRain|Ice Rain||
|26|IceBeemerang|Frost Relay||
|27|IceDrill|Frozen Drill||
|28|AttackDown|Enfeeble||
|29|AttackDownPlus|Enfeeble+||
|30|DefenseUpPlus|Fortify+||
|31|IceSphere|Frost Bowling||
|32|HeavyStrike|Heavy Strike||
|33|Sturdy|Sturdy||
|34|FieldBeemerang|Beemerang Toss||
|35|FieldHalt|Beemerang Halt||
|36|FieldFly|Bee Fly||
|37|FieldHorn|Horn Slash||
|38|FieldDash|Horn Dash||
|39|FieldDig|Beetle Dig||
|40|FieldFreeze|Freeze||
|41|FieldIcecle|Icicle||
|42|FieldShield|Shield||
|43|PepTalk|Pep Talk||
|44|HeavyThrow|Heavy Throw||
|45|SharingStash|Sharing Stash||
|46|RoyalDecree|Royal Decree||
|47|Cleanse|Cleanse||
|48|HardCharge|Hard Charge||
|49|FirstDash|Dash||