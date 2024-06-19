# Known issues of the damage pipeline
The damage pipeline of the game while known to have multiple logic and calculation errors on top of the many errors that exists within the action logic themselves (all are documented and mentioned when relevant), there are also some design issues. These issues aren't necessarilly caused by a programming mistake and more that the pipeline in general wasn't designed with what the game needed to perform its logic. These are more engineering problems that either prevents features to work well or requires imperfect workarounds. They are important enough to have affected a lot of the design decisions and are complex enough that it's worth to document them here in a separate page.

## It is impossible to call [DoDamage](DoDamage.md) using multiple [AttackProperty](AttackProperty.md)
The parameter to DoDamage takes an `AttackProperty?` meaning either there is a property or no property denoted by the value `null`.

This is very problematic because it comes up a lot during [skills](../../Enums%20and%20IDs/Skills.md) based [player actions](../Player%20actions/Player%20actions.md) that the game wants to send multiple properties. For example: ideally, [BeetleDig](../Player%20actions/Skills/BeetleDig.md) should have `Flip` and `Pierce` because it is an attack that ignores defense, but also can inflict [Flipped](../Actors%20states/BattleCondition/Flipped.md) to relevant enemy party members. Since it's not possible to send multiple properties, the action ends up sending `Pierce`, but compensate by doing what the `Flip` property logic would have done inside the action logic. Essentially, this ends up almost duplicating code: the logic that should have been reused from DoDamage is now duplicated inside the `BeetleDig` action logic.

This can lead to strange inconsistencies because the logic wasn't duplicated correctly or the original changed, but not the copy. As an example, `BeetleDig` doesn't cause a stolen item to be dropped, but [Beetle's basic attack](../Player%20actions/Basic%20attacks/Beetle.md) will. This is because the basic attack has an actual `Flip` property which is the correctly logic, but `BeetleDig` has an incorrect duplicate of it. This is something that recurs a lot with many actions affected.

## It is impossible to tune what happens to the total damage once [DoDamage](DoDamage.md) is called
This one more relates to a heavy restriction related to game balancing.

The damageammount parameter is the only value that is tunable from the caller of the damage pipeline which usually is an action logic. This only partially covers the effects that a player can control which can often lead to damage amounts that add up very quickly, particularily with actions that contain multiple [DoDamage](DoDamage.md) calls.

There is an intuitive distinction between what the parameter covers and what it doesn't. Anything that influences the `atk` value of the attacker is controllable, but anything that doesn't isn't. For a player party member, the `atk` value can be consulted easily in the pause menu inside the "Medals & Skills" window.

This creates heavy balancing restrictions because it now means that depending on the source of a damage bonus, it might be tunable by the game or be left completely unrestricted. A prime example of this is [HardCharge](../Player%20actions/Skills/HardCharge.md): it offers 3 `charges` which increases the final damage output by 3, but the actions logic cannot tune this. It almost ALWAYS increases the result by 3, even if an action has 4 DoDamage calls, each will still have their results increased by 3 which leads to a grand total of 12 more damages that happened solely because of having 3 `charges`. This specific case happens to occur in the case of [BeeRangMultiHit](../Player%20actions/Skills/BeeRangMultiHit.md), [HurricaneBeemerang](../Player%20actions/Skills/HurricaneBeemerang.md) and [IceRain](../Player%20actions/Skills/IceRain.md).

The overall effect of this issue is that it is impossible to effectively control what happens with any multi hits skills. Their output is only partially controlled by the game and the part that it can't control is something the player can. This disproportionally benefits players who uses a heavy amount of damage bonuses that do not affect their `atk`, but rather are applied during the damage pipeline. Since the game already offers many sources of these, it can easily get out of hand. Since it is how the damage pipeline works by design, this can't be addressed without a complete refactor.

There are however known mitigations that the game tried to do, but aren't perfect. One example would be heavily dampening the amount to send to DoDamage on each hits of [IceBeemerang](../Player%20actions/Skills/IceBeemerang.md). This however has the very strange outcome that even if the player has a higher `atk` value, their damage will get clamped down to 0 relatively fast, but if they have a lot of bonuses outside of their `atk`, they stack up REALLY fast to the point of defeating most enemies after the action. In other words: it's either low damage or high damage and there's not much in between.

## It is impossible to have more than one attacker
This only affects [MultiSkillsMove](../Actors%20states/MultiSkillMove.md) except for [IceBeemerang](../Player%20actions/Skills/IceBeemerang.md) which means any action where multiple player party members combine their `atk` to deal damage.

[DoDamage](DoDamage.md) only support a limited about of attack flow:

- No attacker to player party member
- No attacker to enemy party member
- Player party member to enemy party member
- Enemy party member to player party member

However, it doesn't support having multiple attackers. This means that player actions where multiple player party members are involved can't really work as is with the current damage pipeline. It's possible to do what [IceBeemerang](../Player%20actions/Skills/IceBeemerang.md) did and have each player party members attack individually, but it's often not desired. What would be ideal is a way to have members combine their `atk` and benefits together to compute the final amount of damage to deal.

Since this isn't possible, there is a workaround available called [GetMultiDamage](GetMultiDamage.md) which allows to send the [trueid](../playerdata%20addressing.md#by-the-player-party-members-trueid) of the player party members involved and it will precalcucale most attack benefits on top of their `atk`. From there, this resulting amount is sent to DoDamage, but without an attacker. This essentially "forces" the damage amount to not be affected by the `atk` or their attack benefits.

This workaround however comes with a lot of caveats which are outlined in the documentation page about the method. It also has a strange interaction with the balancing restrictions mentioned above: since it is the only way the game can sort of dampen the total damage dealt, it makes these move deal a lot less damages than other multi hits coming from a single player party member. This has the side effect to decentivise their usage against the multi hits one which the player can exploit the balancing restrictions.
