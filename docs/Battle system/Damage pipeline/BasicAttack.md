# BasicAttack
A coroutine made for enemy actions that ends up calling [DoDamage](DoDamage.md) from an enemy attacker to a single player party member, but with the ability to configure a simple attack animation logic. The animation involved them moving towards the player party member and then attacking with some animstate changes. This is meant to be used for enemy actions with simple attack animations.

It is meant to set `checkingdead` to the coroutine which will becomes null upon completion of the coroutine allowing the caller on yield on it.

```cs
private IEnumerator BasicAttack(int enemyid, int walkstate, int attackstate, int attackstate2, int damage, Vector3 offset, AttackProperty? property, float shake, float delay, string sounds, bool dontgettarget)
```

## Parameters

- `enemyid`: The `enemydata` index of the enemy party member who is going to be the attacker in the DoDamage call. It is not possible to specify a null attacker
- `walkstate`: The walkstate of the [MoveTowards](../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) before the DoDamage call
- `attackstate`: The stopstate of the [MoveTowards](../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) before the DoDamage call
- `attackstate2`: The animstate to set the enemy party member right before the DoDamage call happens
- `damage`: The damageammount to send to the DoDamage call
- `offset`: The offset to apply to the `playertargetentity` position as the destination of the [MoveTowards](../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) call
- `property`: The property to send to the DoDamage call
- `shake`: If it is above 0.0, the intensity of a ShakeSprite call to do on the enemy party member after they moved. If it's 0.0 or lower, the call doesn't happen
- `delay`: The time in seconds to yield for right before the DoDamage call, but after the movement. It also specify 1/60th of the frametimer of the ShakeSprite call if `shake` is above 0.0
- `sounds`: Specify the sounds to play before the yield and after the yield. 3 formats of this value are possible:
    - null or empty string: no sounds are played before and after the yield
    - A non empty string without any `,` or a string that ends with `,`: The string represents the name of the AudioClip under `Audio/Sounds/` from the ressources tree to play before the yield, but no sound will be played after the yield
    - A non empty string with a `,` in the middle: The portion of the string before the `,` represents the name of the AudioClip under `Audio/Sounds/` from the ressources tree to play before the yield and the portion after the `,` represents the same, but played after the yield 
- `dontgettarget`: If true, [GetSingleTarget](../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) won't be called. This is meant to only be used if it was called already by the caller as this coroutine will call it if the value is false

## Procedure

- If dontgettarget is false, [GetSingleTarget](../Actors%20states/Targetting/GetRandomAvaliablePlayer.md#getsingletarget) is called
- Camera moves to look near `playerdata[playertargetID]`
- `enemydata[enemyid]` [MoveTowards](../../Entities/EntityControl/EntityControl%20Methods.md#movetowards) `playertargetentity` position + offset with a 2.0 multiplier, a walkstate of the sent value and a stopstate of attackstate
- Yield all frames until the `forcemove` of the enemy party member is done
- If shake is above 0.0, ShakeSprite is called on `enemydata[enemyid]` with an intensity of shake and a frametimer of delay * 60.0
- If sounds has a before yield sound in its value, it is played
- Yield for delay seconds
- If sounds has an after yield sound in its value, it is played
- The enemy party member animstate is set to attackstate2
- [DoDamage](DoDamage.md) is called with the attacker, target, damage and property leaving block as `commandsuccess` and overrides as null
- Yield for 0.5 seconds
- `checkingdead` is set to null which informs the caller that the coroutine is done
