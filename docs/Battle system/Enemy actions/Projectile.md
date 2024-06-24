# Projectile
This is a coroutine that is specifically made to be implemented as part of an enemy action involving a [DoDamage](../Damage%20pipeline/DoDamage.md) call through the use of a projectile attacks towards a player party member if their `hp` is above 0.

To check that the coroutine is done, `obj` needs to be checked to be null as it will be destroyed at the very end which will eventually change its value to null.

```cs
private IEnumerator Projectile(int damage, AttackProperty? property, MainManager.BattleData attacker, int playertarget, Transform obj, float speed, float height, string extraargs, string destroyparticle, string audioonhit, string audiomoving, Vector3 spin, bool nosound)
```

There is also an overload available that calls the one above with the same parameters except `height` which will have a value of 0.0 followed by a yield break:

```cs
private IEnumerator Projectile(int damage, AttackProperty? property, MainManager.BattleData attacker, int playertarget, Transform obj, float speed, string extraargs, string destroyparticle, string audioonhit, string audiomoving, Vector3 spin, bool nosound)
```

## Parameters

- `damage`: The damageammount value to send to DoDamage
- `property`: The property value to send to DoDamage
- `attacker`: The attacker value to send to DoDamage
- `playertarget`: The index of the `playerdata` that will be used as the target actor to send to DoDamage. If this is negative or at least the length of `playerdata` or the battleentity of the `playerdata` is null, the coroutine will end early by destroying `obj` followed by a yield break
- `obj`: The projectile object's transform that will be moved during the attack. If it has a `NoMapColor` tag, it will avoid to change all materials of all SpriteRenderers under `obj` to `spritemat` (it's possible to do the same without the tag using the `keepcolor` command, see the section below for details)
- `speed`: The meaning depends on if it's less than 1.0 or not:
    - If it's less than 1.0, this represents a game's frametime fraction that will be added to the `obj`'s movement curve progression factor each frame. Effectively, it means that `obj` will move towards the target over the course of (1 / this value) amount of frames so a value of 0.02 means `obj` will be moved over the course of 50.0 frames
    - If it's at least 1.0, this represent the amount of frames that `obj` will take in its movement curve before reaching the target
