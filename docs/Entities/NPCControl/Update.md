# Update
Most of this event only happens if it's not a `dummy`, but some logic happens regardless at the end of it. The logic depends on the [NPCType](NPCType.md), [ObjectTypes](Object.md#objecttypes) and [ActionBehaviors](ActionBehaviors.md). Consult each's documentation to learn more.

## Non Dummy logic
From there, the update logic depends whether the NPCControl is considered active or not. In order to be active, entity.`alwaysactive` needs to be true or all of the following needs to be true:

- The entity is `incamera`
- The `insideid` matches the current one
- From there, there are 3 cases:
    - `startlife` is less than 50 frames which makes the NPCControl active
    - The entity.`campos.z` is less than 25 when the map `limitbehavior` is false which makes the NPCControl active
    - The entity.`campos.z` is less than 15 when the map `limitbehavior` is true. In this case, the NPCControl is active only if any of the following are true:
        - The entity is currently in a [forcemove](../EntityControl/EntityControl%20Methods.md#forcemove)
        - The entity.`campos.x` is between 0.25 and 0.75 exclusive (it means it's centered enough to be within half the viewport horixontally)

If any of the above is violated, the NPCControl is considered inactive and implicates a different logic entirely.

### Active updates
There are 2 subsections to this. Up to one of them will apply with the main one taking priority, but it is possible that neither applies. No matter which subsection of the active update that applied (if any), the `touchcooldown` is updated according to the frame time if it hasn't expired yet at the end of the active update section.

#### Main active update logic
This Section only applies if any of the following are true:

- The entity is `activeinevents` while we are in a `minipause` or `inevent` and we aren't `trapped`
- The entity is `activeonpause`
- We aren't in a `pause`, `minipause`, [message](../../SetText/Notable%20states.md#message), the entity isn't an [item entity](../EntityControl/Item%20entity.md) and isn't `dead`, `iskill` and we aren't `trapped`

> If this is an [Enemy](Enemy.md), there are 3 special cases that can override the rest of this section (dizzy, frozen or about to be unfrozen). For more information, check the [Enemy](Enemy.md) NPCType documentation.

We only continue from here if we aren't `trapped`.

Further [Enemy](Enemy.md#update-active-main-logic-no-overrides) exclusive logic happens here when applicable.

If `returntoheight` is true, the entity.`initialheight` is above 0.1, the entity.`height` is less than entity.`initialheight` - 0.05 and the `disguisecooldown` hasn't expired yet, entity.`height` is set to a lerp from entity.`height` to entity.`initialheight` with a factor of the frametime / 10. This also sets the entity.`oldfly` to false if its entity.`height` is less then entity.`initialheight` - 0.2 which forces a sprite update. All of this essentially makes the entity progressively comes back to its original height if applicable smoothly.

> If this this is an [Object](Object.md), then perform the Update specific section of the [ObjectTypes](Object.md#objecttypes) which also replaces the rest of this section. Otherwise, the non object logic follows.

Further [Enemy](Enemy.md#update-active-main-logic-no-overrides) exclusive logic happens here when applicable.

Some logic specific to the [DisguiseOnce](ActionBehaviors/DisguiseOnce.md) behavior occurs here.

Finally, `hasenteredrange` is set to true and [DoBehavior](Notable%20methods/DoBehaviour.md) is called if all of the following conditions are true:

- `overridebehavior` is false
- No `behaviorroutine` is in progress
- There is at least a default behavior set in `behaviors`
- The entity isn't `dead` or `iskill` and doesn't have a `deathcoroutine` in progress
- One of the following is true:
    - This is an [Enemy](Enemy.md)
    - The entity is `alwaysactive`
    - The entity.`campos.z` is less than 20.0 (it isn't too far forward the camera)

As for which behaviors is chosen at which frequency, it depends on 2 conditions:

- Whether or not `forcebehavior` is defined. This forces the behavior to use, but not the frequency (see the second condition for how that is selected)
- Whether the player is present and its entity is `digging` or its entity.`icooldown` hasn't expired yet. If this is true, it forces the behavior AND the frequency to be the default one which effectively ignores all `inrange` behaviors in these conditions. Otherwise, the frequency depends on `inrange` and the behavior (assuming `forcebehavior` isn't defined) is also based on it.

#### Non trapped active item update logic
This subsection applies if the main one didn't and it's an [item entity](../EntityControl/Item%20entity.md) while not being `trapped`. Most cases involves logic specific to the [Item](ObjectTypes/Item.md) object, but an [item entity](../EntityControl/Item%20entity.md) used as a shelved shop item is also affected, but not in a meaninful way (only calls [StopForceMove](../EntityControl/EntityControl%20Methods.md#stopforcemove) on the 3rd cycle onward effectively making it fixed in place).

### Inative updates
This segment only applies if the entity is considered inactive which means the active update segmenets didn't apply.

If we aren't `trapped`, the absolute position is set to the entity.`lastpos`.

If applicable according to the [ActionBehaviors](ActionBehaviors.md), some adjustements happens related to the `disguiseobj` when it is present as well as the NPCControl and its entity.

For every 3 frames, if the entity is in a [forcemove](../EntityControl/EntityControl%20Methods.md#forcemove) is called on the entity except if it has a [SetPath](ActionBehaviors/SetPath.md) or [StealhAI](ActionBehaviors/StealthAI.md) behavior.

## Common logic
This segment applies regardless if it's a `dummy` or not.

In there, some specific [ActionBehaviors](ActionBehaviors.md) logic occurs if applicable.