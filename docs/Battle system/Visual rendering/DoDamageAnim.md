# DoDamageAnim
This coroutine is only called as part of [DoDamage](../Damage%20pipeline/DoDamage.md) when applicable and as part of [UseItem](../Battle%20flow/Action%20coroutines/UseItem.md) when processing a `HPto1` item use.

It only performs [animstate](../../Entities/EntityControl/Animations/animstate.md) and other visual update on an actor to indicate that they have sustained damage.

```cs
private IEnumerator DoDamageAnim(MainManager.BattleData entity, int damage, bool block, bool nosound, bool fail, bool noanim, bool fakeanim)
```

There is a coroutine overload available that starts this one where `nosound`, `fail`, `noanim` and `fakeanim` are all false with the only logic difference being there's a frame yield after the coroutine started.

## Parameters

- `entity`: The actor sustaining the damage
- `damage`: The amount of damage the actor is sustaining
- `block`: If the actor is a player party member, whether the player sucessfully blocked the damage it is sustaining
- `nosound`: If true, no sound will be played
- `fail`: If `nosound` is false, this tells if the player failed the attack's action command which changes some logic regarding sounds being played. This is ignored if `nosound` is true
- `noanim`: If true, there won't be any [animstate](../../Entities/EntityControl/Animations/animstate.md) updates on the actor even if entity.battleentity.`overridenanim` is false
- `fakeanim`: If `noanim` is false, this changes the [animstate](../../Entities/EntityControl/Animations/animstate.md) value to set the actor's battleentity to 30 (`FakeHurt`) instead of 11 (`Hurt`) when damage is above 0 and block is false. This is ignored if `noanim` is true, `damage` isn't above 0 or `block` is true.

## Procedure

- If noanim and entity.battleentity.`overrideanim` are false, entity.battleentity.`overrideanim` is set to true
- If nosound is false:
    - If fail is true, the `AtkFail` sound is played
    - If block is true, the `Block` sound is played if it wasn't playing already with a time of 0.1 seconds or below
    - If damage is above 0 while block is false, the `Damage0` sound is played if it wasn't already with a time of 0.1 seconds or below. As for the pitch and volume:
        - If fail is true, the pitch is 0.7 and volume is 0.6
        - If fail is false, the pitch is 1.0 + combo * 0.125 and the volume is 1.0
    - If damage is above 0 while block is false, the `WoodHit` sound is played  with a volume of 0.8 if it wasn't playing already with a time of 0.1 seconds or below
- If damage is above 0
    - The `hitpart` is played at entity.battleentity's position + (0.0, entity.`cursoroffset`.y / 2.0 + entity.battleentity.`height`, 0.0)
    - If MainManager.`screenshake`'s magnitude is below 0.05, ShakeScreen is called with an ammount of (0.1, 0.1, 0.1) for 0.15 seconds with dontreset to true
- If entity.battleentity.`overrideanim` or noanim is true, the coroutine ends here with a yield break
- entity.battleentity.[animstate](../../Entities/EntityControl/Animations/animstate.md) may be set depending on conditions (mutually exclusive, only the first one applies in order):
  - 21 (`Woobly`) if entity has the `Topple` [condition](../Actors%20states/Conditions.md)
  - 16 (`HurtFallen`), if entity has the `Flipped` [condition](../Actors%20states/Conditions.md)
  - If damage is above 0:
      - 24 (`Block`) if block is true
      - 11 (`Hurt`) if block is false. This is 30 (`FakeHurt`) instead if fakeanim is true
  - If none of the above happened:
      - It's set to 24 (`Block`) if block is true
      - A ShakeSprite coroutine starts on the entity.battleentity with an intensity of (0.1, 0.0, 0.0) and a frametimer of 20.0
- 0.35 seconds are yielded
- If entity has the `Topple` [condition](../Actors%20states/Conditions.md):
    - entity.battleentity.`basestate` is set to 21 (`Woobly`)
    - entity.battleentity.`overrideanim` is set to false
    - entity.battleentity.[animstate](../../Entities/EntityControl/Animations/animstate.md) is set to its `basestate`
- Otherwise, if entity has the `Flipped` [condition](../Actors%20states/Conditions.md):
    - entity.battleentity.`basestate` is set to 15 (`Fallen`)
    - entity.battleentity.`overrideanim` is set to false
    - entity.battleentity.[animstate](../../Entities/EntityControl/Animations/animstate.md) is set to its `basestate`
- Otherwise, if entity.`hp` is above 0:
    - entity.battleentity.`overrideanim` is set to false
    - if entity has the `Sleep` [condition](../Actors%20states/Conditions.md), entity.battleentity.[animstate](../../Entities/EntityControl/Animations/animstate.md) is set to 14 (`Sleep`)
    - Otherwise, if the entity is a player party member with an `hp` of 4 or below, its [animstate](../../Entities/EntityControl/Animations/animstate.md) is set to 17 (`WeakBattleIdle`)
    - Otherwise, entity.battleentity.[animstate](../../Entities/EntityControl/Animations/animstate.md) is set to its value before this coroutine started unless it was 31 (`Dig`) or 32 (`DigMove`) where it's the `basestate` it had at the start instead


