# SetAnim

Animations on entity are setup like a state machine controlled by a single integer: the [Entity](../../Entity.md)'s [animstate](animstate.md). Setting the animstate is requesting the game to change the animation of an entity whenever SetAnim occurs, usually during the entity's [LateUpdate](../Update%20process/Unity%20events/LateUpdate.md). It being on [LateUpdate](../Update%20process/Unity%20events/LateUpdate.md) allows coroutines such as [SetText](../../../SetText/SetText.md) to change it and see the render immediately when yielding before the end of the frame.

There are 2 overloads of SetAnim, but one only makes the second parameter optional which gives us the following method that end up being used:

````cs
public void SetAnim(string args, bool force)
````

The `args` parameter is explained in the [animstate > Animation arguments](animstate.md#animation-arguments) section and the `force` parameter allows to override prechecks that could have the animation not transition. Its default value is false (an override is defined without this parameter).

The actual clip that will be played can change under specific conditions. They are (in order):

* if `inice` or `hasiceanim`, the `i` argument is appended to `args`
* The `u` argument is removed from `args` if `force` is true (NOTE: this argument is not used in the game so this effectively does nothing)
* If the new [animstate](animstate.md) is 30 (`FakeHurt`), it is overriden to 11 (`Hurt`). 
  * On top of this, if `flyinganim`, the `f` argument is prepanded to `args` if it didn't had it already.

This method is called at specific places (but notably in the entity's LateUpdate) whenever an [animstate](animstate.md) update is required. In which case, this set the entity's `laststate` and it will ask Unity to change the animation clips using CrossFadeInFixedTime at the new clip's name with the time being the entity's `animspeed`. After, [UpdateAnimSpecific](AnimSpecific.md#updateanimspecific) is called.

However, unless `force` is true, it is possible to not have any of this happen. The conditions to have all of this happen are:

* The computed clip's name is the different than the last one in `laststate` OR
* The entity is the player (the one with a PlayerControl) OR
* The entity has the `PFollower` tag

If none of these are true and `force` is false, then the method will effectively not do anything.
