# UseItem
This action coroutine allows the player party member to use an [item](../../../Enums%20and%20IDs/Items.md). It receives its id in a parameter which should corresponds to the one that was selected during [GetChoiceInput](../../Player%20UI/GetChoiceInput.md) after passing the CheckItemUse check.

## Setup

- `action` is set to true switching to an [uncontrolled flow](../Update%20flows/Uncontrolled%20flow.md)
- [UpdateText](../../Visual%20rendering/UpdateText.md) is called
- The `ItemHold` sounds is played
- `lastitemuser` is set to the `trueid` of the `currentturn` player party member
- A new temporary item object is created with a SpriteRenderer using the corresponding item's sprite from [items data](../../../TextAsset%20Data/Items%20data.md) using the standard spritemat with a renderQueue of 50000 on layer 14 (`Sprite`). The object is positioned at the `currentturn` player party member's battleentity's position + its `cursoroffset` - (0.0, 0.0, 0.1)
- The `currentturn` player party member's battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 4 (`ItemGet`)

## `BanditLeader` specific logic
This coroutine has specoial logic if a `BanditLeader` [enemy](../../../Enums%20and%20IDs/Enemies.md) is present in the enemy party. When that happens, this coroutine will not apply any item effects and it will instead end abruptly with a yield break after some animations and cleanup logic. This logic is what this section is about. It doesn't apply if no `BanditLeader` is present.

If it does apply, the enemy party member index of the first `BanditLeader` is saved locally and used for this section.

- 0.25 seconds are yielded
- The `BanditLeader`'s battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is saved locally for restore later
- [HealConditions](../../Actors%20states/Conditions%20methods/HealConditions.md) is called on the `BanditLeader`
- [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) is called
- The `BanditLeader`'s battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 103
- The `Slash2` sound is played
- 0.5 seconds are yielded
- Gleam is called on the `BanditLeader`'s battleentity using the offset (-0.4, 2.35, -0.1) which plays the `Gleam` particles and sound
- 0.5 seconds are yielded
- The `Slash` sound is played
- The `BanditLeader`'s battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 107
- MainManager.FadeIn is called with a speed of 1.0 (which makes the screen go to black instantly)
- 0.25 seconds are yielded
- ItemDrop is called on the temporary item object which will do the following:
    - The object gets rooted to the scene
    - A RigidBody component is added with a velocity of RandomItemBounce(5.0, 12.5)
    - The object is destroyed in 1.0 second
- The `currentturn`'s battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 11 (`Hurt`)
- MainManager.FadeOut is called with a speed of 1.0 (which makes the screen go back to what it was before the fade in instantly)
- The `ItemStolen` sound is played
- 0.65 seconds are yielded
- The `BanditLeader`'s battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is restrored to the saved value
- The first occurence of the item corresponding to the sent id is removed from instance.`items[0]` (standard items)
- [CancelList](../../Player%20UI/CancelList.md) is called
- [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) is called
- [UpdateText](../../Visual%20rendering/UpdateText.md) is called
- [EndPlayerTurn](../EndPlayerTurn.md) is called
- `action` is set to false switching to a [controlled flow](../Update%20flows/Controlled%20flow.md)
- Yield break which ends this coroutine

## Pre item effects
This section happens if the `BanditLeader` specific logic didn't apply.

- 1.3 seconds are yielded
- The temporary item object is destroyed
- `deadmembers` is set to GetDeadParty which is all player party member indexes whose `hp` is 0 or below in ascending order
- The `currentturn` player party member's battleentity.[animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 13 (`BattleIdle`)
- [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) is called

## Item effects
This section depends on the `itemarea`.

### `SingleAlly` or `AllParty`
This first starts by having all frames yielded while PartyIsDying which means that at least one player party member's `hp` is 0 or below while its battleentity.`deathroutine` is still in progress.

