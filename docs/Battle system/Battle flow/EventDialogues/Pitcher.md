# [Pitcher](../../Enemy%20actions/Enemies/Pitcher.md) EventDialogue
`Pitcher` has an EventDialogue that is meant to trigger as their [eventondeath](../../Actors%20states/Enemy%20features.md#eventondeath) or when the [Eaten](../../Actors%20states/BattleCondition/Eaten.md) condition turn advance found that the player party member just died after sustaining damages.

If it is triggered as a result of the death, it ensures that all `enemydata` dies such as the [PitcherFlytrap](../../Enemy%20actions/Enemies/PitcherFlytrap.md) and if it is triggered as a result of the `Eaten` condition, it spits out the player party member.

## Logic sequence

- The `enemydata` index of `Pitcher` is obtained
- If `Pitcher`'s `hp` is 0 or below:
    - Their `eventondeath` is set to -1 which disables this EventDialogue and prevents [CheckDead](../Action%20coroutines/CheckDead.md) from triggering it again
- If `Pitcher`'s `ate` isn't null (they are currently eating a player party member):
    - `spitout` is set to a new [SpitOut](../../Actors%20states/BattleCondition/Eaten.md#spitout) call with `Pitcher` as the eater
    - Yield for 0.5 seconds
    - `gottaspit` set to false
    - Yield all frames until `spitout` is null (the coroutine completed)
    - If `Pitcher`'s `hp` is above 0, their `ate` is set to null (so they aren't eating anyone)

From there, the logic only continues if `Pitcher`'s `hp` is 0 or below where all `enemydata` will be killed and the battle ended:

- All `enemydata` other than `Pitcher` whose `deathcoroutine` is null has their `deathcoroutine` set to a new [Death](../../../Entities/EntityControl/Notable%20methods/Death.md) call with activatekill followed by their `hp` set to 0
- If `alreadyending` is false ([AddExperience](../Terminal%20coroutines/AddExperience.md) wasn't called before), [EndBattleWon](../Terminal%20wrappers/EndBattleWon.md) is called with addexp and no skipids which switches the battle to a [terminal flow](../Update%20flows/Terminal%20flow.md)
