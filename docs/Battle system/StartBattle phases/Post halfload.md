# StartBattle - Post `halfload`
This is the last of 3 phases of [StartBattle](../StartBattle.md).

This section occurs after `halfload` is set to true and the coroutine yielded making the new value visible to the rest of the game.

## Last player and enemy setup
This part only contains miscellaneous initialisations logic mainly about the `playerdata` and `enemydata`: 

- All `enemydata` battleentity have their `rigid` gravity enabled
- The `dimmer` gets childed to the `battlemap` with a clear color and then disabled (this reveals the scene as it was hidden earlier)
- If instance.`firstbattleaction` is false (this is the first StartBattle call since the save was loaded)
    - A [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md) call starts with the first `playerdata` battleentity with action -555. -555 is a special value that will bypass most meaningful logic of the coroutine. In other words, it can be viewed as doing a blank call: nothing of note will happen and no actual action will be processed. The reason this is done has to do with a performance issue with DoAction where its IL code is large enough to stutter the game when compiling it with the JIT and this provokes this stutters early. Because this is done right before the battle out transition, it's less perceivable than if it was on the first action, hence this odd logic.
    - All frames are yielded while `action` is true until it goes to false
    - 0.5 seconds are yielded
    - instance.`firstbattleaction` gets set to true
- A frame is yielded

## Battle out transition
This part only plays the battle out transition by calling PlayTransition with id 3 (the battle out transition), the data being the map.`battleleaftype` at 0.025 speed and the color being the same one used for the previous transition played earlier.

0.2 seconds are yielded after.

## `switchicon` initialisation
This part initialises the `switchicon` if it didn't exist as a new GameObject with name `switchicon` childed to the `GUICamera` with a local position of (0.0, 99.0, 0.0). 

It also causes 3 children to get added to it:

- A new UI object named `arrow0` with size Vector3.one * 0.4 using the sprite `guisprites[92]` (right turn black arrow) with a sortingOrder of 3 with angles (0.0, 0.0, 180.0). The local y/z positions is 0 and the x is -1.0 except if `longcancel` is true where it becomes -1.25
- Another instance of the above, but the name is `arrow1`, there is no angles and the x position is the positive version instead of the negative one
- A new GameObject named `button` with a ButtonSprite component with SetUp called on it using input 5 (Cancel), -1 as onlytype, no description, local position of Vector3.zero, iconsize of Vector3.on * 0.4, sortingOrder of 0 and the `switchicon` as the parent making it also the actual parent when its Start occurs

## State reset
This resets some important state fields:

- `currentturns` is set to -1 (no player selected)
- `turns` is set to 0
- `cancelupdate` is set to false
- `enemy` is set to false
- instance.`inbattle` is set to true
- if `sadv` is not negative, adv is set to it
- `sadv` is set to adv (which only does anything if it was 3)

## Enemy first strike
This part only happens if adv is 3:

