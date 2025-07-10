# Temp page
TODO: organise this later

## LightSorter
A component meant to be attached on any GameObject via Unity.

It will call MainManager.SortLights on every 2 LateUpdate using all child MeshRenderer of the GameObject.

## KeepAngle
A component meant to be attached on any GameObject via Unity.

It will enforce on every LateUpdate the angles of the GameObject to stay at the `angle` field which can be configured in the inspector. There is also a `getatstart` configuration field that if set to true, it will cause `angle` to be set to the starting angle of the GameObject on Start and to use it instead of the inspector value on LateUpdate.

## PulsingLight
A component meant to be attached on any GameObject via Unity as long as the GameObject has a Light component (it will be collected on the component's Start).

It will on every LateUpdate set the Light's range value to be `basevalue` + Sin(Time.time * `speed`) * `ammount`. The `basevalue`, `speed` and `ammount` values are configurable via the inspector.

## FakeWall
A component meant to be attached on any GameObject via Unity, but it requires a trigger collider on the GameObject to function.

It will cause a call to [PlaySound](../General%20systems/Sounds%20playback.md#playing-sounds) with the `Glow` sound with a pitch of 0.8 and a volume of 1.0 on OnTriggerEnter when the other GameObject is the MainManager.`player`. It will only do this on the first OnTriggerEnter or after an OnTriggerExit has been registered with MainManager.`player`.

## MaterialColor
A component meant to be attached on any GameObject via Unity as long as the GameObject has a Renderer component (it will be collected on the component's Start).

It will set on its Start the Renderer's component's materials\[`materialid`\].color to `color`. Both `materialid` and `color` fields are configurable via the inspector. If the `settag` value is set to true in the inspector, Start will also cause the GameObject tag to be set to `NoMapColor` which will not cause changes to the material's color when entering an inside.

## SoundControl
A component meant to be attached on any GameObject via Unity as long as the GameObject has an AudioSource component (it will be collected on the component's Start).

It will cause the AudioSource's volume to be set to 0.0 on Start until the first LateUpdate that instance.`globalcooldown` expired where it will be set to `startvolume` * MainManager.`soundvolume` (or multiplied by MainManager.`musicvolume` if `musicvolume` is true). Both the `startvolume` and the `musicvolume` bool value are configurable via the inspector.

This is essentially a way for any GameObject producing sounds to get their volume scaled by the in game volume settings. On its own, this volume never changes after the first applicable LateUpdate, but if the settings changes during gameplay, MapControl.RefreshSoundVolume is called which will manually update the volume of all SoundControl in the scene to reflect the new settings.

## LilypadEffects
A component meant to be attached on any GameObject via Unity as long as the GameObject exists in a map in the `WildGrasslands` area.

This is in practice an UNUSED component because while it does feature some logic on Start that would instantiate `Prefabs/Particles/LilypadMove` particles and configure them, it only happens when the `self` field is false. While the field is configurable in the inspector, it never has a value of false except in a prefab that is itself UNUSED.

## EntityTie
A component meant to be attached on any GameObject via Unity.

It will cause on the first LateUpdate after `delay` amount of frames to have MainManager.`map`.`entities[entityid]` have the following happen to them (this component disables itself after):

- Gets childed to the GameObject
- `alwaysactive` set to true
- Locally positioned at `offset`
- Local angles set to `angle` (if the magnitude of the value is above 0.1)

The `delay`, `entityid`, `offset` and `angle` fields are configurable via the inspector.

## InsideSorter
A component meant to be attached on any GameObject via Unity.

It will cause on LateUpdate whenever instance.`insideid` changes to another value to change the enablement of the `child` value which is a GameObject. If the new instance.`insideid` becomes this component's `insideid`, the `child` gets enabled and if not, it gets disabled. This enablement logic can be reversed if the `invert` field is true (so enabled when NOT the inside, disabled if it is). The `child`, `insideid` and `invert` fields are all configurable via the inspector.

Intuitively, it allows to control the enablement of any other GameObject according to being in a specific inside or NOT being in that inside.

## ExpBag
An UNUSED component that was specifically meant for the `Ressources/prefabs/objects/ExpBag` or `Ressources/prefabs/objects/ExpBagOld` where it would have scaled the GameObject according to instance.`partyexp` / instance.`neededexp`. Since both of these prefabs are UNUSED, the component is also UNUSED.

## RenderQueue
A component meant to be attached on any GameObject via Unity, but it requires a Renderer component on the GameObject to function.

It will cause all the materials of the Renderer component (or all the ones whose index is in `materials` if it's not empty) to have their renderQueue set to `queue` 0.1 seconds after Start, then it the component will disable itself. The `materials` and `queue` fields are configurable via the inspector.

## LibraryShelf
This component is specific to the Lore Book functionality of a shelf in the `AntPalaceLibrary` map.

It exposes a method called Refresh that is called on Start and by the librarybook SetText command which will destroy every existing books and recreate them. The amount of books created is given by flagvar 15 (amount of Lore Books given). The GameObject representing the books is in the `book` field which is configured via the inspector.

## ShadowLite
A component meant to be programmatically added to an existing GameObject and have the SetUp method called immediately upon attaching it:

```cs
public void SetUp(float opacity, float size)
```
The `shadowsize` field gets set to the `size` parameter and the `shadowammount` gets set to the `opacity` parameter.

This component manages the SpriteRenderer of a newly created GameObject called `shadow` childed to the GameObject of the component using the MainManager.`shadowsprite` sprite with a pure white color (using `shadowammount` as the opacity).

it essentially acts as a minimal version of the EntityControl's `shadow` functionality. As long as the SpriteRender is visible, its position, scale and angles changes every LateUpdate accoriding to a Raycast test from the GameObject position + 1.0 in y heading down. The `shadow` will look at the normal, be position at the point of the raycast hit and be scaled according to the y of the point (the scale is multiplied by `shadowsize`).

## FlagAnimation
A component meant to be attached on any GameObject via Unity as long as the GameObject has an Animator component (it will be collected on the component's Start).

This components will cause the Animator to play an animation depending on the value of some flags. It has 2 configurable fields via the inspector:

- `flags`: The reverse ordered list of flags slot to test for where the first that is true will be selected as the `anims` index to use. The first element MUST be negative for the component to work
- `anims`: The list of AnimationClip names to use whose indexes needs to match the ones in `flags`. The first element is the falback one where if no `flags` is true, it will be the clip selected instead

The check occurs every LateUpdate, but the animation is only played if the selected index from the 2 arrays above changes. The falback element counts as its own index so it counts in this changed check.

This is only used on a monitor screen at the `RubberPrisonSecurity` map to change the animation displayed on it depending on flag 550 (raised the cafeteria tables) and 553 (turned on the computer in the room).

## StealthCheck
A component that implements part of the StealthAI logic on the applicable NPCControl, check the documentation to learn more.

## CheckOutline
A component meant to be attached anywhere since it operates on all MeshRenderer components in the scene.

On its Start, all materials of all MeshRenderer on the scene using the MainManager.`outlinemain`.shader as their shader will have the following happen to them:

- material color set to pure black
- `_Outline` float property set to `baseoutline` * 2.0 unless they are childed to the `mainmesh` where it's set to the material's GameObject's scale magnitude / 2.0 * `baseoutline`

Additionally, if this is running in the Unity editor, pressing the `0` key will manually call the Start method.

Both the `mainmesh` field GameObject and the `baseoutline` float are configurable via the inspector.

## CrackRockBreak
A component specific to the `Ressources/prefabs/objects/CrashRockBreak` prefab.

This is setup and configured as part of a BreakableRock breaking animation where pieces of the rock will go flying in random direction before fading leading to the destructing of the GameObject. Check the BreakableRock documentation to learn more.

## FaceCamera
A component meant to be attached on any GameObject via Unity or attached programmatically to any GameObject.

On FixedUpdate, the angles of the GameObject will change using the following logic on every FixedUpdate to make it face the game camera:

- If `angle` is false, the change depends on the value of `billboard`:
    - If it's true, the GameObject will LookAt the MainManager.MainCamera
    - If it's false, the GameObject's y angles is set to the MainCam's y angles + `offset`
- If `angle` is true, it leverages a GameObject that was created on Start childed to the attached GameObject reffered to as the `rotater`:
    - `rotater` will LookAt the MainManager.MainCamera (this will implicitly change the GameObject's angles)
    - The GameObject's local xz scales of the GameObject gets inverted compared to the values it had on Start (causes the GameObject to face the opposite direction so away from the camera)
    - The GameObject's y angles is set to the MainCam's y angles + `offset`

The `offset`, `angle` and `billboard` fields are all public and configurable via the inspector.

## MeshMerger
A component meant to be attached on any GameObject via Unity.

On Start, it will merge all the `meshes` MeshRenderer such that `meshes[0]` will be recreated with its materials being the same, but have all the other `meshes` elements combined into it. After, the angles of the GameObject will be set to `rotatefix` Vector3 and if `hasCollider` is true, a MeshCollider will be added where the hasCollider will be the merged mesh. This is an optimisation where less separate meshes will be rendered as all `meshes` will get destroyed before the merge and the recreation of `meshes[0]`.

The `meshes`, `rotatefix` and `hasCollider` fields are all configurable via the inspector.

## Hidder
A component meant to be attached on any GameObject via Unity.

On Start, all of the Renderer components in all the childs of the `objs` GameObject arrays are collected.

On every 2 LateUpdate where MainManager.`player` exists (not loading a map), the enablement of all the collected renders changes depending on if MainManager.`player` xz positions fufills a condition expressed by the `hidepos` float field and the `left` field (it's a Directions enum, but only the values `Right`, `Left`, `Up` and `Down` are valid). The `objs`, `hidepos` and `left` fields are configurable via the inspector. This enablement change only happens if the xz position check actually changes (goes from true to false or vice versa).

It's essentially an optimisation where Renderers may be hidden when the player gets too far away to see them, but they will reappear when the player gets close enough again. Here are the exact conditions evaluated depending on the `left` values which can be intuitively thought as tripwire position checks (the condition being true means that the renderers gets enabled, disabled otherwise):

- `Left`: `hidepos` is less than the MainManager.`player` x position (it's located leftward)
- `Right`: `hidepos` is more than the MainManager.`player` x position (it's located rightward)
- `Up`: `hidepos` is more than the MainManager.`player` z position (it's located towards the back of the map)
- `Down`: `hidepos` is less than the MainManager.`player` z position (it's located towards the front of the map)

> NOTE: This component doesn't ensure that each Renderer isn't null while it's possible one becomes null or gets destroyed after the component's Start so it won't be reflected and it can lead to a NullReferenceException. 

## GlowingAura
A component meant to be attached on any GameObject via Unity. If the component doesn't have a FaceCamera component, one will be added on Start.

This components operates on all the `halos` SpriteRenderer (if it's empty, it's set to all the child SpriteRenderers) where the first, second and third gets different changes to them on every FixedUpdate. Here's the details:

- The first gets its z angle increased by `speed`
- The second gets its z angle decreased by `speed`
- The third gets Rotate called with (`speed`, `speed` / 1.5, `speed` / 2.0)

On top of the above, if `alphavariation` is above 0.0, every `halos` gets their color set to `color` with an alpha set to a lerp from the existing one to `alphavariation` with a factor of the absolute value of Mathf.Sin(`speed` * Time.time) * `frequency`.

The `halos`, `speed`, `alphavariation`, `color` and `frequency` fields are all configurable via the inspector.

## DummyControl
An UNUSED component that is never attached to anything.

It has 5 public fields, a FixedUpdate that can change the angles and scale of the GameObject or destroy it according to those fields and a OnTriggerEnter that may destroy the GameObject.

Its original purpose isn't known and its name doesn't indicate it well.

## RandomPosTime
A component meant to be attached on any GameObject via Unity.

Every `frametime` amount of frames including the first frame on Update (or a random amount of frames between half of `frametime` and `frametime` if `randomizedtime` is true), the GameObject position changes to be its position it had on Start + a random vector where each component is between - (`size` / 2.0) and `size` / 2.0. If `checkground` is true, the position is instead set to the hit point of a Raycast starting from that position heading down for 50.0 units. Optionally, if the Animator `animatorpart` isn't null, the `animname` AnimationClip name is played on it on top of that.

The `frametime`, `randomizedtime`, `size`, `checkground`, `animatorpart`, and `animname` fields are all configurable via the inspector.

## ElecThing
A component specifically used to perform some visual effects on the LineRenderer of 2 GameObjects in the `GiantLairFridgeOutside` map to simulate electricity looking lines. 

Informations about the LineRenderer is collected on Start and the visual effect is performed on each LateUpdate.

## CamChange
A component meant to be attached on any GameObject via Unity as long as the GameObject has a BoxCollider component (it is enforced by a RequireComponent attribute).

It will change the instance.`camoffset`, instance.`camangleoffset` and instance.`camspeed` to this component's `offset`, `angle` and `speed` fields respectively when OnTriggerEnter happens with the MainManager.`player`. The camera fields are restored to their original value when OnTriggerExit happens with the MainManager.`player`.

The `offset`, `angle` and `speed` fields are all configurable via the inspector. It is essentially a simpler and external version of the CameraChange Object that doesn't require a map entity to operate.

## RayDetector
The wall detector component programmatically added to an EntityControl when CreateDetector is called. Check its documentation to learn more.

## DestroyOnLayer
A component meant to be programmatically added to an existing GameObject and have the SetUp method called immediately upon attaching it:

```cs
public void SetUp(string deathparticle, float particletime, int targetlayer, Vector3 partoffset, Vector3 partangle)
public void SetUp(string deathparticle, float particletime, int targetlayer, Vector3 partoffset, Vector3 partangle, bool actuallydontdestroy)
```
The default value of `actuallydontdestroy` on the first overload is false.

This component will destroy or move offscreen the GameObject when an OnTriggerEnter happens where the other collider is on the `targetlayer` layer. It is destroyed if `actuallydontdestroy` is false and moved -9999.0 offscreen if it's true (more specifically, the parent is moved if it exist and the GameObject is moved if it doesn't). If `deathparticle` isn't null, when the trigger collision happens, PlayParticle is called with the particle being `deathparticle` without sound at the GameObject position + `partoffset` with an angle of `partangle` for a time of `particletime`.

## FollowerLite
A component meant to be programmatically added to an existing GameObject and have the SetUp method called immediately upon attaching it:

```cs
public void SetUp(float dist, Transform following, float spd)
```
On Start, the GameObject is set to layer 9 (`Follower`), a RigidBody component gets added with rotation frozen and a BoxCollider component gets added with a center y of 0.5.

Every 3 LateUpdate including the first one where `following` isn't null, the following happens:

- If `following` is disabled, the GameObject is disabled too
- If the Vector3.Distance between the GameObject and `following` is greater than 5.0, the GameObject position is set to `following` position
- The GameObject is set to be in range of `following` if Vector3.Distance between the GameObject and `following` is greater than `dist`

At the end of every LateUpdate, if the GameObject is in range of the `following`, the GameObject position is set to a lerp between its position and `following` position with a factor of TieFramerate(`speed`).

This component is essentially a lighter version of the EntityControl's follow system, but can be attached to any entity. It is only used during the UpdateAnimSpecific of a `Ruffian` animid for its ball and chain to follow them.

## LightFlicker
A component meant to be attached on any GameObject via Unity as long as the GameObject has a Light component (it is collected on its Start).

This component will periodically change the Light's intensity on Update, but there is a cooldown period and complex logic involving several configuration fields set via the inspector to simulate a light periodically flickering at random intervals. Here's a breakdown of how this works:

- There's a cooldown expressed in amount of frames that on Start and when it expires (decreased on every Update), its value will be set to `frequency` + a random value between -`random` and `random`
- The cooldown above changes the logic depending if it's expired or not:
    - If it's not expired, every `framecount` frames, the Light's intensity is set to a lerp from the existing one to the value it had on Start with a factor of `speed` * MainManager.`framestep`
    - If it is expired and it's one of the first `duration` frames of it expiring, the Light's intensity is set to a lerp from the existing one to `targetintensity` with a factor of `speed` * MainManager.`framestep`
    - If it is expired and it's been more than `duration` frames of it expiring, the new value of the cooldown is set using the formula mentioned above. However, there is a `fastflickerpercent` chance that the value gets divided by `fastdivider` before setting the final cooldown value which will make it lower. This fast flicker can't happen on 2 cycles in a row

## HelpArrow
A component meant to be programmatically added to an existing GameObject by calling the NewArrow static method:

```cs
public static HelpArrow NewArrow(Transform parent, Vector3 offset, Color color, float distance, float size)
```
The method creates a GameObject named `Arrow` childed to `parent` with a local position of `offset` and this component attached. 

On Start, a SpriteRenderer is added using the `guisprites[196]` sprite (an arrow) using the MainManager.`spritedefaultunity` material on layer 15 (`3DUI`) initially disabled

The last 3 parameters of NewArrow configure the component's private fields:

- `color`: When satisfying the `distance` condition below, this is the color the arrow SpriteRenderer will go towards (more precisely, it's set to a lerp from pure white to this value with a factor of the absolute value of Sin(Time.time * 5.0))
- `distance`: Decides the maximum distance MainManager.`player` is allowed to be away from the `Arrow` before the SpriteRenderer gets disabled (it is enabled as long as this condition is fufilled)
- `size`: The scale of the arrow SpriteRendrer on Start is set to (0.45, 0.6, 1.0) * this value

There is also an exposed `lockarrow` field that when set to true will force the `distance` check to fail so the SpriteRenderer is always disabled.

Most of the logic happens on Update, but it won't be detailed here for the sake of brevity. Essentially, it is a compoennt specific to when MainManager.`player` is `Beetle` when near enough to a GameObject with this component to indicate the angle at which the object will go when using `Beetle`'s Horn Slash ability on it. Specifically, it's used for:

- Hornable of type `IceCube`
- NPCControl of type Enemy (`lockarrow` is set to true unless they are frozen)
- Any PushRock with a `data[2]` that doesn't exist or is 0

## VenusBattle
A component meant to be programmatically added to an existing GameObject and have the SetUp method called immediately upon attaching it:

```cs
public void SetUp(EntityControl parent, EntityControl targetentity)
```
It's a component that implements specific battle related logic where 2 entities needs to have their animation synchronised. All the logic happens in LateUpdate as long as `parent` and `targetentity` aren't null and `targetentity`.`overrideanim` is false.

It only has logic for the following `targetentity`.animid which won't be detailed for the sake of brevity:

- Venus
- SandWyrmTail
- SandWyrm

## DialogueAnim
A component meant to be programmatically added to an existing GameObject and have the SetUp method called immediately upon attaching it:

```cs
public void SetUp(Vector3 tpos, float spd)
public void SetUp(Vector3 tsize, Vector3 tpos, float spd)
public void SetUp(Vector3 startsize, Vector3 tsize, Vector3 tpos, float spd)
```
The method sets the following fields (any not sent fields aren't set, but the `tsize` value defaults to Vector3.zero):

- `speed` is set to the `spd` value
- `targetscale` is set to the `tsize` value
- `targetpos` is set to the `tpos` value
- The `startsize` sets the GameObject's scale

It's a general purpose component that allows any GameObject to gain smooth scaling animations combined with optional local movement animation. It does the following on FixedUpdate of various kinds. All of its fields are public and on top of the ones mentioned above, it's possible to configure this component further by setting other fields.

Here's what the component does on FixedUpdate:

- The local scale is changed according to the configuration using a lerp from the existing one with a facctor of `shrinkspeed`, but the destination depends on the configuration:
    - If `flipx` is true, the value is (0.0, existing y scale * `multiplier`, 1.0)
    - Otherwise, if `flipy` is true, the value is (existing x scale * `multiplier`, 0.0, 1.0)
    - Otherwise, if `shrink` is true, the value is Vector3.zero
    - Otherwise, the value is `targetscale` * `multiplier`
- If `targetpos` isn't Vector3.zero, the GameObject's local position is set to a lerp from the existing one to `targetpos` with a factor of `speed`

## ShakeHorn
A component meant to be attached on any GameObject via Unity, but it requires the following to function:

- Any trigger collider on the GameObject
- Any ParticleSystem in any children of the GameObject (optional, if not present, particles functionality won't work)

On Start, the GameObject gets some modifications:

- isStatic set to false
- If there's no RigidBody, one gets added without gravity in kinematic with position frozen

The component only does something on OnTriggerEnter in any of the 2 conditions:

- The other collider has the `BeetleHorn` or `BeetleDash` tag (`Beetle`'s overworld abilities). If this is the case, it will cause a call to MainManager.HitPart to the other collider's postion + 1.0 in y
- `playermoveshake` is true and the other collider is the MainManager.`player`

If any condition happens, a coroutine starts (stopped if it was currently ongoing) that will perform some visual effects on the GameObject to rotate it in a sporadic manner to simulate a shake effect. Here's the details:

- If the `sound` AudioClip is isn't null, MainManager.PlaySoundAt is called to play its name at 1.0 volume from the GameObject position
- If there was a child ParticleSystem detected on Start, Emit is called on it with a count of `emission`
- Over the course of 61.0 frames, the angles of the GameObject each frames changes to be their initial angles from Start + `axis` * Sin(Time.time * 30.0) * (1.0 - X) where X is a number from 0.0 to 1.0 that increases as it gets closer to 61.0 frames

## SpriteParticleCaller
A component meant to be attached on any GameObject via Unity.

All the logic are contained in LateUpdate where as soon as a specific map entity's sprite (or a specific Renderer) becomes a specific sprite, some particle effects will be played (instantiated if they don't exist) close to the GameObject. This is all configurable via fields in the inspector:

- `position`: The position (or local position if `relativeposition` is true) of the particles that will play when the watched Renderer's sprite becomes `targetsprite`
- `renderer`: The specific Renderer to watch for its sprite becoming `targetsprite`. If it's null, it will use MainManager.`map`.`entities[mapentityid]`.`sprite` instead
- `mapentityid`: If Renderer is null, this is the map entity id whose `sprite` will be watched for becoming `targetsprite`
- `targetsprite`: The sprite to watch the Renderer to become. When the Renderer's sprite becomes this one, the particles will play
- `particleprefab`: The prefab of the particles to play, it needs to contain a ParticleSystem at its root for particles to play
- `timer`: If above -1.0 or above, the particles gets destroyed in `timer` amount of seconds each time they are created (it may be recreated later when it gets to null)
- `onlyone`: If this is false, it will prevent particles to be replayed after they have been instantiated so it forces them to be recreated. If it's true, it will allow the particles to replay from the start if they have been created already
- `relativeposition`: If this is true, `position` refers to the local position of the particles that will be set when they are played. If this is false, it refers to the position instead
- `parented`: If this is true, the particles GameObject will be childed to the attached GameObject when created

The instantiated particles will always have a local scale of 1.0x. Also, when the watched sprite becomes `targetsprite`, the particles plays, but won't play again until the sprite changes to another before becoming `targetsprite` again.

## MidPos
A component meant to be programmatically added to an existing EntityControl's `sprite` or via Unity to a `model` entity prefab.

This component implements the ability for multiple Transform to be linked in such a way that it simulates a chain where each nodes is in a list of Transform and they move together. The logic is very complex so for the sake of brevity, only the public fields will be documented. The component has a Start and a LateUpdate (where most of the logic happens).

Here are the public fields and their effects:

- `links`: The list of Transform to link together. If `getfromchild` is true, this will be set to all the child Transform on Start
- `start`: The position of the start of the chain. If `getstartandendfromlink` is true, this will be overriden to `links[0]` position. If `localpos` is true, this refers to the local position instead
- `end`: The position of the end of the chain.  If `getstartandendfromlink` is true, this will be overriden to the last `links` element's position. If `localpos` is true, this refers to the local position instead
- `middle`: If the magnitude of this is above 0.1, the chain will form a BeizierCurve3 from start to end with this value being the midpoint of the curve. Otherwise, the chain will move form a straight line using a lerp
- `localpos`: If true, `start` and `end` refers to the local position of the chain instead of the position
- `getstartandendfromlink` If true, `start` and `end` are overriden to `links[0]` position and the last `links` element position respectively (this is incompatible with `localpos` being true)
- `getfromchild`: If true, `links` will be set to all child Transform on Start
- `ismodel` If true and `parenttransform` isn't null, the related EntityControl (used to determined if their `sprite` has `flip`) is assumed to be in the grand grand parent of `parenttransform`. Otherwise, it's assumed to be int the `parenttransform` directly
- `parenttransform`: If this isn't null, the Transform where an EntityControl resides related to the chain (if `ismodel` is also true, it's assumed to be on its grand grand parent instead)

## Caravan
A component meant to be attached on any GameObject via Unity as long as the GameObject has an Animator in a child who also have a SpriteRenderer components (they are collected on the component's Start).

This component is specific to the animation logic of the Caravan shop's snail where it implements some visual effects. Most of the logic involves if the `Sleep` or `Idle` AnimationClip should be played on the snail's animator. Additionally, it features enablement updates of the snail's sprites if the current inside changes. Both of these happens on LateUpdate.

Since the logic is complex, it won't be documented here, but the inspector configuration fields will be:

- `isslep`: This should always be false because this component manages it itself (it has no good reason to be public)
- `facingright`: Whether the sprite's logical facing direction is to the right, it's assumed to be the left if it's false
- `insideid`: The map's inside id where the snail's sprite will remain enabled and disabled elsewhere. A value of -1 means being in any inside will disable it while not being in any will enable it

On MoveInside, the first Caravan's Refresh method will be called which will update the AnimationClip to play according to the `isslep` value. It is assumed that there will only be one Caravan component in the scene at the time.

## MiniBubble
A component meant to be programmatically added to a new GameObject using the SetUp static method:

```cs
public static MiniBubble SetUp(string text, EntityControl target, Vector3 pos, int sortingorder)
public static MiniBubble SetUp(string text, EntityControl target, Vector3 pos, int sortingorder, float timer)
```
The default value of `timer` on the first overload is 1.5.

This component is specific to the minibubble SetText command implementation, check its documentation to learn more.

The only public field is the `target` EntityControl that the minibubble refers to. Outside of the SetUp methods above, the only public method is DestroyThis which cleanly destroys the MiniBubble's GameObject.

## ParticleRange
A component meant to be attached on any GameObject via Unity or attached programmatically, but it needs a ParticleSystem in any children of the GameObject to function.

This component continuously plays some particles (the first child ParticleSystem) as long as the MainManager.`player` gets close enough and stops playing when they are further away. It also features the ability to fade all child MeshRenderer from Color.Clear to a color smoothly as the MainManager.`player` gets in and out of range. Since the logic is complex, only the public fields will be detailed. It has a Start and LateUpdate (where most of the logic is).

Here's the details of the public fields:

- `radius`: The max Vector3.Distance away the MainManager.`player` needs to be to remain the particles playing. Falling further away than this distance will stop the particles from playing. It also controls the fading distance in the same way
- `fadeframeammount`: The maximum amount of frames the fading takes of the `render` towards `matcolor`
- `getmaterialcolor`: If true, `matcolor` is overriden to the first `render`.material.color on Start
- `getmeshes`: If true, `render` is overriden to all child MeshRenderer on Start
- `fademesh`: This field is UNUSED and does nothing
- `invertfademesh`: If true, the fading of the `render` elements is reversed and it will go towards Color.Clear away from `matcolor` instead when the MainManager.`player` gets in range
- `matcolor`: The color to fade towards the `render` elements's materials when the MainManager.`player` gets in range (the opposite happens when out of range, but both directions can be swapped if `invertfademesh` is true)
- `render`: The MeshRenderers to have their material.color fade towards `matcolor` as the MainManager.`player` gets in range and out of range

The range check and the playback of particles only happens every 3 LateUpdate.

## RandomColor
A component meant to be attached on any GameObject via Unity anywhere as it doesn't manage its own GameObject, but it manages other MeshRenderer.

This component periodically changes other MeshRenderer's material.color to specific ones with multiple mods available and optional randomness in timings or colors. All the logic is contained in LateUpdate, but since it's complex, only the public fields will be detailed:

- `type`: An enum value that tells how to decide the colors to use for the `renders`:
    - `RandomFromList`: Randomly selects a color from the `colors` array whenever the color needs to be changed
    - `Rainbow`: Always use MainManager.RainbowColor(`variant`) when changing the color
    - `InOrder`: The same as `RandomFromList`, but without randomness so it goes from first to last repeatedly
- `colors`: The list of colors to cycle the `renders` when `type` isn't `Rainbow`
- `offcolor`: When the `flag` condition isn't fufilled, this is the color all `renders` will be set to (up to once if `setoffonce` is true)
- `renders`: The MeshRenderer whose material.color will get cycled by this component periodically
- `flag`: If this is -1, the color cycling logic always happen on LateUpdate. If it's 0 or above, it only happens when flags slot `flag` is true as otherwise, the color cycling is disabled and all `renders`'s material.color gets set to `offcolor` instead. NOTE: Any value of -2 or below are invalid and will cause an exception to be thrown
- `speed`: This field is UNUSED and does nothing
- `frametime`: The minimum amount of frames between color cycling
- `variant`: This serves both as the maximum amount of frames to add to `frametime` each cycle to calculate the cooldown in frames to wait before the next color cycling. It also serves as the parameter to MainManager.RainbowColor when `type` is `Rainbow`
- `setoffonce`: If this is true, whenever the `flag` check fails, the `renders` will have their material.color set to `offcolor` only once for the lifetime of this component
- `index`: The starting index - 1 value to use for when `type` is `InOrder`. Any starting value of -2 or below are invalid and will cause an exception to be thrown (-1 works and it means 0)

## PrefabParticle
A component meant to be attached on any GameObject via Unity.

This component instantiates a bunch of instances of a particles prefab and manages them all together with various configuration. This component honors the particle levels settings set from the config file. The component has a Start where the instances are created and a LateUpdate where most of their management happens. The logic is complex so only the public fields will be detailed:

- `prefabpart`: The prefab particle to instantiate up to `maxammount` amount of times
- `maxammount`: The amount of instances of particles to created, but if the particle settings in the config file is set to LOW, this amount is halved (floored) on Start
- `speed`: The magnitude to move the particles on each LateUpdate from their heading direction
- `liveframes`: This specifies the time in amount of LateUpdates that the particle will grow towards `maxsize`. It is random between `liveframes` / 2.0 and `liveframes` each time a cycle happens
- `cooldown`: The additional amount of frames to wait after the `liveframes` period is over when the particles will start to shrink towards Vector3.zero
- `shrinkspeed`: The MainManager.TieFramerate parameter to use as the lerp factor when growing or shrinking the particles. The growth however is done with MainManager.TieFramerate(`shrinkspeed` / 2.0) instead
- `limits`: The position of each particles when cycling them is random between -`limits` and `limits`
- `maxsize`: The maximum size each particles will have during the `liveframes` period
- `childspin`: The rotation vector to use to rotate each particles on each LateUpdate. If this is Vector3.zero, no rotation happens

If the particle level in the config file is set to OFF, this component gets disabled on Start.

## SpinAround
A component meant to be added programmatically or via Unity to a GameObject. If done programmatically, the StartUp method needs to be called:

```cs
public void StartUp(Transform spintarget, float spinfrequency, float spinspeed, float yfrequency, float yspeed, float yoffset)
```
It sets the following fields (explained below):

- `target`: `spintarget`:
- `sf`: `spinfrequency`
- `ss`: `spinspeed`
- `yf`: `yfrequency`
- `ys`: `yspeed`
- `offset`: `yoffset`

This component has all of its fields public so more can be configured on top of the method above.

It's a general purpose component to have any kinds of GameObject rotate in various ways around an axis or move around another GameObject continuously in a circle around it. It has a Start and an Update (which contains most of the logic). Since the logic is complex, only the fields will be detailed:

- `target`: If not null, this is the GameObject the attached one will move around of. If `gettarget` is true, it is overriden to the parent of the attached GameObject on Start
- `itself`: If this isn't Vector3.zero, it will be the axis of rotation used to rotate the attached GameObject around and the `target` gets ignored unless `itselfandtarget` is true where both the rotation and movement around happens
- `gettarget`: If true, the `target` field gets overriden to the attached GameObject's parent on Start
- `local`: If true, the GameObject will rotate by increased its local angles by `itself` * MainManager.`framestep` instead of calling Rotate(`itself` * MainManager.`framestep`)
- `requiresflag`: If it's not -1, flags slot `requiresflag` needs to be true for the component to do anything on Update. Any value of -2 and below are invalid and will cause an exception to be thrown
- `sf`: See below for the rotation formula, only applicable when moving around `target`
- `ss`: See below for the rotation formula, only applicable when moving around `target`
- `yf`: See below for the rotation formula, only applicable when moving around `target`
- `ys`: See below for the rotation formula, only applicable when moving around `target`
- `offset`: See below for the rotation formula, only applicable when moving around `target`
- `randoms`: If this isn't 0.0, the value of `sf` and `ss` are overriden to be random between -`randoms` and `randoms` on Start
- `randomy`: If this isn't 0.0, the value of `yf` and `ys` are overriden to be random between -`randoms` and `randoms` on Start

When moving around `target`, this is the formula used to set the new position:

`target` position + a Vector3 with the following components:

- x: Sin(Time.time * `sf`) * `ss`
- y: Sin(Time.time * `yf`) * `ys` + `offset`
- z: (0.0 - Cos(Time.time * `sf`)) * `ss`

## IcePlatform
A component meant to be attached on any GameObject via Unity, but it requires the following to function:

- Any collider in any children of the GameObject
- Any ParticleSystem in any children of the GameObject

This component is specific to involve a platform that only has its collider enabled under the effects of a StencilSwitch and scales smoothly accordingly when this changes. It has a Start and LateUpdate (where most of the logic happens). Since the logic is complex, only the public fields will be detailed:

- `center`: The local position offset that when added to the attached GameObject's position, it is considered as the center point whose distance to the StencilSwitch will be checked against
- `growspeedstep`: The maximum amount of frames that the platform will grow from Vector3.zero when in range of the StencilSwitch that's activated or shrink to Vector3.zero when not in range or deactivated
- `smoothgrow`: If true, the grow/shrink of the platform is done via MainManager.SmoothLerp instead of a lerp
- `playsound`: If true, the `Freeze` or `IceMelt` sound is played at the GameObject position + `center` when they become in range of the StencilSwitch or not respectively
- `obj`: The GameObject representing the platform. If this is null, it's set to the first child of the attached GameObject on Start

The distance check happens after 4 frames since the last (includes the first frame). For the check to succeed, the following all needs to be true:

- MainManager.`map`.`stencilid` isn't negative (a StencilSwitch is activated)
- MainManager.`map`.`entities[MainManager.map.stencilid]`.`npcdata`.`hit` is true (meaning the current StencilSwitch is activated)
- The Vector3.Distance of the attached GameObject position + `center` and the StencilSwitch's position is less than the StencilSwitch `vectordata[0].y` (its radius) * 1.85

The first child ParticleSystem is played when the state becomes in range and stopped when it becomes out of range.

If `playsound` is true, the sound is only played when transitioning to in and out of range.

The first Collider has its enablement updated to be enabled only if the local scale of `obj` has a magnitude above 0.5 (disabled otherwise). Since it gets shrinked towards Vector3.zero when going out of range, this is what will lead to the platform's Collider being disabled when out of range of the StencilSwitch and enabled when in range.

## ConditionChecker
A component meant to be attached on any GameObject via Unity.

This is a general purpose component to essentially disable or move offscreen a GameObject that doesn't fufill certain conditions regarding flags slots and regionals flags. It implements this using MainManager.CheckIfCanExist which is used throughout the game and it is the system NPCControl also uses for disabling themselves when their flags requirements aren't met. The purpose of using this component is it allows to have the same logic, but done completely standalone from any GameObject in Unity. It has a Start where most of the logic happen and a LateUpdate that may move the GameObject offscreen after the fact.

It has several public fields to configure its behavior:

- `requires`: The require parameter to send to MainManager.CheckIfCanExist which is a list of flags slots that must be true for the GameObject to exist (can be empty or have its first element be negative to have no required flag slots)
- `limit`: The limit parameter to send to MainManager.CheckIfCanExist which is a list of flags slots where any among them must be false for the GameObject to exist (can be empty or have its first element be negative to have no limit flag slots)
- `regionID`: The regionalflag parameter to send to MainManager.CheckIfCanExist which is a regionalflag slot of the current area that must be false for the GameObject to exist (can be negative to have no regionalflag slot requirement)
- `spriteflagchange`: A flag slot that if true on Start, it will cause the GameObject's SpriteRenderer (if it exists)'s sprite to change to `spritetochange`. If the value is negative, this feature is disabled
- `activepos`: The local position (or position if `worldpos` is true) to set the GameObject on LateUpdate if it fails the MainManager.CheckIfCanExist requirements. If the magnitude of this is 0.1 or below, LateUpdate checks effectively are disabled
- `startpos` (not editable via the inspector): This contains the GameObject position it had on Start, but it is exposed programmatically (meant for reading)
- `startangle` (not editable via the inspector): This contains the GameObject angles it had on Start, but it is exposed programmatically (meant for reading)
- `spritetochange`: The sprite to change the GameObject's SpriteRenderer to if the `spriteflagchange` check is fufilled and the feature is enabled
- `dontdelete`: If true, the GameObject won't be disabled on Start if it fails to meet the MainManager.CheckIfCanExist requirements
- `worldpos`: This determines what the `activepos` will do if the MainManager.CheckIfCanExist requirements during LateUpdate. If it's true, the GameObject's position is set to `activepos` and if it's false, the GameObject's local position is set to `activepos`
- `pauseactive`: If this is true, the MainManager.CheckIfCanExist requirements are never disabled on LateUpdate, even when the game is paused after 20.0 frames

Essentially, there's 2 parts divided in the Start and LateUpdate:

- Start does 2 things:
    - If `dontdelete` is false, MainManager.CheckIfCanExist is called (this feature is disabled otherwise). If it succeeds, nothing happens, but if it fails, what happens depends on if the GameObject where if it has an NPCControl, its `entity`.`iskill` is set to true which will effectively cause NPCControl to disable it. If it's not an NPCControl, it's disabled and moved offscreen at 9999.0 in y with the Animator component destroyed if it exists
    - If `spriteflagchange` isn't negative and the GameObject has a SpriteRenderer, its sprite is set to `spritetochange`
- LateUpdate only manages one thing if `activepos`'s magnitude is above 0.1 (otherwise, this feature is disabled). It involves the GameObject to `activepos` (by position or local position depending on `worldpos`) if another MainManager.CheckIfCanExist check fails. This is only checked every 5 LateUpdate cycles, but after 20.0 frames, checks are disabled if the game is paused (unless `pauseactive` is true in which case checks happens even if the game is paused). It's essentially a way to move the GameObject out of the way after the fact if it's not destroyed by then

## PromptAnim
A component meant to be programmatically added to a new GameObject using the SetUp static method:

```cs
public void SetUp(int selectid, bool underlines)
```
The `selectid` is the `option` refered to by the text and `underlines` is whether or not to render a red `_` under the option if selected.

This component is specific to the choicewave command, check its documentation to learn more. The component has a Start that setups everything and most of the animation is done on FixedUpdate. It is what implements the visuals of the prompt choice highlight.

## TempItemDesc
A component meant to be attached on any GameObject via Unity.

This component is only used in the `TermiteIndustrial` map to implement the flow of purchasing the `BombPlus` medal. Some explanation is in order for why such a component is needed.

In order for the game to render a description box near a shelved item or medal, a shop system needs to be defined. This will handle everything from creation to the purchasing flow and it includes rendering description boxes when in the interact range of the SemiNPC shelved items / medals. So in theory, it could be possible to define a medal shop with only 1 medal, but in practice, it would be much more complicated. This is because defining a medal shop has great implications due to the game making assumpions on how they work. For example, it would have an impact on the save file because it contains data about the medals shops's stock and it would .

Since it's not possible to add a medal shop with 1 medal, this component was made to compensate. It "fakes" the presence of a description box when going near the GameObject with configurable fields even if it is only used for a single occasion:

- `requireEntity`: A map entity entity id that needs to be present (not null) for the component to remain enabled
- `insideID`: The inside id that the GameObject is in (used to see if the player gets close enough as the inside id needs to match too). If this is -1, the GameObject isn't in any insides
- `itemType`: The general type of the item/medal refered to by the GameObject. 0 means standard item, 1 means key item and 2 means medal
- `itemID`: The item or medal id refered to by the GameObject
- `itemValue`: A buying price in berries to show in the description box
- `range`: The maximum Vector3.Distance that the MainManager.`player` can be away from the GameObject while still having the description box rendered
- `itemSprite`: The sprite to show as the item/medal to buy. If `itemType` is 2 (medal) and flags 681 is true (MYSTERY? is active), this is overriden to MainManager.`guisprites[190]` (the ? gray medal sprite)

This component only handles rendering the description box, it doesn't handle the actual purchase, but this can be done using an Event interaction on an NPCControl.

The component has a Start that will enforce the MYSTERY? sprite mentioned above, but also disable the component so no description box gets rendered on MYSTERY? (this is inconsistent with the rest of the game). It also has an OnDestroy that will destroy the description box UI.

Finally, it has an Update where most of the logic happens including the `range` and `insideID` checks.

## SpriteBounce
A component meant to be added programmatically or via Unity to a GameObject. If done programmatically, the SetUp method needs to be called:

```cs
public void SetUp(float freq, float spd)
```
`freq` is used to set the `frequency` value and `spd` is used to set the `speed` value of the component (details below).

This component is a general purpose bouncing scaling animation that simulates some form of bouncing visuals. It has a Start and a FixedUpdate where most of the logic resides. It also features many configurations available, here's all the public fields that configures it:

- `maxx`: See the formula below when the scaling of the GameObject changeds on FixedUpdate
- `maxy`: See the formula below when the scaling of the GameObject changeds on FixedUpdate
- `frequency`: See the formula below when the scaling of the GameObject changeds on FixedUpdate
- `speed`: See the formula below when the scaling of the GameObject changeds on FixedUpdate
- `ydifference`: See the formula below when the scaling of the GameObject changeds on FixedUpdate
- `basescale`: See the formula below when the scaling of the GameObject changeds on FixedUpdate. If `startscale` is true, the value is overriden to the GameObject's local scale on Start
- `spritecolor`: If the GameObject has a SpriteRenderer, its sprite's color will be set to this value on Start
- `facecamera`: If true, a FaceCamera component is added to the GameObject on Start
- `startscale`: If true, the value of `basescale` is overriden to the GameObject's local scale on Start
- `requiresflag`: If it's not negative, a flag slot that must be true for the GameObject's local scale to change in FixedUpdate
- `requiresentity`: If it's not negative, a map entity id whose matching `entity`.`npcdata`.`hit` needs to be true for the GameObject's local scale to change in FixedUpdate

Each FixedUpdate that the `requiresflag` and `requiresentity` checks passes, the GameObject's local scale is set to a value using the following formula:

`basescale` + a Vector3 with the following components:

- x: Sin(Time.time * `speed`) * `frequency` * `maxx`
- y: Cos(Time.time * `speed` + `ydifference`) * `frequency` * `maxy`
- z: 0.0

There's also a public method that can be called after the fact: MessageBounce:

```cs
public void MessageBounce()
public void MessageBounce(float multiplier)
```
The default `multiplier` value of the first overload is 1.0.

It will set `frequency` to 0.1 * `multiplier` and `speed` to 7.0 * `multiplier`.

## ConveyorSprite
A component meant to be attached on any GameObject via Unity.

This component implements a visual effect with multiple sprites are initially positioned along a path made of nodes and they each moves towards the next node until reaching it. They are created using MainManager.NewSpriteObject at each nodes childed to this GameObject using their respective sprites. This process repeats indefintely so it simulates the sprites moving along a conveyer path using lerps.

It has a Start and a LateUpdate where most of the logic happens. Here are all the public fields:

- `type`: An enum value that dictates how the `sprites` will be used when placing the nodes on Start:
    - `AlternateEach`: They are ordered as they appear in the `sprites` array
    - `RandomEach`: Each node is a random sprite from the `sprites` array
- `nodes`: The positions of all the nodes in Vector3 specified in `sprites` order
- `sprites`: The matching sprites used for each nodes, it must have the same amount of elements as `nodes`
- `framespeed`: The amount of frames each nodes takes to move to their next node
- `particle`: When reaching the next node, all of these particles names will be played using MainManager.PlayParticle at the matching `nodes`
- `billboard`: If true, each sprite object will get a FaceCamera component on Start with a `billboard` field set to true

## GroundDetector
A component meant to only be attached to the `Prefabs/GroundDetector` prefab and used programmatically.

This component implements the logic of ground detection that is added by EntityControl to applicable entities. It allows the game to consult if the entity is `onground`.

Check the CreateFeet documentation to learn more on how this prefab is typically instantiated.

## FaderRange
A component meant to be attached on any GameObject via Unity, but it requires to have Renderers children to function (collected on Start).

This component will fade / unfade all of the child Renderer depending on if the MainManager.`player` position or the MainManager.`MainCamera`.WorldToViewportPoint's z falls within a range or not of the GameObject position . It also manages the renderQueue and shader of the Renderer throughout this process.

Here's all the public fields:

- `maxdistance`: The maximum Vector3.Distance away the MainManager.`player` z position or the upper bound the MainManager.`MainCamera`.WorldToViewportPoint's z can be from the GameObject position + `pivot` before the Renderers starts to unfade
- `mindistance`: The minimum Vector3.Distance away the MainManager.`player` z position or the upper bound the MainManager.`MainCamera`.WorldToViewportPoint's z can be from the GameObject position + `pivot` before the Renderers starts to fade
- `fadedelay`: The parameter of MainManager.TieFramerate to use as the lerp factor when fading or unfading the Renderers
- `fadepercent`: The alpha value of `color` to use as the destination of the lerp when fading the Renderers
- `color`: The color to use as the destination of the lerp when unfading the Renderers and also the RGB parts of the color when fading them
- `pivot`: The offset to apply to the GameObject position before using it for the range check
- `invert`: If this is true, any fade becomes unfades and vice-versa
- `player`: If this is true, the range check is done with the MainManager.`player` position instead of MainManager.`MainCamera`.WorldToViewportPoint(`pivot` + GameObject's position)
- `ignorematerial`: If this is true, it does 2 things:
    - When unfading, it always happen even if the Renderer's material.shader is MainManager.`Fade3D`.shader
    - The shader of the Renderers won't change after the distance check in LateUpdate and the color won't be set to pure white if the alpha gets higher than 0.9
- `framestep`: The amount of frames between distance checks in LateUpdate

Some other details to note:

- On Start: all Renderers gets the `NoMapColor` which will not cause changes to the material's color when entering an inside
- On Start, all Renderers using the MainManager.`outlinemain`.shader will have their renderQueue set to 3000
- No distance checks happens on LateUpdate if any of the following is true:
    - MainManager.`player` is null (loading a map)
    - The Renderer's shader is MainManager.`fakelight`. In which case, what happens instead is their renderQueue are set to 3000 + 100.0 * the z position of MainManager.`player` (or the MainManager.`MainCamera`.WorldToViewportPoint if `player` is false)
    - The Renderer's shader isn't MainManager.`Fade3D`.shader or MainManager.`Main3D`.shader

## Wind
A component meant to be attached on any GameObject via Unity as long as the GameObject has a TrailRenderer component (it will be collected on the component's Start).

This component implements the visual effects of having wind trails of various sizes move horizontally accross the screen. It is done by placing the GameObject on Start, it moves to the right or left until a certain point where it will stop moving. Once its trail is barely visible after stopping, Start is called again to repeat this cycle. The movement is done on FixedUpdate. On LateUpdate, the TrailRenderer's material.renderQueue is set to 5000.

Here's the public fields:

- `origins`: The base position used for the GameObject on Start. If this is null, the GameObject position is the base instead
- `insideid`: The inside id the GameObject needs to be in for its TrailRenderer to remain enabled. It is disabled otherwise. This is checked on FixedUpdate
- `randomizer`: This influences the maximum amount of FixedUpdate cycle before the GameObject will move when stopped as soon as the TrailRenderer's endWith gets to 0.5 or lower. More speicically, each time the wind is stopped, the amount is random between 60.0 * TrailRenderer's time and this value
- `limit`: The x position limit the GameObject can be before it stops and teleports somewhere else. If this is negative, the GameObject moves in -x, +x otherwise. There is always an implied 5.0 units of grace period where the GameObject can go 5.0 over this value in the respective direction
- `bobammount`: See the y position moving formula below
- `bobfrequency`: See the y position moving formula below
- `horizontalOffset`: This specifies the range that the z position of the GameObject will be on Start on each movement cycle. The z position will be `oirigins`.z + (random between -`horizontalOffset` and `horizontalOffset`)
- `VerticalOffset`: This specifies the range that the initial y position of the GameObject will be on Start on each movement cycle. The y position will be `oirigins`.y + (random between -`VerticalOffset` and `VerticalOffset`)
- `speed`: The amount of units the GameObject moves in x on each FixedUpdate that it moves
- `owncenter`: This value does nothing. It influences if `center` is set, but that field also does nothing
- `center`: This value does nothing, but it may be assigned from Start to MainManager.`map`.`centralpoint` if the current `map` exists (not loading one) and `owncenter` is false

Each FixedUpdate that the GameObject moves, the new y position is calculated using the following formula:

Starting y position since the last Start + Sin((Time.time + R) * `bobammount`) * `bobfrequency` where R is a random number between 0 and 9 that is regenerated on each Start.

## FontEffects
A component meant to be programmatically attached to a letter slot.

It is the component that emplements several SetText font effects commands, check their documentation to learn more:

- wavy
- shaky
- rainbow
- glitchy
- fadeletter

## MusicSpinner
A component meant to be attached on any GameObject via Unity, it will get a RigidBody component on its own on Start.

This component is only used in the `BugariaTheater` map to implement GameObjects spinning as a sound is played multiple times in succession with varying pitch and length as it is spinned using `Beetle`'s Horn Slash field ability. When it has spun enough, an item is launched from it if it wasn't obtained already.

While it's only used there, it has several configurations available. Here are the public fields:

- `notes`: The list of pitches and length of each times to play the `soundclip` during a full spinning cycle. Each x componets represents the timestamps the playback should happen in seconds and each y components represents the pitch to use for the playback
- `maxtime`: The maximum playback time of all the `notes` in a full spinning cycle before restarting from the first `notes` element
- `spinlimit`: The spin value requited (the value increases in increments of `spinhit` each Horn Slash) before the item spawns if it wasn't obtained already
- `maxspin`: The maximum spin value (the value increases in increments of `spinhit` each Horn Slash)
- `spinstop`: An amount of frames that presents the value to decrease the spin value on each LateUpdate (the value increases in increments of `spinhit` each Horn Slash)
- `spinhit`: The increment to increase the spin value of the `spinner` on each Horn Slash (done on OnTriggerEnter with any `BeetleHorn` tagged colliders)
- `musiclower`: The volume multiplier to use as the destination of a lerp applied to `music[0]`.volume on each LateUpdate while the music is playing (the lerp is done from the existing volume with a factor of the spin / `maxspin` where spin is clamped from 0.0 to `maxspin` so it approaches the scaled volume more as the cylinder spins more). This is mostly used to fade the current music using a value between 0.0 and 1.0
- `flag`: A flag slot indicating if the item was obtained already (also bound to the NPCControl's `activationflag` of the created item when launched if `itemtype` isn't 3 so not a Crystal Berry). If `itemtype` is 3 (Crystal Berry), this value is a crystalbflags instead
- `itemtype`: The type of item launched from the cylinder:
    - 0: Standard item
    - 1: Key item
    - 2: Medal
    - 3: Crystal Berry (with the crystalbflags specified in `flag`)
- `itemid`: The item, medal id or crystalbflags of the launched item. NOTE: If `itemtype` is 3 (Crystal Berry), this value MUST match the `flag` because they're both used differently, but represents the same thing
- `itemtime`: The amount of time in frames to spawn the item before it disappears. If it's -1.0, the item won't disappear
- `itempos`: The position to spawn the item before it is launched
- `bouncepos`: The direction to launch the item once it is spawned
- `spinner`: The list of GameObject to rotate using the spin value. All of them's isStatic will be set to false if they weren't when they spin
- `rotation`: The axis of rotate of each `spinner` as they rotate
- `soundclip`: The base sound to use to play the `notes`
- `musicvolume`: This field is UNUSED and it doesn't do anything.

On Start, the GameObject gets a Rigidbody component added without gravity in kinematic. Additionally, if the item hasn't been obtained yet and the `Detector` medal is equipped, MainManager.`map`.`hiddenitem` is set to 100 which lets the beeper sound play.

All the spinning and item spawning logic happens in LateUpdate and the logic to detect Horn Slash to start the spin happens in OnTriggerEnter where the collider has the `BeetleHorn` tag.

## StaticModelAnim
A component meant to be attached on any GameObject programmatically or via Unity, but the GameObject has to have a Renderer component.

This component is a general purpose bobbing and texture shifting animation. It is used for a wide variety of reasons such as animating water by texture shifting or bobbing some object periodically by moving it slightly. Most features of the component are optional.

Here's all the public fields:

- `speed`: The direction of `_MainTex` shifting where the magnitude is expressed is scaled by MainManager.`framestep` and represents the rate of the shifting
- `current`: This act as the current shifting offset to `_MainTex` (it increases by `speed` * MainManager.`framestep` every Update). This can be set externally, but it's typically going to be Vector3.zero initially
- `limitmin`: The value the `current` value will loop back to when it reaches `limitmax`. This is calculated per component so the x and y will cycle independently
- `limitmax`: The maximum value the `current` value can have before looping back to `limitmin`. This is calculated per component so the x and y will cycle independently
- `bobspeed`: A Vector3 that expresses the speed of the bobbing, calculated per components. If this value and `bobfreq` are Vector3.zero, no bobbing happens
- `bobfreq`: A Vector3 that expresses the period of the bobbing, calculated per components. If this value and `bobspeed` are Vector3.zero, no bobbing happens
- `bobangle`: If this isn't Vector3.zero, it changes the bobbing to bob the angles instead of the position of the GameObject
- `conveyor`: This field isn't used by the component, but if it is set via the inspector, the game can access it. This is only used for PlayerControl to move if they are on their `conveyor` and the value controls the direction of the movement
- `nomove`: If this is true, bobbing is disabled and can't happen
- `pausetied`: If this is true, it will do 2 things:
    - Nothing will happen on the first FixedUpdate if the game is paused
    - On Update, `internalt` won't increase by MainManager.`framestep` if the game is paused
- `internaltimer`: If this is true, the way angle bobbing happens (which is only applicable if `bobangle` isn't Vector3.zero) changes where instead of using Time.time in the oscilation, `internalt` * Time.deltatime is used instead
- `onlyonce`: If this becomes true, the GameObject gets disabled on the next LateUpdate
- `firstcycle`: This becomes true after the first LateUpdate is done, but it is exposed to the game
- `stopbob`: If this is true, movment bobbing is disabled and can't happen, but angle bobbing can still happen if `bobangle` isn't Vector3.zero
- `requiresflag`: This is a flag slot that needs to be true for `internalt` to increase if all other conditions are satisfied. If this value is -1, no flags is required
- `internalt`: If `internaltimer` is true, this value * ime.deltatime is used instead as the base for angle oscillation rather than Time.time. On Update, if all conditions are fufilled, the value increases by MainManager.`framestep`

If `bobfreq` or `bobspeed` aren't Vector3.zero, the GameObject's isStatic is set to false on Start.

## DynamicFont
A component meant to be added programmatically or via Unity to a GameObject. If done programmatically, the StartUp method needs to be called:

```cs
public static DynamicFont SetUp(bool nokerning, float frequency, int fonttype, int sortorder, Vector2 fontsize, Transform parent, Vector3 position)
public static DynamicFont SetUp(string displaytext, bool centralized, bool nokerning, float frequency, int fonttype, int sortorder, Vector2 fontsize, Transform parent, Vector3 position, Color color)
public static DynamicFont SetUp(string displaytext, bool centralized, bool nokerning, float frequency, int fonttype, int sortorder, Vector2 fontsize, Transform parent, Vector3 position, Color color, Vector2? small)
```
The default `small` parameter is null for the first 2 overloads and for the first overload, here are what the other unsent parameters defaults to:

- `displaytext`: Empty string
- `centralized`: false
- `color`: Color.white

It will create a GameObject named `DynFont - X` where `X` is `displaytext` with a DynamicFont component childed to `parent` (or MainManager.`GUICamera`.transform if it's null) with a local scale of 1.0x. The component is configured with the following fields (see below for details):

- `startpos`: `position`
- `text`: `displaytext`
- `center`: `centralized`
- `monospace`: `nokerning`
- `updatefrequency`: `frequency`
- `fontindex`: `fonttype`
- `fontcolor`: `color`
- `size`: `fontsize`
- `sort`: `sortorder`
- `smallersize`: `small`

This component is important because it serve as a secondary way to render text that doesn't features any of the versatility then SetText, but it does have one huge advantage over it: it is made for text that constantly changes. SetText has to manage a lot of ressources (and it's very inefficient at it) and it has a letter slot limit on top of having a lot of overhead to just render the text or process the commands, but DynamicFont doesn't have such limits.

This is because all DynamicFont does on Update is simply to change the text field of all the TextMeshes. There's no materials being created, no effects, just the text changes. Since Unity is particularily efficient at handling this, it makes DynamicFont really performant for dynamically rendering text that reacts to the game. A prime example of in game usage is rendering HP numbers in the HUD: since they are expected to change a lot, it's much more efficient to use DynamicFont. The rendering only happens once on Start where one GameObject is created per TextMesh and they aren't rendered again. Only changes to the text are monitored on Update, but it also has a configurable throttle to reflect those changes.

Here are the public fields:

- `center`: This field doesn't do anything
- `monospace`: This field doesn't do anything
- `dropshadow`: If this is true, 2 TextMeshes will be rendered per letter where the second one will have a color of pure black, but half transparent, be locally positioned at `dropoffset` and have a MeshRenderer's sortingOrder being the same as the first TextMesh - 1
- `tridimentional`: If true, the TextMeshes will render on layer 0 (`Default`) instead of `layer`, but `triui` can still take priority if it is true
- `triui`: If true, the TextMeshes will render on layer 15 (`3DUI`) instead of `layer`. This takes precedance over `tridimentional`
- `text`: The text to render. Any changes to this field will be reflected on Update to the TextMeshes
- `updatefrequency`: The amount of frames to wait before Update actually updates the TextMeshes's text to `text`
- `fontindex`: The font id of the text to render (expressed as a MainManager.SetFont's fontid parameter)
- `sort`: The MeshRenderer's sortingLayer of the TextMeshes are set to this value + 1 (+ 0 is for the secondary TextMeshes if `dropshadow` is true)
- `layer`: The layer to render the TextMeshes when `tridimensional` and `triui` are both false
- `size`: The local x and y scale of the TextMeshes to render. Specifically, the local scale on render is set to (`size`.x, `size`.y, 1.0) * 0.07
- `dropoffset`: The local position of each secondary TextMesh when `dropshadow` is true
- `startpos`: The initial local position of the GameObject
- `fontcolor`: The color of the TextMeshes to render

Some additional notes about Start:

- `fontindex` on Start is set to MainManager.FontID(`fontindex`) so it represents the actually resolved font id after
- If the parent of the GameObject is still null by then, it is set to MainManager.`GUICamera`.transform
- The local angles are set to Vector3.zero

And some additional notes about the rendering:

- All TextMeshes's GameObjects are childed to the attached GameObject
- All TextMeshes's anchor are set to LowerLeft
- All TextMeshes's tag are set to `Text`
- All TextMeshes's local angles are set to Vector3.zero

## OverworldProjectile
A component meant to be added programmatically by calling the NewProjectile method:

```cs
public static OverworldProjectile NewProjectile(NPCControl parent, int spriteindex, Vector3 startpos, Vector3 target, Vector3 spin, Vector3 angle, Vector3 size, string particleondeath, float arc, float time, float shadowsize)
```
It will create the GameObject with the component on it.

This component is specific to the ShootProjectile ActionBehavior or related behavioirs, check the documentation to learn more.

## AnimationFunctions
A component meant to be attached on any GameObject via Unity.

This is a component made to hold many animation related methods meaning it doesn't really have logic on its own, but rather provides logic meant to be called by Unity's Animator using an animation event. It's meant to be attached to the GameObject where animations happens so that the methods can be called by the Animator.

The component does have a Start though that will collect the EntityControl component if `getentity` and `model` are true which is assumed to be at the grand grant parent of the GameObject. This enables some methods that requires to operate on the entity for a model entity. The `getentity` and `model` fields are configurable via the inspector.

## Pips
A component specifically meant for the `Prefabs/Objects/Pip0` and `Prefabs/Objects/Pip1` prefabs.

They implement the pips dropping logic where they can heal 1 HP or TP each when trigger colliding and they have a limited spawn time. More information on this component can be found in the Death's pips drop logic section.

## Audience
A component meant to be added programmatically or via Unity to a GameObject.

This is a component meant to hold many instances of the `Prefabs/Objects/Audience` prefab located closely together in a defined area with configurations to render their sprite through different AnimationClips selected more or less randomly. They are all childed to the attached GameObject while being locally positioned differently within a range.

The prefab has an Animator and a SpriteRenderer at its root. The animator controls the sprite to use through the `animationcontrollers/_misc/audience` controller which features clips named in specific way. Their naming format is `X_Y` where `X` is a number from 0 to 5 that represents the audience member kind to render and `Y` is the specific sprite number to use (0 is idle, 1 is cheering sprite). All audience member's sprites are gray because they are meant to be colorised at runtime by the component using colors that are randomly selected within a range using a lerp from one color to another with the factor being random. Here are the audience member kind mappings and their color range:

- 0: Moth - from pure yellow to pure red with the factor between 0.2 and 0.8
- 1: Ant - from pure yellow to pure red with the factor between 0.5 and 0.9
- 2: Beetle - from pure green to pure blue with the factor between 0.2 and 0.8
- 3: Bee - from pure yellow to pure red with the factor between 0.2 and 0.45
- 4: Termite - from pure white to pure yellow with the factor between 0.2 and 0.5
- 5: Termite (older looking) - from pure red to pure yellow with the factor between 0.3 and 0.45

Since most of the logic are complex animations, only the global details will be given. Here are all the public fields:

- `animtype`: An enum value that determines how to select the audience member kind:
    - `MothAntBeetle`: Random between 0 and 2
    - `OnlyAnt`: 1
    - `OnlyBee`: 3
    - `OnlyBeetle`: 2
    - `OnlyMoth`: 0
    - `All`: Random between 0 and 3
    - `Termites`: Random between 4 and 5
- `ammount`: The amount of audience members to spawn
- `currentammount`: This field holds `ammount` after Start, but it isn't used for anything
- `spawnarea`: The xz local position range to use for each audio member. More precisely, the range is random between (-(`spawnarea`.x / 2.0), 0.0, -(`spawnarea`.y / 2.0)) and (`spawnarea`.x / 2.0, 0.0, `spawnarea`.y / 2.0)
- `constantjump`: The 2 components of this Vector2 controls how to change the y local position of the audience members on LateUpdate periodically. If this is Vector2.zero, this feature is disabled
- `noflip`: This value does nothing
- `lowfps`: This value does nothing
- `entities` (not configurable via the inspector): The Animators of all the audience members. It is exposed to the game.

Most of the setup is done on Start, but there is an OnEnable that will force all audience members on their idle animation. It's also possible to do the same externally by calling the RefreshAnim public method. The `constantjump` feature is implemented in LateUpdate as well as the ability to periodically change their y angles.

Finally, there's a Jump public method that starts a coroutine that has all audience members temporarilly go to their cheering animation while either jumping, rotating in y or both.

## Hornable
A component meant to be programmatically added to a new GameObject and calling the SetUp method immediately after adding it:

```cs
public void SetUp(Vector2 pushammount, bool pusher, bool breakondash, NPCControl createdfrom)
```
This is a component specific to a Dropplet's ice cube, check its documentation to learn more.

## GlowTrigger
A component meant to be added programmatically or via Unity to a GameObject.

TODO: This is complex, document later

## Fader
A component meant to be added programmatically or via Unity to a GameObject.

This is a general purpose component meant to fade the GameObject attached when it is obstructing the camera view. The logic is too complex to detail, but the map can configure them and this is documented in the fader control section of the graphics documentation.

## ScrewPlatform
TODO: this is complex, document later
