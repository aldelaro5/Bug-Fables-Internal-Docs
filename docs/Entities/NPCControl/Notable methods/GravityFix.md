TODO: It was never clear what was the ACTUAL issue that necessited this hacky workaround. More investigations should be done on why this was needed and if found, the proper fix should be documented because this can have unintended consequences.

# GravityFix
This coroutine was made to workaround an issue where enemies could fall under the map. It messes with the entity.`rigid` and its position several times in rather intrusive ways. It always gets started at the end of [Start](../Start.md) unless the `NGF` [Modifier](../../EntityControl/Modifiers.md) is enabled.

Here are the steps performed in order:

- A frame is yeilded
- The entity.`rigid`'s useGravity and isKinematic are saved temporarilly
- A raycast from the current position + Vector3.up is done downwards of any layer of Ground and NoDigGround. If hit with a transform, this enemy's absolute y position is set to the y of that hit point
- The current position is saved into a temporary variable
- For a minimum of 1 second, the following is done every frame so long as `minipause` is true or that `hasenteredrange` is false:
    - The entity.`rigid` gravity is disabled and placed in kinematic mode without velocity
    - This enemies's position is set to the variable saved earlier
- For an additional 5 frames, the following is performed each frame:
    - The position is again set to the variable saved earlier
    - LatePos is called on this transform with the variable saved earlier using a 0.02 delay only trying once
    - The entity.`rigid` velocity is zeroed out
- Finally, the original values of the entity.`rigid`'s useGravity and useKinematic are restored
