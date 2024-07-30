# EventDialogue
This is a coroutine that will process what is known as an EventDialogue. An EventDialogue is a specialised version of an [event](../../Enums%20and%20IDs/Events.md), but specifically made for being handled during a battle. 

Starting this coroutine is all that's needed to have any event start and they may be started in a variety of ways mainly through [eventondeath](../Actors%20states/Enemy%20features.md#eventondeath), [eventonfall](../Actors%20states/Enemy%20features.md#eventonfall) (which is practically UNUSED) and [CheckEvent](Update%20flows/Controlled%20flow.md#checkevent). The game can know if an EventDialogue is in progress by checking the `inevent` field.

```cs
private IEnumerator EventDialogue(int id)
```

## Parameters

- `id`: The id of the EventDialogue to start

## Procedure

- `inevent` is set to true
- `caninputcooldown` and `blockcooldown` are reset to 0.0 which resets the [GetBlock](GetBlock.md) state
- `choicevine`'s position is set to offscreen (999.0 in y) if it exists
- [UpdateAnim](../Visual%20rendering/UpdateAnim.md) is called
- [UpdateText](../Visual%20rendering/UpdateText.md) is called
- [RefreshEnemyHP](../Visual%20rendering/RefreshEnemyHP.md) is called
- The EventDialogue whose id is the sent id value has its logic happen (see the section below for more information on them)
- [ReorganizeEnemies](../../Actors%20states/Enemy%20party%20members/ReorganizeEnemies.md) is called with order
- `inevent` is set to false
- UpdateConditionIcons is called which, if we aren't in the [enemy](Main%20turn%20life%20cycle.md#enemy-phase), calls UpdateConditionBubbles on all battleentity (all `playerdata` with right to false and all `enemydata` with `hp` above 0 with right to true)
- [UpdateText](../Visual%20rendering/UpdateText.md)

## EventDialogue tables
Here are all the EventDialogues defined along with links to their detail:

|ID|Description|
|-:|-----------|
|[0](Combat%20tutorials.md#eventdialogue-0)|Basic combat and action commands tutorial|
|[1](Combat%20tutorials.md#eventdialogue-1)|Part of the combat tutorial about turn flow|
|[2](Combat%20tutorials.md#eventdialogue-2)|Skills tutorial|
|[3](Combat%20tutorials.md#eventdialogue-3)|Turn Relay tutorial|
|[4](EventDialogues/Spuder.md#eventdialogue-4)|`Spuder` first encounter event|
|[5](EventDialogues/Spuder.md#eventdialogue-5)|`Spuder` second encounter event|
|[6](EventDialogues/Spuder.md#eventdialogue-6)|`MothWeb` death event|
|[7](EventDialogues/Spuder.md#eventdialogue-7)|`Spuder` third encounter event when they are past an `hp` treshold|
|[8](EventDialogues/Spuder.md#eventdialogue-8)|`Spuder` third encounter event when starting the battle with their `moves` set to 2|
|[9](EventDialogues/Acolyte.md)|`Acolyte` and `AcolyteVine` death event|
|[10](EventDialogues/VenusBoss.md)|`VenusBos` death event|
|[11](EventDialogues/Scarlet.md)|`Scarlet` death event|
|[12](../Enemy%20actions/Enemies/Kali.md)|`Kali` death event|
|[13](EventDialogues/SandWyrm%20and%20SandWyrmTail.md#eventdialogue-13)|`SandWyrmTail1` death event|
|[14](EventDialogues/SandWyrm%20and%20SandWyrmTail.md#eventdialogue-14)|`SandWyrm` death event|
|[15](EventDialogues/Cenn%20and%20Pisci.md)|`Cenn` and `Pisci` first encounter fleeing event (also happens on their death)|
|[16](EventDialogues/UltimaxTank.md)|`UltimaxTank` death event|
|[17](EventDialogues/WaspKing.md)|`WaspKing` death event|
|[18](EventDialogues/EverlastingKing.md)|`EverlastingKing` death event|
|[19](EventDialogues/Pitcher.md)|`Pitcher` event where either their death or [early spit out](../Actors%20states/BattleCondition/Eaten.md#spitout) are processed|
