# Eaten
A special stopping condition specific to the `Pitcher` [enemy](../../../Enums%20and%20IDs/Enemies.md) that allows through unorthodox methods to treat a player party member as if they were dead alongside having their `eatenby` not null.

## [IsStopped](../IsStopped.md)
This condition is considered a stop condition and will always make this method returns true (unless skipimmobile is false while the actor'a [actimmobile](../Enemy%20features.md#actimmobile) is true). This include the lite version used in the [enemy phase](../../Battle%20flow/Main%20turn%20life%20cycle.md#enemies-phase). Being stopped makes the actor unable to act regardless of their `cantmove` as well as a bunch of feature they no longer get access to.

## [GetFreePlayerAmmount](../Player%20party%20members/GetFreePlayerAmmount.md)
This condition makes a player party member not count as free. This affects many logic such as knowing if a player can act.

## [Relay](../../Battle%20flow/Action%20coroutines/Relay.md)
This condition will not be transfered to the target of the relay if the relayer had a `RelayTransfer` [medal](../../../Enums%20and%20IDs/Medal.md). It should be noted that it is not possible to relay from a player party member with this condition under normal gameplay so this clause effectively does nothing.

## [AdvanceTurnEntity](../../Battle%20flow/AdvanceTurnEntity.md)
When the condition is processed (when `hp` is above 0):

- A yield of 0.75 seconds is set to happen after the method is done
- If `eatenby` exists backed by an enemy party member, the player party member's `hp` is above 0 and there is at least one enemy party member:
    - [DoDamage](../../Damage%20pipeline/DoDamage.md) is called to the player party member with no attacker with a `NoExceptions` property, a `NoCounter` overrides without block. The damageammount is the player party member's `maxhp` / 10.0 + 1 ceiled and then clamped from 1 to 99
    - [ShowDamageCounter](../../Visual%20rendering/ShowDamageCounter.md) is called with type 0 with the damage amount as the amount with the start being the `eatenby` enemy party member's `cursoroffset` and the end being (0.0, 2.0, 0.0)
    - [Heal](../Heal.md) is called on the `eatenby` enemy party member for the same damage amount dealt to the player party member
- If the player party member's `hp` is 0 or below while there are at least 1 player party member with an `hp` above 0 while not having an `eatenby`:
    - `eatenkill` is set to true (this is used in [AdvanceMainTurn](../../Battle%20flow/Action%20coroutines/AdvanceMainTurn.md) to handle this specific case)
    - [EventDialogue](../../Battle%20flow/EventDialogue.md) 19 is started and stored in `checkingdead` (this will eventually call SpitOut, check the section below for more detail)

The turn counter of the condition is never decremented because it is expected to be removed manually.

## [AdvanceMainTurn](../../Battle%20flow/Action%20coroutines/AdvanceMainTurn.md)
Right after the return of [AdvanceTurnEntity](../../Battle%20flow/AdvanceTurnEntity.md), this is where the `eatenkill` handling logic happen if it is true (meaning they just died as a result of the HP drain):

- `eatenkill` is set to false (this is meant as a one shot flag field)
- All frames are yielded while `spitout` is in progress
- `checkingdead` is set to a new [CheckDead](../../Battle%20flow/Action%20coroutines/CheckDead.md) coroutine starting
- All frames are yielded while `checkingdead` is in progress

This part is necessary because the player party member may have not died properly and in order to address this, a CheckDead is needed, but it is only safe to do so after `spitout` is done.

After the player party had all their actor turns advanced, this condition will prevent a player party member from receiving the effects of the `FavoriteOne` [medal](../../../Enums%20and%20IDs/Medal.md) when another party member with the medal gets attacked.

## [ClearStatus](../Conditions%20methods/ClearStatus.md)
This condition is excluded from removal meaning it will remain even after calling this method.

## How Eat and SpitOut works
This condition has very special logic associated with it because it involves very unorthodox methods to essentially "fake" the death of a player party member. This section will clarify these methods.

This condition is specifically made to be inflicted via a method called Eat and removed with a coroutine called SpitOut (which also performs a lot of other visual effects with a strong assumption that the eater is a `Pitcher` [enemy](../../../Enums%20and%20IDs/Enemies.md)).

### Eat
Inflicts this condition to `playerdata[playerid]` for `turns` turns and set their `eatenby` to `enemydata[enemyid].battleentity` and the `ate` of the enemy to `playerdata[playerid].battleentity`. This is meant to be called in the enemy party member's [DoAction](../../Battle%20flow/Action%20coroutines/DoAction.md)'s action logic.

```cs
private void Eat(int enemyid, int playerid, int turns)
```

NOTE: In practice, this only works if the enemy party member is a `Pitcher` [enemy](../../../Enums%20and%20IDs/Enemies.md) because this eating system wasn't made to handle any other enemies.

For simplicity, `playerdata[playerid]` will be refered to as the eaten and `enemydata[enemyid]` will be refered as the eater.

The method sets the following fields:

|Field|Value|
|-----|-----|
|eaten's `eatenby`|eater's battleentity|
|eater's `ate`|eaten's battleentity|
|eaten's `moreturnnextturn`|0|
|eaten's `charge`|0|
|eaten's battleentity.`shieldenabled`|false|

And it also does the following:

- The eaten's battleentity.`sprite` is disabled
- The eaten's battleentity.`shadow` is disabled
- [ClearStatus](../Conditions%20methods/ClearStatus.md) is called on the eaten
- The `Eaten` condition is inflicted to the eaten for `turns` amount of turns via [SetCondition](../Conditions%20methods/SetCondition.md). It should be noted that the turn counter doesn't mean anything because it will never get decremented.

The overall effect is that the eaten will appear to no longer be in the battle because their sprite and shadows are disabled, but they are physically still present. In fact, they are still considered an active party member and the Eaten condition alone can't change this (the furthest it goes on that front is making [IsStopped](../IsStopped.md) return true, but this only prevents acting, it doesn't treat them as if they were dead). 

