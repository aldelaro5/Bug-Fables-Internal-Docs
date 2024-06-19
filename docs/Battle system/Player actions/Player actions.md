# Player Actions
This directory contains all the player actions logic contained in [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md). They are important and complex enough to warrant its own directory in the BattleControl documentation.

A player action is any piece of logic that is specific to the case where the actor sent to DoAction is a player party member. In that case, the actionid is an id that uniquely identifies an action (it may or may not be backed by a [skill](../../Enums%20and%20IDs/Skills.md)) and the actor's information as well as the battle state (such as `selecteditem` for an item usage) will be used alongside the actionid to determine the specific logic the action should be doing.

Anything else is detailed in the DoAction documentation. For the enemy actions, check the [enemy actions](../Enemy%20actions/Enemy%20actions.md) documentation.

## Player actions table
The following is a list of the action per their actionid. Please note the vast majority (but not all) of the actionid doubles as a [skill](../../Enums%20and%20IDs/Skills.md) id, but some of them (such as 9) have a broader meaning and some actionid aren't backed by a skill (such as regular attacks) and thus, doesn't have a corresponding skill. If a skill backs the action, its name and English name will be provided for convenience alongside some notes about its semantics if anything special is worth to mention.

Here is the player actions table:

|ID|Name|English Name|Notes|
|-:|----|------------|-----|
|-2|-|-|The action for using a `LonglegSummoner` [items](../../Enums%20and%20IDs/Items.md). This action is documented [here](Items%20usage/LonglegSummoner.md)|
|-1|-|-|Any player party member's basic attacks performed when confirming an [Attack](../Player%20UI/Confirmation%20handling/SelectEnemy.md#attack) in the player UI:<ul><li>[Bee](Basic%20attacks/Bee.md)</li><li>[Beetle](Basic%20attacks/Beetle.md)</li><li>[Moth](Basic%20attacks/Moth.md)</li></ul>|
|0|RESERVED|RESERVED|This action is UNUSED|
|1|RESERVED2|RESERVED|This action is UNUSED|
|2|[BeeRangMultiHit](Skills/BeeRangMultiHit.md)|Tornado Toss||
|3|[BeetleTaunt](Skills/BeetleTaunt.md)|Taunt||
|4|[Icefall](Skills/Icefall.md)|Icefall||
|5|[BeeFly](Skills/BeeFly.md)|Fly Drop||
|6|[BeetleDig](Skills/BeetleDig.md)|Under Strike||
|7|[BubbleShield](Skills/BubbleShield.md)|Bubble Shield||
|8|[Empower](Skills/Empower.md)|Charge Up||
|9|[PeebleToss](Skills/PeebleToss.md)|Pebble Toss|This is also the action used for every toss item usage other than `LonglegSummoner`. Anything not in the following list will default to the [default item toss action](Items%20usage/Default%20toss%20action.md):<ul><li>[abombhoney](Items%20usage/Abombhoney.md)</li><li>[BurlyBomb](Items%20usage/BurlyBomb.md)</li><li>[CherryBomb](Items%20usage/CherryBomb.md)</li><li>[ClearBomb](Items%20usage/ClearBomb.md)</li><li>[FlameRock](Items%20usage/FlameRock.md)</li><li>[FrostBomb](Items%20usage/FrostBomb.md)</li><li>[HardSeed](Items%20usage/HardSeed.md)</li><li>[Ice](Items%20usage/Ice.md)</li><li>[NumbBomb](Items%20usage/NumbBomb.md)</li><li>[NumbDart](Items%20usage/NumbDart.md)</li><li>[PoisonBomb](Items%20usage/PoisonBomb.md)</li><li>[PoisonDart](Items%20usage/PoisonDart.md)</li><li>[SleepBomb](Items%20usage/SleepBomb.md)</li><li>[SpicyBomb](Items%20usage/SpicyBomb.md)</li></ul>|
|10|[HornDash](Skills/HornDash.md)|Dash Through||
|11|[SecretStash](Skills/SecretStash.md)|Secret Stash||
|12|[DefenseBreak1](Skills/DefenseBreak1.md)|Break||
|13|[DefenseBreakAll](Skills/DefenseBreakAll.md)|Break+||
|14|[AttackUp](Skills/AttackUp.md)|Empower||
|15|[DefenseUp](Skills/DefenseUp.md)|Fortify||
|16|[NeedleToss](Skills/NeedleToss.md)|Needle Toss||
|17|[BubbleShieldLite](Skills/BubbleShieldLite.md)|Bubble Shield Lite||
|18|[HurricaneBeemerang](Skills/HurricaneBeemerang.md)|Hurricane Toss||
|19|[PebbleTossPlus](Skills/PebbleTossPlus.md)|Boulder Toss||
|20|RevivalMassage|Revival Massage|There is no action for this skill|
|21|[FrigidCoffin](Skills/FrigidCoffin.md)|Frigid Coffin||
|22|[ChargeUpPlus](Skills/ChargeUpPlus.md)|Charge Up+||
|23|[EmpowerPlus](Skills/EmpowerPlus.md)|Empower+||
|24|[NeedlePincer](Skills/NeedlePincer.md)|Needle Pincer||
|25|[IceRain](Skills/IceRain.md)|Ice Rain||
|26|[IceBeemerang](Skills/IceBeemerang.md)|Frost Relay||
|27|[IceDrill](Skills/IceDrill.md)|Frozen Drill||
|28|[AttackDown](Skills/AttackDown.md)|Enfeeble||
|29|[AttackDownPlus](Skills/AttackDownPlus.md)|Enfeeble+||
|30|[DefenseUpPlus](Skills/DefenseUpPlus.md)|Fortify+||
|31|[IceSphere](Skills/IceSphere.md)|Frost Bowling||
|32|[HeavyStrike](Skills/HeavyStrike.md)|Heavy Strike||
|33|[Sturdy](Skills/Sturdy.md)|Sturdy||
|34|FieldBeemerang|Beemerang Toss|There is no action for this skill|
|35|FieldHalt|Beemerang Halt|There is no action for this skill|
|36|FieldFly|Bee Fly|There is no action for this skill|
|37|FieldHorn|Horn Slash|There is no action for this skill|
|38|FieldDash|Horn Dash|There is no action for this skill|
|39|FieldDig|Beetle Dig|There is no action for this skill|
|40|FieldFreeze|Freeze|There is no action for this skill|
|41|FieldIcecle|Icicle|There is no action for this skill|
|42|FieldShield|Shield|There is no action for this skill|
|43|[PepTalk](Skills/PepTalk.md)|Pep Talk||
|44|[HeavyThrow](Skills/HeavyThrow.md)|Heavy Throw||
|45|[SharingStash](Skills/SharingStash.md)|Sharing Stash||
|46|[RoyalDecree](Skills/RoyalDecree.md)|Royal Decree||
|47|[Cleanse](Skills/Cleanse.md)|Cleanse||
|48|[HardCharge](Skills/HardCharge.md)|Hard Charge||
|49|FirstDash|Dash|There is no action for this skill|
