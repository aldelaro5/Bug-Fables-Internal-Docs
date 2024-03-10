# Heal
This method increases an actor's `hp` alongside some visual effects.

```cs
private void Heal(ref MainManager.BattleData entity, int? ammount, bool nosound)
```

## Parameters

- `entity`: The actor to heal
- `amount`: The amount to heal. If null, it heals by 999
- `nosound`: If true, the `Heal` sound won't be played. There is an overload without this parameter that sends false

## Procedure

- MainManager.HealParticle is called on the battleentity with a size of Vector3.one and an offset of Vector3.up * the battleentity.`height`
- If `nosound` is false and the `Heal` sound isn't playing or its more than halfway done player, it is played
- The actor's `hp` is increased by amount (or 999 if it's null) clamped from 0 to the actor's `maxhp`
- [ShowDamageCounter](../Visual%20rendering/ShowDamageCounter.md) is called with type 1 with the damage amount as the amount healed earlier with the start being the battleentity position + (2.0, 2.0, 2.0) and the end being (5.0, 5.0, 5.0)