For more information on how this ends up treating the player party member as dead, see the `eatenby` section below. The `ate` field is less important and it is mainly used for the `Pitcher` [enemy](../../../Enums%20and%20IDs/Enemies.md)'s own logic.

### SpitOut
Performs the visual animations of spitting out the actor the `eater` ate (obtained through its `ate` field) and remove the eaten's Eaten condition and set their `eatenby` to null.

```cs
private IEnumerator SpitOut(EntityControl eater)
```

The details of this coroutine won't be detailed for the sake of brevety because most of it involves very verbose animations logic, but it essentially undo what Eat did and if the eater died as a result, [Death](../../../Entities/EntityControl/Notable%20methods/Death.md) is called on it.

What is important to mention is the circumstances in which this coroutine is called because it's done at very specific points:

- [EventDialogue](../../Battle%20flow/EventDialogue.md) 19: This is explained in the [AdvanceTurnEntity](../../Battle%20flow/AdvanceTurnEntity.md) section above, but this event dialogue is responsible for spitting out the eater in the case the eaten just died as a result of the HP drain attack
- [DoAction](../../Battle%20flow/Action%20coroutines/DoAction.md): This is where the `Pitcher` [enemy](../../../Enums%20and%20IDs/Enemies.md) can decide to spit out the player party member on its own (under normal gameplay, it's after 3 turns, counted by the `Pitcher`'s action logic).

### `eatenby` influences
The real way the system treats a player party member that has the Eaten condition as if they were dead is through their `eatenby` field because while it is mainly used for [AdvanceTurnEntity](../../Battle%20flow/AdvanceTurnEntity.md) to perform the HP drain, it is used in many places in the battle system.

It should always be null except when Eat is called until SpitOut is processed. When it is not null, it means the actor has been eaten and the system will explicitly treat them as if they were completely dead in some ways (it is usually treated similarly then as if their `hp` reached 0). Due to how much influence this field has, it's explicitly set to null on [StartBattle](../../StartBattle.md) to make doubly sure it's always null until Eat is called.

This implies that if the only player party member left is one that's eaten, the system will inevitably switch to a [terminal flow](../../Battle%20flow/Update%20flows/Terminal%20flow.md) because it will be treated as a dead party.

