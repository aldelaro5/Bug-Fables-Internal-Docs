# Rendering components
These components deals with rendering, visual effects and rendering animations.

## LightSorter
A component meant to be attached on any GameObject via Unity.

It will call MainManager.SortLights on every 2 LateUpdate using all child MeshRenderer of the GameObject.

## PulsingLight
A component meant to be attached on any GameObject via Unity as long as the GameObject has a Light component (it will be collected on the component's Start).

It will on every LateUpdate set the Light's range value to be `basevalue` + Sin(Time.time * `speed`) * `ammount`. The `basevalue`, `speed` and `ammount` values are configurable via the inspector.

## MaterialColor
A component meant to be attached on any GameObject via Unity as long as the GameObject has a Renderer component (it will be collected on the component's Start).

It will set on its Start the Renderer's component's materials\[`materialid`\].color to `color`. Both `materialid` and `color` fields are configurable via the inspector. If the `settag` value is set to true in the inspector, Start will also cause the GameObject tag to be set to `NoMapColor` which will not cause changes to the material's color when entering an inside.

## InsideSorter
A component meant to be attached on any GameObject via Unity.

It will cause on LateUpdate whenever instance.`insideid` changes to another value to change the enablement of the `child` value which is a GameObject. If the new instance.`insideid` becomes this component's `insideid`, the `child` gets enabled and if not, it gets disabled. This enablement logic can be reversed if the `invert` field is true (so enabled when NOT the inside, disabled if it is). The `child`, `insideid` and `invert` fields are all configurable via the inspector.

Intuitively, it allows to control the enablement of any other GameObject according to being in a specific [inside](../MapControl/Insides.md) or NOT being in that inside.

## RenderQueue
A component meant to be attached on any GameObject via Unity, but it requires a Renderer component on the GameObject to function.

It will cause all the materials of the Renderer component (or all the ones whose index is in `materials` if it's not empty) to have their renderQueue set to `queue` 0.1 seconds after Start, then it the component will disable itself. The `materials` and `queue` fields are configurable via the inspector.

## ShadowLite
A component meant to be programmatically added to an existing GameObject and have the SetUp method called immediately upon attaching it:

```cs
public void SetUp(float opacity, float size)
```
The `shadowsize` field gets set to the `size` parameter and the `shadowammount` gets set to the `opacity` parameter.

This component manages the SpriteRenderer of a newly created GameObject called `shadow` childed to the GameObject of the component using the MainManager.`shadowsprite` sprite with a pure white color (using `shadowammount` as the opacity).

it essentially acts as a minimal version of the [EntityControl](../Entities/EntityControl/EntityControl.md)'s `shadow` functionality. As long as the SpriteRender is visible, its position, scale and angles changes every LateUpdate accoriding to a Raycast test from the GameObject position + 1.0 in y heading down. The `shadow` will look at the normal, be position at the point of the raycast hit and be scaled according to the y of the point (the scale is multiplied by `shadowsize`).

## CheckOutline
A component meant to be attached anywhere since it operates on all MeshRenderer components in the scene.

On its Start, all materials of all MeshRenderer on the scene using the MainManager.`outlinemain`.shader as their shader will have the following happen to them:

- material color set to pure black
- `_Outline` float property set to `baseoutline` * 2.0 unless they are childed to the `mainmesh` where it's set to the material's GameObject's scale magnitude / 2.0 * `baseoutline`

Additionally, if this is running in the Unity editor, pressing the `0` key will manually call the Start method.

Both the `mainmesh` field GameObject and the `baseoutline` float are configurable via the inspector.

## FaceCamera
A component meant to be attached on any GameObject via Unity or attached programmatically to any GameObject.

On FixedUpdate, the angles of the GameObject will change using the following logic on every FixedUpdate to make it face the game camera:

- If `angle` is false, the change depends on the value of `billboard`:
    - If it's true, the GameObject will LookAt the MainManager.[MainCamera](../General%20systems/Camera%20system.md#main-camera)
    - If it's false, the GameObject's y angles is set to the MainCam's y angles + `offset`
- If `angle` is true, it leverages a GameObject that was created on Start childed to the attached GameObject reffered to as the `rotater`:
    - `rotater` will LookAt the MainManager.[MainCamera](../General%20systems/Camera%20system.md#main-camera) (this will implicitly change the GameObject's angles)
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

## RandomPosTime
A component meant to be attached on any GameObject via Unity.

Every `frametime` amount of frames including the first frame on Update (or a random amount of frames between half of `frametime` and `frametime` if `randomizedtime` is true), the GameObject position changes to be its position it had on Start + a random vector where each component is between - (`size` / 2.0) and `size` / 2.0. If `checkground` is true, the position is instead set to the hit point of a Raycast starting from that position heading down for 50.0 units. Optionally, if the Animator `animatorpart` isn't null, the `animname` AnimationClip name is played on it on top of that.

The `frametime`, `randomizedtime`, `size`, `checkground`, `animatorpart`, and `animname` fields are all configurable via the inspector.

## ElecThing
A component specifically used to perform some visual effects on the LineRenderer of 2 GameObjects in the `GiantLairFridgeOutside` map to simulate electricity looking lines. 

Informations about the LineRenderer is collected on Start and the visual effect is performed on each LateUpdate.

## CamChange
A component meant to be attached on any GameObject via Unity as long as the GameObject has a BoxCollider component (it is enforced by a RequireComponent attribute).

It will change the instance.`camoffset`, instance.`camangleoffset` and instance.`camspeed` to this component's `offset`, `angle` and `speed` fields respectively when OnTriggerEnter happens with the MainManager.`player`. The camera fields are restored to their original value when OnTriggerExit happens with the MainManager.`player`. For more information on the camera fields, check the [camera system](../General%20systems/Camera%20system.md) documentation.

The `offset`, `angle` and `speed` fields are all configurable via the inspector. It is essentially a simpler and external version of the [CameraChange](../Entities/NPCControl/ObjectTypes/CameraChange.md) Object that doesn't require a map entity to operate.

## LightFlicker
A component meant to be attached on any GameObject via Unity as long as the GameObject has a Light component (it is collected on its Start).

This component will periodically change the Light's intensity on Update, but there is a cooldown period and complex logic involving several configuration fields set via the inspector to simulate a light periodically flickering at random intervals. Here's a breakdown of how this works:

- There's a cooldown expressed in amount of frames that on Start and when it expires (decreased on every Update), its value will be set to `frequency` + a random value between -`random` and `random`
- The cooldown above changes the logic depending if it's expired or not:
    - If it's not expired, every `framecount` frames, the Light's intensity is set to a lerp from the existing one to the value it had on Start with a factor of `speed` * MainManager.`framestep`
    - If it is expired and it's one of the first `duration` frames of it expiring, the Light's intensity is set to a lerp from the existing one to `targetintensity` with a factor of `speed` * MainManager.`framestep`
    - If it is expired and it's been more than `duration` frames of it expiring, the new value of the cooldown is set using the formula mentioned above. However, there is a `fastflickerpercent` chance that the value gets divided by `fastdivider` before setting the final cooldown value which will make it lower. This fast flicker can't happen on 2 cycles in a row

## VenusBattle
A component meant to be programmatically added to an existing GameObject and have the SetUp method called immediately upon attaching it:

```cs
public void SetUp(EntityControl parent, EntityControl targetentity)
```
It's a component that implements specific battle related logic where 2 entities needs to have their animation synchronised. All the logic happens in LateUpdate as long as `parent` and `targetentity` aren't null and `targetentity`.`overrideanim` is false.

It only has logic for the following `targetentity`.[animid](../Enums%20and%20IDs/AnimIDs.md) which won't be detailed for the sake of brevity:

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

## SpriteParticleCaller
A component meant to be attached on any GameObject via Unity.

All the logic are contained in LateUpdate where as soon as a specific [map entity](../Entities/NPCControl/NPCControl.md)'s sprite (or a specific Renderer) becomes a specific sprite, some particle effects will be played (instantiated if they don't exist) close to the GameObject. This is all configurable via fields in the inspector:

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

## Caravan
A component meant to be attached on any GameObject via Unity as long as the GameObject has an Animator in a child who also have a SpriteRenderer components (they are collected on the component's Start).

This component is specific to the animation logic of the Caravan shop's snail where it implements some visual effects. Most of the logic involves if the `Sleep` or `Idle` AnimationClip should be played on the snail's animator. Additionally, it features enablement updates of the snail's sprites if the current inside changes. Both of these happens on LateUpdate.

Since the logic is complex, it won't be documented here, but the inspector configuration fields will be:

- `isslep`: This should always be false because this component manages it itself (it has no good reason to be public)
- `facingright`: Whether the sprite's logical facing direction is to the right, it's assumed to be the left if it's false
- `insideid`: The map's [inside](../MapControl/Insides.md) id where the snail's sprite will remain enabled and disabled elsewhere. A value of -1 means being in any inside will disable it while not being in any will enable it

On [MoveInside](../MapControl/Insides.md#moveinside), the first Caravan's Refresh method will be called which will update the AnimationClip to play according to the `isslep` value. It is assumed that there will only be one Caravan component in the scene at the time.

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

This component periodically changes other MeshRenderer's material.color to specific ones with multiple configuration available and optional randomness in timings or colors. All the logic is contained in LateUpdate, but since it's complex, only the public fields will be detailed:

- `type`: An enum value that tells how to decide the colors to use for the `renders`:
    - `RandomFromList`: Randomly selects a color from the `colors` array whenever the color needs to be changed
    - `Rainbow`: Always use MainManager.RainbowColor(`variant`) when changing the color
    - `InOrder`: The same as `RandomFromList`, but without randomness so it goes from first to last repeatedly
- `colors`: The list of colors to cycle the `renders` when `type` isn't `Rainbow`
- `offcolor`: When the `flag` condition isn't fufilled, this is the color all `renders` will be set to (up to once if `setoffonce` is true)
- `renders`: The MeshRenderer whose material.color will get cycled by this component periodically
- `flag`: If this is -1, the color cycling logic always happen on LateUpdate. If it's 0 or above, it only happens when [flags](../Flags%20arrays/flags.md) slot `flag` is true as otherwise, the color cycling is disabled and all `renders`'s material.color gets set to `offcolor` instead. NOTE: Any value of -2 or below are invalid and will cause an exception to be thrown
- `speed`: This field is UNUSED and does nothing
- `frametime`: The minimum amount of frames between color cycling
- `variant`: This serves both as the maximum amount of frames to add to `frametime` each cycle to calculate the cooldown in frames to wait before the next color cycling. It also serves as the parameter to MainManager.RainbowColor when `type` is `Rainbow`
- `setoffonce`: If this is true, whenever the `flag` check fails, the `renders` will have their material.color set to `offcolor` only once for the lifetime of this component
- `index`: The starting index - 1 value to use for when `type` is `InOrder`. Any starting value of -2 or below are invalid and will cause an exception to be thrown (-1 works and it means 0)

## PrefabParticle
A component meant to be attached on any GameObject via Unity.

This component instantiates a bunch of instances of a particles prefab and manages them all together with various configuration. This component honors the particle levels settings set from the [config file](../External%20data%20format/Config%20File.md). The component has a Start where the instances are created and a LateUpdate where most of their management happens. The logic is complex so only the public fields will be detailed:

- `prefabpart`: The prefab particle to instantiate up to `maxammount` amount of times
- `maxammount`: The amount of instances of particles to created, but if the particle settings in the config file is set to LOW, this amount is halved (floored) on Start
- `speed`: The magnitude to move the particles on each LateUpdate from their heading direction
- `liveframes`: This specifies the time in amount of LateUpdates that the particle will grow towards `maxsize`. It is random between `liveframes` / 2.0 and `liveframes` each time a cycle happens
- `cooldown`: The additional amount of frames to wait after the `liveframes` period is over when the particles will start to shrink towards Vector3.zero
- `shrinkspeed`: The MainManager.TieFramerate parameter to use as the lerp factor when growing or shrinking the particles. The growth however is done with MainManager.TieFramerate(`shrinkspeed` / 2.0) instead
- `limits`: The position of each particles when cycling them is random between -`limits` and `limits`
- `maxsize`: The maximum size each particles will have during the `liveframes` period
- `childspin`: The rotation vector to use to rotate each particles on each LateUpdate. If this is Vector3.zero, no rotation happens

If the particle level in the [config file](../External%20data%20format/Config%20File.md) is set to OFF, this component gets disabled on Start.

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
- `requiresflag`: If it's not -1, [flags](../Flags%20arrays/flags.md) slot `requiresflag` needs to be true for the component to do anything on Update. Any value of -2 and below are invalid and will cause an exception to be thrown
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

## PromptAnim
A component meant to be programmatically added to a new GameObject using the SetUp static method:

```cs
public void SetUp(int selectid, bool underlines)
```
The `selectid` is the `option` refered to by the text and `underlines` is whether or not to render a red `_` under the option if selected.

This component is specific to the [choicewave](../SetText/Individual%20commands/Choicewave.md) SetText command, check its documentation to learn more. The component has a Start that setups everything and most of the animation is done on FixedUpdate. It is what implements the visuals of the prompt choice highlight.

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
- `requiresflag`: If it's not negative, a [flag](../Flags%20arrays/flags.md) slot that must be true for the GameObject's local scale to change in FixedUpdate
- `requiresentity`: If it's not negative, a [map entity](../Entities/NPCControl/NPCControl.md) id whose matching `entity`.`npcdata`.`hit` needs to be true for the GameObject's local scale to change in FixedUpdate

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

## FaderRange
A component meant to be attached on any GameObject via Unity, but it requires to have Renderers children to function (collected on Start).

This component will fade / unfade all of the child Renderer depending on if the MainManager.`player` position or the MainManager.[MainCamera](../General%20systems/Camera%20system.md#main-camera).WorldToViewportPoint's z falls within a range or not of the GameObject position . It also manages the renderQueue and shader of the Renderer throughout this process.

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

## AnimationFunctions
A component meant to be attached on any GameObject via Unity.

This is a component made to hold many animation related methods meaning it doesn't really have logic on its own, but rather provides logic meant to be called by Unity's Animator using an animation event. It's meant to be attached to the GameObject where animations happens so that the methods can be called by the Animator.

The component does have a Start though that will collect the [EntityControl](../Entities/EntityControl/EntityControl.md) component if `getentity` and `model` are true which is assumed to be at the grand grant parent of the GameObject. This enables some methods that requires to operate on the entity for a [model entity](../Entities/EntityControl/Notable%20methods/AddModel.md#addmodel). The `getentity` and `model` fields are configurable via the inspector.

## GlowTrigger
A component meant to be added programmatically or via Unity to a GameObject.

TODO: This is complex, document later

## Fader
A component meant to be added programmatically or via Unity to a GameObject.

This is a general purpose component meant to fade the GameObject attached when it is obstructing the camera view. The logic is too complex to detail, but the map can configure them and this is documented in the [fader control](../MapControl/Graphics%20configuration.md#fader-control) section of the graphics documentation.
