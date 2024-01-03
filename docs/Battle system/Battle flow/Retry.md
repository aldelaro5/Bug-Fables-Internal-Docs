# Retry
This method serves as a transition from a [GameOver](Terminal%20coroutines/GameOver.md) to a complete retry of the battle using [StartBattle](../StartBattle.md).

```cs
private void Retry(bool skipdata)
```

## Parameters

- `skipdata`: If true, ReloadInitialData won't be called. This is done if the caller already called it which is needed to adjust the starting stats to what they were on the first [StartBattle](../StartBattle.md) recorded from `sdata`

## Prcedure

- All `delprojs`'s `obj` are destroyed followed by the `delprojs` array set to null
- The `battlemap` is destroyed
- [SetDefaultCamera](../Visual%20rendering/SetDefaultCamera.md) with reset is called
- `chompyattacked` is set to false
- `aiattcked` is set to false
- `tskybox` is set to RenderSettings.skybox
- RenderSettings.skybox is set to null
- instance.`enemyencounter` is set to `sdata.encounter`
- If skipdata is false, ReloadInitialData is called which does the following (mostly loads from the [StartUpData](../../StartUpData.md)):
    - [ApplyBadges](../../ApplyBadges.md) is called
    - [ApplyStatBonus](../../ApplyStatBonus.md) is called
    - instance.`maxtp` is set to `sdata.maxtp`
    - instance.`tp` is set to `sdata.tp` clamped from 0 to instance.`maxtp`
    - For each player party member:
        - `atk` is set to the matching `sdata.atk`
        - `def` is set to the matching `sdata.def`
        - `hp` is set to the matching `sdata.hp` clamped from 0 to the player party member's `maxhp`
        - `lockitems` is set to the matching `sdata.partyitemuse`
    - instance.`items[0]` (standard items) is set to `sdata.items`
    - instance.`items[1]` (key items) is set to `sdata.keyitem`
- A [StartBattle](../StartBattle.md) call starts with the following:
    - enemyids: `sdata.enemies`
    - stageid: `sdata.stage`
    - adv: `sdata.adv`
    - music: `sdata.music`
    - calledfrom: `sdata.called`
    - canescape: `canflee` (from this BattleControl)