- `height`: If it's 0.0, `obj` will be moved using a lerp. If it's not, it will be moved using a BeizierCurve3 with this value being the ymax of the curve
- `extraargs`: An optional list of extra commands that can take effect before or after the DoDamage call. See the section below for more details on the commands and the format. This is optional, a value of null means no commands will take effect
- `destroyparticle`: The name of the particles under the `Prefabs/Particles/` path of the rssources tree to play after the DoDamage call before the projectile gets destroyed. This is optional, a value of null won't play any particles
- `audioonhit`: The name of the AudioClip under the `Audio/Sounds/` path of the resources tree to play after the DoDamage call before the projectile gets destroyed. This is optional, a value of null won't play any AudioClip
- `audiomoving`: The name of the AudioClip under the `Audio/Sounds/` path of the resources tree to play before `obj` starts to move on loop until the projectile gets destroyed. If the name is prepanded with a `@`, it will not play the AudioClip on loop. The name is resolved after removing all `@` in the value to allow this feature to function. This is optional: a value of null or empty string won't play a clip
- `spin`: A multiplier to the game's frametime that controls the angles of `obj` after each frame of movement. The angles gets incremented by the game's frametime * this value each frame. A value of Vector3.zero will cause no angle changes
- `nosound`: If this is true, the DoDamage call will have an overrides value of {[NoSound](../Damage%20pipeline/DoDamage.md#nosound)}. If this is false, the DoDamage call will have an overrides value of null

## extraargs format
This parameter is a `,` separated list of strings each representing a command. Each command is a `@` separated list of strings where the first element is the name of the command and the rest are its parameters if any are supplied.

A command can take effect before `obj` is moved or after the DoDamage call. Each command gets executed in the order than they appear when the moment of effect is reached.

Here are the different commands available and their parameters

|Name|Moment of effect|Parameters|Effect|
|---:|----------------|----------|-----------|
|wait|Before `obj` is moved|<ol><li>float: The amount of secconds to wait</li></ol>|Yield for a certain amount of seconds|
|anim|Before `obj` is moved|<ol><li>string: The animation clip's name to play</li></ol>|Plays an animation clip on the Animator component of `obj`|
|keepcolor|Before `obj` is moved|None|This command will make it so that the material of all SpriteRenderers under `obj` won't be set to `spritemat`. NOTE: If `obj` has a `NoMapColor` tag, the same effect will occur without the need of this command|
|SepPart|After DoDamage call|<ol><li>int: The child index of `obj` to destroy</li><li>float: The amount of time in seconds before the destruction of the child</li></ol>|Destroys a child of `obj` after a certain amount of seconds ellapsed. Right before the destruction, the transform gets rooted and placed offscreen at 999.0 in y|
|resetcam|After DoDamage call|None|Calls [SetDefaultCamera](../Visual%20rendering/SetDefaultCamera.md)|
|animhit|After DoDamage call|<ol><li>string: The animation clip's name to play</li></ol>|Plays an animation clip on the Animator component of `obj`|
|destoffscr|After DoDamage call|<ol><li>float: The ymax of the BeizierCurve3 used to move `obj` offscreen past the player party</li><li>float: The amount of frames - 1.0 to move `obj` offscreen past the player party</li></ol>|Causes the audioonhit and destroyparticle effects to take place earlier before setting their value to null so they don't take effect later. After, `obj` gets moved offscreen past the player party at (-15.0, 0.0, 0.0) using a BeizierCurve3|
|atkdown|After DoDamage call|<ol><li>int: The default amount of turns to inflict [AttackDown](../Actors%20states/BattleCondition/AttackDown.md)</li><li>string: If it's `2`, the amount of turns to inflict [AttackDown](../Actors%20states/BattleCondition/AttackDown.md) is param 3 when `commandsuccess` is true (blocked) or param 1 if it's false (not blocked). If it's `1`, the [condition](../Actors%20states/Conditions.md) is not inflicted if `commansuccess` is true (blocked)</li><li>int: The amount of turns to inflict [AttackDown](../Actors%20states/BattleCondition/AttackDown.md) if param 2 is `2` and `commandsuccess` is true (this does nothing if param 2 isn't `2`)</li></ol>|Calls [StatusEffect](../Actors%20states/Conditions%20methods/StatusEffect.md) to inflict [AttackDown](../Actors%20states/BattleCondition/AttackDown.md) with visual effect on the target for a certain amount of turns. This command supports not inflicting if the player blocked<sup>1</sup> sucessfully or to inflict using a different amount of turns if the player blocked<sup>1</sup>|

1: This logic disregard the FRAMEONE modifier which is [flags](../../Flags%20arrays/flags.md) 615 because even regular blocks will count here.

## Procedure

- playertarget is checked and if it's not in bounds of `playerdata` or if it is, but the battleentity is null, obj is destroyed followed by an early yield break.
- The extraargs commands that are set to take effect before obj movement are executed. See the section above for details
- The target position of obj is set to the `playerdata[playertarget]`'s position + (0.0, 1.0, -0.2)
- Unless a `keepcolor` command was processed or that obj has a tag of `NoMapColor`, all materials of all SpriteRenderers under obj are set to `spritemat`
- If audiomoving isn't null or empty, an AudioSource is added to obj with the clip set to the resource at `Audio/Sounds/` + audiomoving (after removing all `@`), the volume set to MainManager.`soundvolume` and the loop to be true unless audiomoving started with `@` where it's false. The clip is then played immediately
- obj is moved as described by the parameters section above using the speed, spin and height values
- If `playerdata[playertarget]`'s `hp` is above 0, the DoDamage call happens with:
    - attacker: The sent attacker value
    - target: `playerdata[playertarget]`
    - damageammount: The sent damage value
    - property: The sent property value
    - overrides: null if nosound is false, {[NoSound](../Damage%20pipeline/DoDamage.md#nosound)} if it's true
    - block: `commandsuccess`
- The extraargs commands that are set to take effect after the DoDamage call are executed. See the section above for details
- If destroyparticle isn't null and no `destoffscr` commands were processed, PlayParticle is called to play the destroyparticle particles at the obj's position
- If audioonhit isn't null and no `destoffscr` commands were processed, PlaySound is called to play the audioonhit sound
- obj is destroyed
- Yield for a frame
