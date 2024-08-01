# `firststrike` system
There is an exceptional case to the regular battle flow where half a main turn can be processed completely while [StartBattle](../StartBattle.md) is ongoing. This is the case where `adv` had a value of 3 which allows the enemy party to perform what is called a `firststrike`.

A `firststrike` is a main turn that occurs before the first one recognised by the battle system. It's a special main turn because the [player phase](Main%20turn%20life%20cycle.md#player-phase) is completely skipped. Only the [Enemy phase](Main%20turn%20life%20cycle.md#enemy-phase) and the [end of turn phase](Main%20turn%20life%20cycle.md#turn-end-phase) occurs on that main turn.

The way it works is when StartBattle receives an adv of 3, it setups the battle like the following:

- `enemy` is set to true
- `action` is set to true
- `firststrike` is set to true
- [DoAction](Action%20coroutines/DoAction.md) is called with `enemydata[0]`
- Yield until `action` goes to false

The overall effect is it changes the battle flow to an [uncontrolled flow](Update%20flows/Uncontrolled%20flow.md) as if the battle was in the enemy phase with the first enemy party member taking their actor turn.

However, there is one key difference with this actor turn: `firststrike` is set to true. This has some implications:

- The [enemy action](Action%20coroutines/DoAction.md#enemy-action) can behave differently depending on if they are in a `firststrike` or not
- DoAction will not call a [CheckDead](Action%20coroutines/CheckDead.md) in [post action](Action%20coroutines/DoAction.md#post-action) when no enemy fled 
- DoAction won't froce drops the enemy party members in [post action](Action%20coroutines/DoAction.md#post-action)

In a normal situation, DoAction gets done, `action` gets set to false and the battle switches to a [controlled flow](Update%20flows/Controlled%20flow.md). However, keep in mind that StartBattle is yielding until `action` goes to false so at this point, StartBattle is ready to take over from now.

Except that's not going to happen immediately due to coroutine's lower priority.

## What happens after the first enemy action?
StartBattle is a corotuine. It means that no matter what happens, Update gets processed BEFORE it resumes. It means that Update does get a chance to run before StartBattle realises that the battle already set `action` to false and can continue.

Update however won't let StartBattle resume right away. Since StartBattle setup the battle system in such a way that the system already started and setup, it will act like normal. On that next Update, it will find that `enemydata[0]` spent an actor turn, but it will also process the next enemy actor turn available. Effectively, the battle proceeds like it would normally in the enemy phase despite the fact StartBattle isn't done and is actually still waiting!

`firststike` is still set to true when the enemy phase goes on like this. This gives an additional difference to a regular main turn: DoAction will NOT let other enemy party members act other than the first one if the `DoublePainReal` [medal](../../Enums%20and%20IDs/Medal.md) isn't equipped and [flags](../../Flags%20arrays/flags.md) 614 is false (HARDEST is inactive). This is a difficulty scaling feature: on hard mode or HARDEST, the entire enemy party member can act during a `firststrike`, but without either applying, only the first can act during a `firststrike`. If the action is skipped, DoAction will process as if the enemy did nothing in their action, but still consume their actor turn like normal.

This keeps going until all enemy actor turns were consumed. All enemy actor turns will be consumed before StartBattle resumes because it only takes a single Update cycle to process the next enemy action. Effectively, the battle system switches back and forth between [controlled flow](Update%20flows/Controlled%20flow.md#controlled-flow) and [uncontrolled flow](Update%20flows/Uncontrolled%20flow.md#uncontrolled-flow).

## A special main turn advance
Since all enemy actor turns gets processed, it eventually leads the main turn to be in the [turn end phase](Main%20turn%20life%20cycle.md#turn-end-phase). This will call [AdvanceMainTurn](Action%20coroutines/AdvanceMainTurn.md) preventing once again StartBattle to resumes since this also changes to an uncontrolled flow.

The main turn doesn't get advanced like normal however. For the most part, most of the procedure is the usual, but with some key differences due to `firststrike` being true still:

- [AdvanceTurnEntity](AdvanceTurnEntity.md) won't increment `turnssincealive` and `turnssincedeath`
- [AdvanceTurnEntity](AdvanceTurnEntity.md) won't advance the `cantmove` or `moreturnnextturn` if the entity's `hp` is above 0. Effectively, their actor turn availability is frozen
- [AdvanceMainTurn](Action%20coroutines/AdvanceMainTurn.md) won't process the `MiracleMatter` [medal](../../Enums%20and%20IDs/Medal.md) effects

The main turn however still gets advanced normally other than that. It means things like [conditions](../Actors%20states/Conditions.md), [delproj](../Actors%20states/Delayed%20projectile.md) process as normal, but the turn counters don't advance like normal.

From there, AdvanceMainTurn eventually switches back to a controlled flow, but that Update won't lead to `action` being set to true this time. This finally gives a chance for StartBattle to resume as it's still been waiting.

## Cleaning up
From there, a main turn was advanced, but StartBattle would like to proceed as if it didn't happen. This leads to the following final steps to cleanup:

- A [CheckDead](Action%20coroutines/CheckDead.md) happens just in case
- All enemy party members gets their `cantmove` and `tired` reset to 0 which resets their exhaustion and gives them 1 actor turn on this main turn. NOTE: This is incorrect and it should been set to -[moves](../Actors%20states/Enemy%20features.md#moves) + 1. The overall effect of this error is enemies with a `moves` of 2 or above looses all, but one actor turn on the battle's first main turn
- `currentturn` is reset to -1 (the previous Update potentially set the value, but it's undesired here as StartBattle isn't done yet)
- `action` is set to false (it already was false so this makes doubly sure the battle is in a controlled flow)
- `enemy` is set to false (same reason)
- `firststrike` is set to false (ends the `firststrike`)
- SetLastTurns is called which resets `lastturns` to a new aray with the length being the amount of free players - 1 and all elements being -1 (this resets the player selection cycle)

On top of this, later on, StartBattle ends by setting `turns` to 0. It effectively means that the main turn that was just advanced doesn't count towards the global main turn counter. It can be seen as if it was "main turn number -1".

From there, the battle continues as normal.
