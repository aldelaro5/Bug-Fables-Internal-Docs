# GlowTrigger
A component meant to be added programmatically or via Unity to a GameObject.

This component has 2 purposes with one being optional: it is both a glow effect that can be conditionally activated or deactivated and it can also be used to have a [Hazard](Hazard.md) attached that will be active for certain periods of time periodically. Some Objects NPCControl supports having a GlowTrigger with or without the hazards feature:

- [PathPlatform](../../Entities/NPCControl/ObjectTypes/PathPlatform.md)
- [RotatingPlatform](../../Entities/NPCControl/ObjectTypes/RotatingPlatform.md)
- [Geizer](../../Entities/NPCControl/ObjectTypes/Geizer.md)
- [PressurePlate](../../Entities/NPCControl/ObjectTypes/PressurePlate.md)
- [Switch](../../Entities/NPCControl/ObjectTypes/Switch.md)
- [ScrewSwitch](../../Entities/NPCControl/ObjectTypes/ScrewSwitch.md)
- [StencilSwitch](../../Entities/NPCControl/ObjectTypes/StencilSwitch.md)
- [WaterSwitch](../../Entities/NPCControl/ObjectTypes/WaterSwitch.md)

The Hazard is optional and it isn't configured by the GlowTrigger, but it does directly control its BoxCollider such that even though the Hazards will technically default to a `Spikes` type, GlowTrigger manages it such that the BoxCollider's center is offscreen when the MainManager.`player` is shielding. It's effectively behaving as if it was a `WalkableSpikes`, but through the GlowTrigger and everything else in the Hazards behaves as if it was a `Spikes`. It's possible to override this by having the GameObject have its own Hazards preattached.

This component has many configurations available, here are all the public fields:

- `parent`: The associated NPCControl with this component. It is meant to be set by the NPCControl adding the component and its `hit` value determines if the glow is active or not. If it is not associated with any NPCControl, this is null. It is not meant to be set via the inspector, just be accessible to the games. If `targetentityid` isn't negative, this is overriden to be the `npcdata` of the map entity whose id is `targetentityid`
- `refreshdelay`: The amount of LateUpdate cycles to skip per glow refresh
- `targetentityid`: If this isn't negative, the value of `parent` is overriden to be the `npcdata` of the map entity whose id is `targetentityid`. Its `hit` value determines if the glow is active or not
- `flagid`: If this isn't negative, a [flags](../../Flags%20arrays/flags.md) slot that determines if the glow is active or not
- `materialid`: The index of the materials array to change the color and the `_Emission` property of each `glowparts` elements when refreshing the glow
- `glowparts`: The array of MeshRenderer whose materials[`materialid`\].color and `_Emission` color property will change when refreshing the glow. If this is null or empty, Start will override it to be a single element being the first child MeshRenderer
- `glowspeed`: The MainManager.TieFramerate parameter to use as the lerp factor when changing the `glowparts` materials's color and `_Emission` color property
- `eleccd`: If `electime` isn't 0.0, this is the amount of frames left before the Hazards's BoxCollider becomes active. This isn't meant to be set via the inspector, but it is exposed to the game specifically for [event](../../Enums%20and%20IDs/Events.md) 95 (actuating a switch in on the first room of the Honey Factory) to control for animation reasons
- `getactivecolorfromstart`: If this is true, the `activecolor` value won't be used. Instead, the values of `glowparts[0]` materials's color and `_Emission` color property will be used as the `activecolor`
- `force`: If this is true, it forces the glow to always be active even if none of the requirements are met which are any of the following:
    - `parent`'s `hit` value is true
    - `flagid` check passes
    - `flagvar` check passes
- `nosound`: This should NEVER be true because it will lead to unexpected behaviors. It was supposed to disable a functionality involving an AudioSource when the hazard feature is enabled, but not all the components properly checks for this and still assumes that the AudioSource exists
- `invert`: If this is true, reverse the activity of the glow where if it was supposed to be active, it will be inactive and vice versa. NOTE: `force` being true takes precedence over this
- `flagvar`: A vector2 that expresses a [flagvar](../../Flags%20arrays/flagvar.md) check that determines whether or not the glow is active. If the x component is negative, this featrue is disabled. The check involves flagvar slot x being at least the value of the y component
- `electime`: The amount of frames that needs to elapse until the Hazards's BoxCollider becomes active. If this is 0.0, the Hazards feature is disabled. NOTE: It is invalid to have a negative value and doing so will lead to unexpected behaviors where the Hazards will be created, but won't be used in any practical way
- `elecstay`: If `electime` isn't 0.0, this is the amount of frames that the Hazards's BoxCollider stays active once `eleccd` became 0.0
- `deactivatedcolor`: The color to use as the destination of the lerp when the glow is inactive which will affect the values of all `glowparts` materials's color and `_Emission` color property
- `elecp`: An instance of `Prefabs/Particles/Elec` childed to the GameObject, used when the hazard feature is enabled. This isn't meant to be set via the inspector, but it is exposed to the game specifically for [event](../../Enums%20and%20IDs/Events.md) 95 (actuating a switch in on the first room of the Honey Factory) to control for animation reasons 
- `activecolor`: The color to use as the destination of the lerp when the glow is active which will affect the values of all `glowparts` materials's color and `_Emission` color property. If `getactivecolorfromstart` is true, this value is ignored

Here's some additional notes about the component:

- All `glowparts` gets the `NoMapColor` tag on Start which will not cause changes to the material's color when entering an inside
- When the hazard becomes active, its BoxCollider's center is set to (0.0, 1.0, 0.0) and when it's inactive or if the MainManager.`player` has a `shield`, it is set to offscreen at (0.0, 999.0, 0.0)
- When `eleccd` becomes lower than 100.0 (there's less than 100.0 frames left before the hazard becomes active) and the glow is active, the glow refresh logic is overriden to blink the color to pure yellow and pure red periodically until `eleccd` becomes 0.0 or lower. This also involves the `Audio/Sounds/Alarm2` being played periodically using this component's own AudioSource
- When the hazard is active, `elecp` plays and `Audio/Sounds/ShockLoop` is played on loop using this component's own AudioSource. It will only stop playing on the next cycle where `eleccd` gets set to `electime`
- When the hazard becomes inactive, `elecp` stops playing
- Any sound playing as a result of the hazard will be done with a volume of MainManager.`soundvolume` (reduced by half if MainManager.FreePlayer without fly is false)
