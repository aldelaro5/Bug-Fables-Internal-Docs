# IcePlatform
A component meant to be attached on any GameObject via Unity, but it requires the following to function:

- Any collider in any children of the GameObject
- Any ParticleSystem in any children of the GameObject

This component is specific to involve a platform that only has its collider enabled under the effects of a [StencilSwitch](../../Entities/NPCControl/ObjectTypes/StencilSwitch.md) and scales smoothly accordingly when this changes. It has a Start and LateUpdate (where most of the logic happens). Since the logic is complex, only the public fields will be detailed:

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