- `enemy` is set to true (we temporarilly go into the [enemy phase](../Battle%20flow/Main%20turn%20life%20cycle.md#enemies-phase))
- `firststrike` is set to true
- [UpdateAnim](../Visual%20rendering/UpdateAnim.md) is called
- instance.`hudcooldown` is set to 10.0 which shows the main HP / TP HUD
- instance.`showmoney` is set to -1.0 which hides the berry count HUD
- `action` is set to true (enter an [uncontrolled flow](../Battle%20flow/Update%20flows/Uncontrolled%20flow.md))
- [RefreshEnemyHP](../Visual%20rendering/RefreshEnemyHP.md) is called
- 0.15 seconds are yielded
- A [DoAction](../Battle%20flow/Action%20coroutines/DoAction.md) call is started on `enemydata[0]` with action 0
- All frames are yielded while `action` is true until it goes to false (waits that DoAction is done)
- `enemydata[0].tired` is set to 0
- `enemydata[0].cantmove` is set to 0
- `action` is set to true (goes back into an [uncontrolled flow](../Battle%20flow/Update%20flows/Uncontrolled%20flow.md))
- A [CheckDead](../Battle%20flow/Action%20coroutines/CheckDead.md) is started with the coroutine being stored in `checkingdead`
- All frames are yielded while `checkingdead` is in progress
- All `enemydata` have their `tired` and `cantmove` set to 0
- `currentturns` is set to -1
- `action` is set to false (resets to a [controlled flow](../Battle%20flow/Update%20flows/Controlled%20flow.md#controlled-flow))
- `firststrike` is set to false
- `enemy` is set to false (reset to be in the [player phase](../Battle%20flow/Main%20turn%20life%20cycle.md#player-phase))
- SetLastTurns is called which resets `lastturns` to a new array with the length being the amount of free players - 1 and all elements being -1

## Conditions special logic
This part manages the player and enemies conditions as they have special logic to them that must occur at the very end.

If `saveddata` is true (this is a retry), SetStartConditions is called which will do the following:

- Add all the corresponding `sdata.psc` to each `playerdata`'s [condition](../Actors%20states/Conditions.md) from the existing ones
- Add all the corresponding `sdata.esc` to each `enemydata`'s [condition](../Actors%20states/Conditions.md) from the existing ones
- Add all the corresponding `sdata.enemyweakness` to each `enemydata`'s [weakness](../Actors%20states/Enemy%20features.md#weakness) from the existing ones
- SetLastTurns is called which resets `lastturns` to a new aray with the length being the amount of free players - 1 and all elements being -1

After, all `playerdata`'s `condition`'s element that have a turn count of 0 or below are removed

From there, the `RandomStart` [medal](../../Enums%20and%20IDs/Medal.md) takes effect if it's equipped for each `playerdata` that has it. The effec involves a GetChance test that is performed with 16 weights with outcomes from 0 to 6 and what happens here depends on the result:

- 0 (3/16): [StatusEffect](../Actors%20states/Conditions%20methods/StatusEffect.md) is called on the `playerdata` with [AttackUp](../Actors%20states/BattleCondition/AttackUp.md) for 2 actor turns with effect
- 1 (3/16): [StatusEffect](../Actors%20states/Conditions%20methods/StatusEffect.md) is called on the `playerdata` with [DefenseUp](../Actors%20states/BattleCondition/DefenseUp.md) for 2 actor turns with effect
- 2 (2/16): 
    - The `StatUp` sound is played if it wasn't already 
    - [StatEffect](../Visual%20rendering/StatEffect.md) is called with the battleentity using type 4 (green up arrow)
    - The player party member's `charge` is incremented
- 3 (2/16): If the `playerdata`'s `poisonres` is 100 or above (it's immune to poison), this is treated as an outcome of 2 described above. If it's not:
    - The `Poison` sound is played if it wasn't already
    - The `PoisonEffect` particles are played on the battleentity position
    - [SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called with [Poison](../Actors%20states/BattleCondition/Poison.md) on the player party member for 2 actor turns unless the `Eternal Venom` [medal](../../Enums%20and%20IDs/Medal.md) is equipped on it where it's 99999 actor turns instead
- 4 (2/16): The `StatUp` sound is played if it wasn't already followed by a [StatEffect](../Visual%20rendering/StatEffect.md) coroutine being started with the battleentity using type 5 (yellow up arrow) followed by the `playerdata`'s `cantmove` being decremented (gives an additional actor turn)
- 5 (2/16): 
    - The `Heal3` sound is played if it wasn't already
    - The `MagicUp` particles being played on the battleentity position without sound
    - [SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called with [GradualHP](../Actors%20states/BattleCondition/GradualHP.md) on the player party member for 2 actor turns
- 6 (2/16): 
    - The `Heal3` sound is played if it wasn't already
    - The `MagicUp` particles are played on the battleentity position without sound
    - [SetCondition](../Actors%20states/Conditions%20methods/SetCondition.md) is called with [GradualTP](../Actors%20states/BattleCondition/GradualTP.md) on the player party member for 2 actor turns

Finally, UpdateConditionIcons is called which calls UpdateConditionBubbles on the battleentity (all `playerdata` with right to false and all `enemydata` with `hp` above 0 with right to true)

## Final setup
These are the final steps of StartBattle:

- `turns` is set to 0
- MainManager.SortLights is called
- If MainManagr.`music[0]` is played and its clip exists, `sdata.music` is set to the clip's name
- `saveddata` is set to true
