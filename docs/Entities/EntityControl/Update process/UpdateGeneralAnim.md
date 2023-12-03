# UpdateGeneralAnim
This update method is called by [LateUpdate](Unity%20events/LateUpdate.md). It handles updates to the `sprite`'s sortingorder, the `line` if applicable and `extrasnims`.

The sorting order of the `sprite` is set to the viewport point of the `transform`'s z * 1000.0 which ensures the order respects the actual depths as seen by the camera.

If there is a `line`, the positions of it are set to be from the own `line`'s transform to the second position in the LineRenderer's position array.

Finally, if there are `extraanims`, all their speed are set to the `anim`'s speed if they didn't match already.
