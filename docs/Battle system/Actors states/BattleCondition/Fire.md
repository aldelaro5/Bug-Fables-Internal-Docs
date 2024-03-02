# Fire
A condition that deals non lethal damage (more than [Poison](Poison.md)) every actor turn while it is active that may be inflicted in [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md) if property is `Fire` (without a resistance field) featuring a `Prefabs/Particles/Flame` as a `firepart`.

## [SetCondition](../Conditions%20methods/SetCondition.md)
When amending the condition, the turn counter is always increased by absolute value of the sent one / 2.0 ceiled (this means it stacks, but by around half of the sent turns amount).

When inflicted as a new condition, nothing special happens.

## [CalculateBaseDamage](../../Damage%20pipeline/CalculateBaseDamage.md)
This condition may be inflicted if the property is `Fire`. This means it's also supported by the `StatusMirror` [medal](../../../Enums%20and%20IDs/Medal.md). It does not use a resistance field, check the method's documentation to learn more on the conditions needed for the infliction to work.

## [AdvanceTurnEntity](../../Battle%20flow/AdvanceTurnEntity.md)
When the condition is processed (when `hp` is above 0 and there is at least one enemy party member):

- The `Flame` sound is played
- [DoDamage](../../Damage%20pipeline/DoDamage.md) is called to the actor with no attacker with a `NoExceptions` property without block. The damageammount is the player party member's `maxhp` / 7.5 ceiled - 1 and then clamped from 2 to 3. The call also has these overrides:
    - `NoFall`,
    - `NoIceBreak`,
    - `FakeAnim`,
    - `DontAwake`,
    - `IgnoreNumb`
- If the actor's `hp` became 0, battleentity.`overrideanim` is set to false followed by an OverrideOver call being invoked on the battleentity in 1.0 seconds
- The actor's `hp` is clamped from 1 to its `maxhp`
- A yield of 0.75 seconds is set to happen after the method is done

## [UpdateEntities](../../Visual%20rendering/UpdateEntities.md)
This condition will have the battleentity.`sprite`.material.color to be set to a lerp from the existing one to 0xBF5919 (bright orange) with a factor of 1/5 of the frametime. Check the documentation of the method to learn more about how the alpha channel is determined.

This condition includes the initialisation of the actor's `firepart` as a new instance of `Prefabs/Particles/Flame` childed to the `sprite` with a `DelAftBtl` tag and a local position of Vector3.up.

## [UpdateAnim](../../Visual%20rendering/UpdateAnim.md)
This condition will cause the `currentturn` player party member to have its battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) set to 20 (`WeakPickAction`) instead of (`PickAction`) when its battleentity.`overrideanim` is false.

This condition may cause the other player party members to have their battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) set to 17 (`WeakBattleIdle`) instead of 13 (`BattleIdle`). Check the method's documentation to learn more on the specifics.