Unfortunately, this has to be done on a case by case basis: there is no universal way to treat a player party member as dead because the battle system was never designed to entirely disable (not just stop) a player party member during the battle.

The following details the places affected by having an `eatenby` of not null.

#### [DoItemEffect](../../../TextAsset%20Data/Items%20data.md#doitemeffect)
Being eaten prevents the `ReviveAll` and `CureParty` effects from healing or reviving the eaten member.

#### GetAlivePlayerAmmount
Beaing eaten counts as not being alive. This method is used all over the battle system in multiple places.

#### UpdateConditionBubbles
Not only will being eaten prevent the normal icons from being rendered, but it also specifically doesn't treat them as being dead for the purpose of rendering the icon involved with the `MiracleMatter` [medal](../../../Enums%20and%20IDs/Medal.md). This effectively prevents any icons to be rendered under any circumstances even if they would have fufilled the conditions required to render them.

#### [AdvanceMainTurn](../../Battle%20flow/Action%20coroutines/AdvanceMainTurn.md)
Being eaten prevents a [delproj](../Delayed%20projectile.md) from damaging the player party member. The delproj will still land like normal, but it will not do damages the same way a player party member with an `hp` at 0 would.

#### [GetChoiceInput](../../Player%20UI/GetChoiceInput.md) (handling the [skills list type](../../Player%20UI/ItemList%20confirmation%20handling/Skills%20list%20type.md) confirmation)
Being eaten prevents [DoAction](../../Battle%20flow/Action%20coroutines/DoAction.md) to be called on the player party member. Instead, the `Cancel` sound is played on sourceid 10.

Also, it will deny the confirmation of a player party member when selecting who to relay to when handling the `Relay` choice for a `SelectPlayer` action.

#### CheckItemUse
Being eaten always makes this return false and prevents the [UseItem](../../Battle%20flow/Action%20coroutines/UseItem.md) call. This mainly impacts the handling of the `Item` [Action](../../Player%20UI/Actions.md) by [GetChoiceInput](../../Player%20UI/GetChoiceInput.md) when [currentchoice](../../Player%20UI/Actions.md) is `SelectEnemy` or `SelectPlayer`.

Effectively, it's not possible to use an item from or to the eaten player party member.

#### [UseItem](../../Battle%20flow/Action%20coroutines/UseItem.md)
Being eaten prevents the following effects from processing on the actor:

- `HPRecoverAll`
- `ReviveAll`
- `CureParty`
- `CurePoisonAll`
- `GradualHPParty`

It covers healing effects targetting the whole player party.

#### [UseCharm](../../Battle%20flow/UseCharm.md)
Being eaten prevents the `DefenseUp` and `HealHP` charm types from processing on the actor. This covers the charm types that target the whole player party.

#### [GetRandomAvaliablePlayer](../Targetting/GetRandomAvaliablePlayer.md)
If `forceattack` is set to a player party member that is being eaten, the `forceattack` override that normally would have happened is ignored and instead, the method will select a random target as it normally would, but the eaten member is excluded from the possible selection.

If `forceattack` is -1, the method will select a random target as it normally would, but the eaten member is excluded from the possible selection.

NOTE: All this logic implies that if the only player party member remaining is being eaten, this will cause a softlock, but this can't happen because this is treated as a dead party leading to a certain [terminal flow](../../Battle%20flow/Update%20flows/Terminal%20flow.md).

#### [DoAction](../../Battle%20flow/Action%20coroutines/DoAction.md)
Being eaten prevents the following [skills](../../../Enums%20and%20IDs/Skills.md) from processing on the eaten player party member:

- `DefenseUpPlus`
- `SharingStash`
- `BubbleShield` (also `BubbleShieldLite`, but it's not possible to even select the eaten player party member in the first place)

It also prevents the `Abombhoney` [items](../../../Enums%20and%20IDs/Items.md) from processing on them.

For a `Pitcher` [enemy](../../../Enums%20and%20IDs/Enemies.md), his poison attack that targets the player party won't affect an eaten player party member.
