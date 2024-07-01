# PartyDamage
A method specifically for enemy actions that calls [DoDamage](DoDamage.md) for each player party member whose `hp` is above 0. The method also supports adding some visual effects such as jumping or spinning.

```cs
private void PartyDamage(int caller, int damage, AttackProperty? property, bool block, float jumpheight, Vector3 spinammount, bool jumpevenonblock, DamageOverride[] overrides)
```

There is also an overload that relays to the method except for the following parameters which are ommited:

- `jumpheight`: 0.0
- `spinammount`: Vector3.zero
- `jumpevenonblock`: false
- `overrides`: null

```cs
private void PartyDamage(int caller, int damage, AttackProperty? property, bool block)
```

## Parameters

- `caller`: The `enemydata` index that will be used as the attacker of the DoDamage calls
- `damage`: The damageammount that will be passed to the DoDamage calls
- `property`: The property that will be passed to the DoDamage calls
- `block`: The block that will be passed to the DoDamage calls
- `jumpheight`: If this is higher than 0.1 and either `block` is false or `jumpevenonblock`, all the affected player party members will jump once DoDamage is called on them
- `spinammount`: If this is higher than 0.1, a SlowSpinStop will happen on all of the affected player party members after DoDamage is called on them with this value as the spnammount and 60.0 frametime
- `jumpevenonblock`: If `jumpheight` is higher than 0.1, the jumping animation will always happen even if `block` is true. This parameter does nothing if `jumpheight` is 0.1 or lower
- `overrides`: The overrides that will be passed to the DoDamage calls

## Procedure
The following happens for each player party member whose `hp` is above 0:

- DoDamage is called using the parameters mentioned above and the player party member as the target
- If jumpheight is higher than 0.1 and either block is false or jumpevenonblock is true:
    - The player party member's `overridejump` is set to true
    - [Jump](../../Entities/EntityControl/EntityControl%20Methods.md#jump) called on the player party member with an h of jumpheight
    - ResetOverrides invoked on the player party member in 1.25 seconds
- If spinammount is higher than 0.1, SlowSpinStop is called on the player party member with the spinammount and a frametime of 60.0
