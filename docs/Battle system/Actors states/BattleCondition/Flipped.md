# Flipped
An ephemeral condition specific to enemy party members that is only inflicted with a `Flip` [AttackProperty](../../Damage%20pipeline/AttackProperty.md) to an enemy party member with a `Flip` [weakness](../../Damage%20pipeline/AttackProperty.md). It causes all the enemy party member's `def` to be ignored during damage calculation which is recognised by [TrueDef](../../Visual%20rendering/RefreshEnemyHP.md). This process can be hampered by [Topple](Topple.md)'s defense mechanism if it applies. NOTE: The condition logic has caveats, check the [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) documentation to learn more.

## [IsStopped](../IsStopped.md)
This condition is considered a stop condition and will always make this method returns true (unless skipimmobile is false while the actor'a [actimmobile](../Enemy%20features.md#actimmobile) is true). Being stopped makes the actor unable to act regardless of their `cantmove` as well as a bunch of feature they no longer get access to.

This however does not include the lite version used in the [enemy phase](../../Battle%20flow/Main%20turn%20life%20cycle.md#enemy-phase) meaning enemy party members are allowed to act even with this condition. However, this will only have an effect if there are more than 1 main turns left on the condition because as explained further in the DoAction logic below, if only 1 main turn is left, DoAction will remove the condition immediately and the enemy party member gets to act. DoAction is called regardless, it's just that if there were more than 1 main turns, the enemy action logic part won't happen.

## [Relay](../../Battle%20flow/Action%20coroutines/Relay.md)
This condition will not be transfered to the target of the relay if the relayer had a `RelayTransfer` [medal](../../../Enums%20and%20IDs/Medal.md). It should be noted that it is not possible to inflict this condition on a player party member under normal gameplay so this clause effectively does nothing.

## [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md)
This condition will cause the battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) to be set to 25 (`SleepFallen`) instead of 14 (`Sleep`) if a [Sleep](Sleep.md) infliction occurs.

If property is `Flip` while the target has this condition, it will change the logic to call DefaultDamageCalc instead of the standard 1 `def` ignore logic that the property normally has. In DefaultDamageCalc, this condition will actually prevent the damage to be decreased by the target's `def` which means the overall effect is this condition causes the `def` to be ignored (but not other forms of defenses). This is recognised by [TrueDef](../../Visual%20rendering/RefreshEnemyHP.md#truedef), but this comes with caveats, check the method's documentation to learn more.

If this condition wasn't present, but property is `Flip` while the target has a `Flip` [weakness](../../Damage%20pipeline/AttackProperty.md), a weaknesshit is recognised and the condition will be inflicted for 1 turn unless the target has the `ToppleFirst` [weakness](../../Damage%20pipeline/AttackProperty.md) without a [Topple](Topple.md) condition (since this is where [Topple](Topple.md) is inflicted instead as a defensive mechanism). In the case where `ToppleFirst` exists AND it already has [Topple](Topple.md), Topple is removed and `Flipped` is added in its place. For more information, check the method's documentation.

## [AdvanceTurnEntity](../../Battle%20flow/Action%20coroutines/AdvanceMainTurn.md)
When the condition is processed (when `hp` is above 0), if there's more than 1 turn left on it, the actor turn count is decremented. Otherwise, this condition isn't processed in any way meaning it exceptionally doesn't feature a normal actor turn decrease (it is always manually removed).

## [DoAction](../../Battle%20flow/Action%20coroutines/DoAction.md)
Before an enemy party member acts, if it has this condition or [Topple](Topple.md) for exactly 1 actor turn left, both are removed alongside the following logic:

- [Jump](../../../Entities/EntityControl/EntityControl%20Methods.md#jump) is called on the enemy party member with a height of 10.0
- entity.`overrideanim` is set to false
- entity.`basestate` and entity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 0 (`Idle`)
- startstate is set to 0 (`Idle`)
- 0.1 seconds are yielded

However, if more than 1 main turns is left, the entire enemy action is skipped and the coroutine proceeds to [post action](../../Battle%20flow/Action%20coroutines/DoAction.md#post-action).

## [ClearStatus](../Conditions%20methods/ClearStatus.md)
This condition is excluded from removal meaning it will remain even after calling this method.

## [DoDamageAnim](../../Visual%20rendering/DoDamageAnim.md)
This condition may cause the actor's battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) and `basestate` to be set to 15 (`Fallen`) alongside setting `overrideanim` to false. Check the method's documentation to learn more.

## [UpdateAnim](../../Visual%20rendering/UpdateAnim.md)
For all alive (`hp` above 0) enemy party members with this condition and whose `droproutine` isn't in progress, their battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 15 (`Fallen`) (or 25 (`SleepFallen`) if they also have the [Sleep](Sleep.md) condition).

## [Numb](../../../Entities/EntityControl/EntityControl%20Methods.md#numb)
For every enemy party member  with this condition, every 60 frames, their [animstate](../../../Entities/EntityControl/Animations/animstate.md) gets set to 16 (`HurtFallen`) instead of 11 (`Hurt`).
