# BattleControl
BattleControl is a component whose logic contain the entire battle system of the game. This makes it the most monolithic component of the game, but it is also relatively isolated because most of its members are private. The current battle is stored in instance.`battle` with the ability to query if we are in one by checking if it's not null or instance.`inbattle` being true. Some logic belongs in MainManager, but the vast majority belongs to BattleControl which means this will mostly be documenting BattleControl.

For details on BattleControl's fields, consult the [battle state](Battle%20state.md) documentation.

## Terminology
This system is the most complex in the game and as such, it is very easy to confuse terms for the purpose of documenting it. Due to this, there will be some standard terms with a consistent definition to reduce ambiguities as much as possible.

Here are all the terms that will be used throughout the battle system documentation:

- `player party member`: A `playerdata` element which is a [BattleData](Actors%20states/BattleData.md) corresponding to one of the player party members. It can be thought as someone part of the player's side of the battle. These elements are also used in the overworld, but the battle system documentation will focus on their roles during a battle
- `enemy party member`: An `enemydata` element which is a [BattleData](Actors%20states/BattleData.md) corresponding to one of the enemy party members. It can be thought as one of the enemies fought during the battle and is part of the enemy's side. These elements are only used during a battle
- `player party`: Refers to the the aggregate of all `playerdata` elements. This can be thought of as everyone from the player's side
- `enemy party`: Refers to the aggregate of all `enemydata` elements. This can be thought of as all the enemies fought in the battle
- `party`: Refers to either side of the battle (player or enemy)
- `actor`: Refers to any party members no matter which side they are on
- `action`: Refers to what an actor can do during the battle. In more concrete turn, anything that normally consumes an actor turn (usually through `cantmove`) counts as an action
- [player phase](Battle%20flow/Main%20turn%20life%20cycle.md#player-phase): Refers to the processing of all the player party's actions and all logic associated with it. More details in its documentation
- [enemy phase](Battle%20flow/Main%20turn%20life%20cycle.md#enemy-phase): Refers to the processing of all the enemy party's actions and all logic associated with it. More details in its documentation
- [turn end phase](Battle%20flow/Main%20turn%20life%20cycle.md#turn-end-phase): Refers to the special phase that happens after the player and enemy phases where the main turn is advanced and finished
- `phase`: Refers to any of the 3 phases
- [main turn](Battle%20flow/Main%20turn%20life%20cycle.md): Refers to the processing of all 3 phases as a unit which starts on the player phase and ends after the turn end phase. More details about a main turn's lifecycle in its documentation
- `actor turn`: Refers to an individual's actor's concept of turn which includes their actions and ends with its turn advancement including their `cantmove` change. The actor turn is consumed by [DoAction](Battle%20flow/Action%20coroutines/DoAction.md) for player party members (with additional processing in [EndPlayerTurn](Battle%20flow/EndPlayerTurn.md)) or [EndEnemyTurn](Battle%20flow/EndEnemyTurn.md) for enemy party members and it is advanced to the next one by [AdvanceTurnEntity](Battle%20flow/AdvanceTurnEntity.md)

## Execution flow
There is only one way to start a battle: starting a [StartBattle](StartBattle.md) coroutine. This will perform a wide variety of setup and fields initialisation to start the battle from a fresh state. This will not be recalled unless a [Retry](Battle%20flow/Retry.md) occur which is the opt-in ability to recreate the battle from the existing one, but preserving the [StartUpData](StartUpData.md). The caller is able to further configure the battle as needed either by setting battle.`haltbattleload` to true to make the coroutine stop early or by yielding on battle.`halfload` to wait the coroutine stops later on on the caller side and configure it that way.

After the battle is fully started (which may include enemy actions if they got the advantage), [Update](Battle%20flow/Update.md) takes over. From there, its flow can be in one of 3 states from now on:

- [Controlled flow](Battle%20flow/Update%20flows/Controlled%20flow.md) (the one the battle starts at after StartBattle is done)
- [Uncontrolled flow](Battle%20flow/Update%20flows/Uncontrolled%20flow.md) (delegates the control of the battle to another method or coroutine)
- [Terminal flow](Battle%20flow/Update%20flows/Terminal%20flow.md) (relegates control almost completely when the battle is about to end such that it can end or be retried properly)

The key point to understand how a battle flows is the first 2 flows are constantly toggled with each other. Update is the main dispatcher that runs by default, but it's only making sure the usual flow is respected. Most of the heavy lifting is done by a coroutine or a method taking control.

### Action coroutines
Whenever something complex needs to happen such as an attack, battle.`action` (or battle.`inevent`) is set to true which switches to an uncontrolled flow as the coroutine or method takes most control away from Update. Update has very reduced logic during this, notably handling [blocking](Battle%20flow/GetBlock.md) during the enemy phase. Eventually, the procedure gets done so battle.`action` (or `inevent`) gets set back to false switching to a controlled flow. A coroutine which sets battle.`action` to true is refered to as an action coroutine.

Here are all the action coroutines defined in the battle system:

- [Tattle](Battle%20flow/Action%20coroutines/Tattle.md)
- [SwitchParty](Battle%20flow/Action%20coroutines/SwitchParty.md)
- [SwitchPos](Battle%20flow/Action%20coroutines/SwitchPos.md)
- [Relay](Battle%20flow/Action%20coroutines/Relay.md)
- [UseItem](Battle%20flow/Action%20coroutines/UseItem.md)
- [DoAction](Battle%20flow/Action%20coroutines/DoAction.md)
- [CheckDead](Battle%20flow/Action%20coroutines/CheckDead.md)
- [Chompy](Battle%20flow/Action%20coroutines/Chompy.md)
- [AIAttack](Battle%20flow/Action%20coroutines/AIAttack.md)
- [AdvanceMainTurn](Battle%20flow/Action%20coroutines/AdvanceMainTurn.md)
- [TryFlee](Battle%20flow/Action%20coroutines/TryFlee.md)

[DoNothing](Player%20UI/ItemList%20confirmation%20handling/Battle%20strategy%20list%20type.md#2-do-nothing) is notably NOT an action coroutine because it is handled directly in place during player UI handling.

The interesting point these have in common is with the exception of doing nothing from a player party member, these represents all actions that can possibly be taken by an actor. Notably, [DoAction](Battle%20flow/Action%20coroutines/DoAction.md) covers every single attacks in the entire game from every actor. This is why BattleControl can be caracterised as a coroutine driven system: while it does have an Update, it only acts as a dispatcher for the coroutines mentioned above.

### Battle events
Separatly from this, but still features an uncontrolled flow is [EventDialogue](Battle%20flow/EventDialogue.md) which is a coroutine that processes some kind of events or cutscenes. The only difference with an action coroutine is instead of setting battle.`action` to true, it sets battle.`inevent` to true instead, but it has similar effects. These are triggered on specific conditions and behave very similarily to [events](../Enums%20and%20IDs/Events.md), but are specialised to handle the ongoing battle.

### Terminal coroutines
A terminal flow is reserved for cases where the battle will end for sure in some ways and it features battle.`cancelupdate` or instance.`inbattle` being set to false. It features almost no control from Update and instead, the terminal coroutine controls the battle completely. This is done both for retries, losses and wins.

Here are all the terminal coroutines:

- [GameOver](Battle%20flow/Terminal%20coroutines/GameOver.md)
- [ReturnToOverworld](Battle%20flow/Terminal%20coroutines/ReturnToOverworld.md)
- [DeadParty](Battle%20flow/Terminal%20coroutines/DeadParty.md) (this one is a wrapper to GameOver and ReturnToOverworld)
- [AddExperience](Battle%20flow/Terminal%20coroutines/AddExperience.md)

They act similarly to action coroutines, but with even more control which is needed to properly end the battle or retry it.

Additionally, there are also wrapper methods available that will call some of the above and also change to a terminal flow:

- [EndBattleWon](Battle%20flow/Terminal%20wrappers/EndBattleWon.md) (wraps AddExperience)
- [ExitBattle](Battle%20flow/Terminal%20wrappers/ExitBattle.md) (wraps ReturnToOverWorld, part of AddExperience)

## Main turn flow
As this is a turn based battle system, it has a concept of turns. A main turn is composed of 3 phases executed in order:

- [Player phase](Battle%20flow/Main%20turn%20life%20cycle.md#player-phase)
- [Enemy phase](Battle%20flow/Main%20turn%20life%20cycle.md#enemies-phase)
- [Turn end phase](Battle%20flow/Main%20turn%20life%20cycle.md#turn-end-phase)

The player phase is the most involved mainly handling UI naviguations on top of player actions. It is only active when battle,`enemy` is false.

The enemy phase mostly involves each enemies doing their action when `enemy` is true and [AdvanceMainTurn](Battle%20flow/Action%20coroutines/AdvanceMainTurn.md) being the whole procedure that advances the main turn and allows a fresh one to start again.

Each actor also have their own concept of turns called actor turn. An actor turn is composed of their action and other advancement logic such as the `cantmove` advance. `cantmove` is a multi purpose actor turn counter that serves as the way for the game to track that an actor can act, needs to wait before being able to, or has multiple actions available. The actor turn advancement notably includes logic associated with the [conditions](Actors%20states/Conditions.md) currently inflicted on the actor among other details.

When all the actor turns available are consumed, the actor needs to wait their actor turn gets advanced which is done by [AdvanceTurnEntity](Battle%20flow/AdvanceTurnEntity.md) and it is only called as part of [AdvanceMainTurn](Battle%20flow/Action%20coroutines/AdvanceMainTurn.md) during the turn end phase.

### hitactions
There is a notable exception to this flow: an enemy is able to do what's known as a [hitaction](Actors%20states/Enemy%20features.md#hitaction). A hitaction occurs when the field of the same name on the enemy party member is true which causes the next Update cycle to process a temporary DoAction call on the enemy while placing `enemy` to true. Check the documentation for more information on this feature.

## Damage pipeline
During the course of the battle, damages might be processed for various reasons such as attacks or using an item. These damages are processed by the damage pipeline which has a single entry point: [DoDamage](Damage%20pipeline/DoDamage.md). This method takes in a target, an optional attacker belonging to the opposte party of the target (it is possible to have damages comes from no actor, but friendly fire attacks aren't possible) and a base amount to inflict. This amount may change for various reasons as part of the damage calculation logic which is held in [CalculateBaseDamage](Damage%20pipeline/CalculateBaseDamage.md). 

The damage might have more effects processed after its calculation for example, by applying some [medals](../Enums%20and%20IDs/Medal.md) that act on it. It is greatly influenced by the property parameter and the enemy party members's [weakness](Actors%20states/Enemy%20features.md#weakness) field, a list of [AttackProperty](Damage%20pipeline/AttackProperty.md) that applies to this enemy party member.

It should be noted however that the caller of DoDamage is free to perform any logic that goes beyond the scope of the damage pipeline which happens very frequently in actions's logic.

## Actor state
The player party's stats are in instance.`playerdata` (which the whole game uses elsewhere in various places) while the enemy party's are specifically inside BattleControl and is called `enemydata`.  The `playerdata` array has complex addressing methods, more info can be found at the [`playerdata` addressing documentation](playerdata%20addressing.md)

They are both the same type: a struct called [BattleData](Actors%20states/BattleData.md) which means some fields have different meanings depending if it's a player party member or an enemy party member and some fields are only relevant for one of the 2.

More details on the fields can be consulted in the [BattleData](Actors%20states/BattleData.md) documentation.

## Other Unity events
Besides Update, this component has a [LateUpdate](Visual%20rendering/LateUpdate.md) and a [FixedUpdate](Visual%20rendering/FixedUpdate.md), but they only update UI informations visually and are therefore much more minor in terms of importance.