From there, the ItemUse elements of the item are processed by first calling [DoItemEffect](../../../TextAsset%20Data/Items%20data.md#doitemeffect) with the element's information and `option` as the characterid.

After, there's additional logic depending on the ItemUsage.

#### `HPRecover`, `HPRecoverFull`

- HealParticle is called on the `option` player party member with a size of Vector3.one and an offset of Vector3.up
- [ShowDamageCounter](../../Visual%20rendering/ShowDamageCounter.md) is called with type 1 (HP) with the ammount being the return of DoItemEffect starting at the `playerdata[option].battleentity`'s position + `playerdata[option].cursoroffset` and ending at Vector3.up
- If there's more than 1 effect, 0.5 seconds are yielded before processing more

#### `Revive`
The same than `HPRecover` or `HPRecoverFull`, but if the `option` player party member's battleentity is `dead`, [RevivePlayer](../../Actors%20states/Player%20party%20members/RevivePlayer.md) is called with its id for -1 hp and showcounter as false.

#### `TPRecover` or `TPRecoverFull`

- [ShowDamageCounter](../../Visual%20rendering/ShowDamageCounter.md) is called with type 2 (TP) with the ammount being the return of DoItemEffect starting at the `playerdata[option].battleentity`'s position + `playerdata[option].cursoroffset` and ending at Vector3.up
- If there's more than 1 effect, 0.5 seconds are yielded before processing more

#### `HPRecoverAll`
This effect applies to each player party member that isn't `eatenby` and has an `hp` above 0:

- HealParticle is called on the player party member with a size of Vector3.one and an offset of Vector3.up
- [ShowDamageCounter](../../Visual%20rendering/ShowDamageCounter.md) is called with type 1 (HP) with the ammount being the return of DoItemEffect starting at the `playerdata[option].battleentity`'s position + `playerdata[option].cursoroffset` and ending at Vector3.up

If at least one player party member was affected and there's more than 1 effect, 0.5 seconds are yielded before processing more.

#### `ReviveAll`
The same than `HPRecoverAll`, but it also affects player party members with an `hp` of 0 or below and if its battleentity is `dead`, [RevivePlayer](../../Actors%20states/Player%20party%20members/RevivePlayer.md) is called with its id for -1 hp and showcounter as false.

#### `HPorDamage`

- An RNG check is performed with 2 possible outcomes at 66% and 34% respectively:
    - 66%: [Heal](../../Actors%20states/Heal.md) is called on the `option` player party member with the ammount being the returned value of DoItemEffect
    - 34%: [DoDamage](../../Damage%20pipeline/DoDamage.md) is called without an attacker to the `option` player party member with the damageammount being the returned value of DoItemEffect with a `NoExceptions` property and without block
- The `hp` of the `option` player party member is clamped from 1 to its `maxhp`
- If there's more than 1 effect, 0.5 seconds are yielded before processing more

#### `Battle`
An action may be set to trigger in the post item effect section depending on the item id:

- `VitalitySeed`: 14
- `GenerousSeed`: 15
- `ShellOil`: 33

#### `AddPoison`
The `PoisonEffect` particle are played without sound at the `option` player party member's battleentity's position.

#### `AddNumb`
The `ElecFast` particle are played without sound at the `option` player party member's battleentity's position + Vector3.up all scaled by (1.5, 1.5, 1.5).

#### `AddFreeze`

- The `mothicenormal` particle are played without sound at the `option` player party member's battleentity's position + Vector3.up all scaled by (1.5, 1.5, 1.5)
- If the `option` player party member has a `Freeze` [condition](../../Actors%20states/Conditions.md) and its battleentity.`icecube` is null or inactive, [Freeze](../../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#freeze) is called on the battleentity

#### `AddSleep`
DeathSmoke particles are played at the `option` player party member's battleentity's position

#### `TurnNextTurn`

- [StatEffect](../../Visual%20rendering/StatEffect.md) is called with the `option` player party member's battleentity with type 5 (orange up arrow)
- If there's more than 1 effect, 0.5 seconds are yielded before processing more

#### `CureNumb`

- The `option` player party member's `isnumb` is set to false
- The `CurePoison` effects applies

#### `CureFreeze`

- [BreakIce](../../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#breakice) is called on the `option` player party member's battleentity
- The `CurePoison` effects applies

#### `CureSleep`

- The `option` player party member's `isasleep` is set to false
- The `CurePoison` effects applies

#### `CureParty`

- For each player party members that isn't `eatenby` and has an `hp` above 0:
    - `isnumb` is set to false
    - [BreakIce](../../../Entities/EntityControl/Notable%20methods/Freeze%20handling.md#breakice) is called on the battleentity
    - `isasleep` is set to false
- The `CurePoisonAll` effects applies

#### `CurePoison`, `GradualHP`, `GradualTP`, `CureFire` and `CureAll`

- The `MagicUp` particle are played without sound at the `option` player party member's battleentity's position
- If there's more than 1 effect, 0.5 seconds are yielded before processing more

#### `CurePoisonAll`

- The `MagicUp` particle are played without sound at each of the player party members's battleentity's position who isn't `eatenby` and whose `hp` is above 0
- If there's more than 1 effect, 0.5 seconds are yielded before processing more

#### `ChargeUp`

- The `StatUp` sound is played
- [StatEffect](../../Visual%20rendering/StatEffect.md) is called with the `option` player party member's battleentity with type 4 (green up arrow)
- If there's more than 1 effect, 0.5 seconds are yielded before processing more

#### `GradualHPParty`
The `MagicUp` particle are played without sound at each of the player party members's battleentity's position who isn't `eatenby` and whose `hp` is above 0

#### `HPto1`

- [ShowDamageCounter](../../Visual%20rendering/ShowDamageCounter.md) is called with type 0 (Damage) with the ammount being the `option` player party member's `maxhp` - 1 starting at its battleentity's position and ending at (-1, 2.0, 0.0)
- [DoDamageAnim](../../Visual%20rendering/DoDamageAnim.md) is called with the `option` player party member with a damage of its `maxhp` - 1 without block
- If there's more than 1 effect, 0.5 seconds are yielded before processing more

#### `AtkUpStat`

- [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) is called with the `option` player party member using the `AttackUp` condition with the amount of turns being the returned value from DoItemEffect with effect being true
- If there's more than 1 effect, 0.5 seconds are yielded before processing more

#### `DefUpStat`

- [StatusEffect](../../Actors%20states/Conditions%20methods/StatusEffect.md) is called with the `option` player party member using the `DefenseUp` condition with the amount of turns being the returned value from DoItemEffect with effect being true
- If there's more than 1 effect, 0.5 seconds are yielded before processing more

#### `Sturdy`

- [flagvar](../../../Flags%20arrays/flagvar.md) 0 is set to the returned value from DoItemEffect
- The action id 33 is set to trigger in the post item effects section

### `SingleEnemy`, `AllEnemies` or `All`
An action will may be set to trigger in the post item effects section depending on the item id:

- `LonglegSummoner`: The -2 action id will be set to trigger
- If the item id is in the following list, the 9 action id will be set to trigger:
    - `HardSeed`
	- `SpicyBomb`
	- `PoisonBomb`
	- `ClearBomb`
	- `NumbDart`
	- `Ice`
	- `FrostBomb`
	- `NumbBomb`
	- `SleepBomb`
	- `Abombhoney`
	- `PoisonDart`
	- `CherryBomb`
	- `BurlyBomb`
	- `FlameRock`

## Post item effects

- The first occurence of the item corresponding to the sent id is removed from instance.`items[0]` (standard items)
- [CancelList](../../Player%20UI/CancelList.md) is called
- [UpdateAnim](../../Visual%20rendering/UpdateAnim.md) is called
- [UpdateText](../../Visual%20rendering/UpdateText.md) is called
- If an action was set to trigger after the item use:
    - `currentaction` is set to `ItemList`
    - [DoAction](DoAction.md) is called with the `currentturn` party member using the action id set to trigger earlier
- Otherwise:
    - 0.5 seconds are yielded
    - [EndPlayerTurn](../EndPlayerTurn.md) is called
    - `action` is set to false switching to a [controlled flow](../Update%20flows/Controlled%20flow.md)
