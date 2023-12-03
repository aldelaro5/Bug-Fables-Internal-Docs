# UpdateAirAnim
This is an update method called by [LateUpdate](Unity%20events/LateUpdate.md) when applicable. All update cycles are skipped if `overridefly` or `overrideanim` is true.

The method first sees if the [animstate](../Animations/animstate.md) should be updated to `Jump` or `Fall` depending on the vertical velocity. Determining this involve verifying that the entity is in the air with `overridejump` being false. More specifically:

* `offgroundframes` is higher than 3.0
* If `isplayer`, it isn't dashing
* Not `onground`
* `height` is 0.0

After, the method prechecks if we proceed to check if the entity has landed which simply checks if it's `onground`. If it is, then there are 2 possible ways to update the [animstate](../Animations/animstate.md): from the `basestate` or the `npcdata`'s `dialogues` at `currentdialogueindex`'s z. 

The `dialogues` one is done when it applies while not in an event, the current [animstate](../Animations/animstate.md) is `Idle` and `changedstate` was not enabled externally. The only way to enable it is through [SetText](../../../SetText/SetText.md)'s [anim](../../../SetText/Individual%20commands/Anim.md) command.

The `bassestate` one applies when either the above applies, but there is no `currentdialoguesindex` or that the [animstate](../Animations/animstate.md) was at `Fall`.
