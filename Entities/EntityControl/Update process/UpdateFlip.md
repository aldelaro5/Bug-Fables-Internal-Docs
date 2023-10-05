# UpdateFlip

This update method is called by [LateUpdate](Unity%20events/LateUpdate.md). It will update the digging states as well as the `spritetransform` angles as long as `overrideflip` is false.

The method can either perform a `digging` update or a regular flip update, but it will only do the latter when the former doesn't apply.

## Digging update

The `digging` one happens whenever `digging` is true and the entity isn't `dead`, `iskill` and `deathcoroutine` isn't in progress. In that case, a regular digging update happens when `diganim` is false (otherwise, `instdig` is set to false and the method ends). In such an update, the angles and scale of the `spritetransform` are adjusted such that the scale is the lerp between Vector3.zero and `startscale` with a factor of `digtime` / 30.0, and the angle is incremented by 15.0 on the y axis. 

Next is the `digpart` update which happens unless `nodigpart` is true which will just have the `digtime` be increased by the framestep and `instdig` being set to false. 

In that segment, if the `digtime` hasn't reached 30 frames yet, then the `digpart` 0 is initialised if it wasn't present so long as `insdig` is false and `digpart` 1 is moved offscreen if it exists followed by increasing `digtime` by framestep. The way `digpart` 0 is created involves instantiating `Prefabs/Particles/DirtFlying` rotated by -90.0 degrees on the x axis and childed to the `transform` with a position of zero. On top of this, if it has an Enemy `npcdata`, the material's renderqueue of that object is set to 3000.

Otherwise, if the `digtime` had reached 30 frames, then `digpart` 0 is put offscreen while `digpart` 1 is ensured to be initialised. The creation process is very similar than `digpart` 0 on the other `digtime` branch, but the 2 differences is that the object instantiated is at `Prefabs/Particles/Digging` and the scale is `digscale`. Finally, `instdig` is set to false.

## Regular flip update

In this update, since it's established the entity isn't digging, `digtime` is set to 0.0 and both `digpart`s are moved offscreen if they were present.

Next either `spritetransform` is rotated by `spin` if it has a non zero vector or regular flipping occurs as long as `overrideonlyflip` is false.

If we are updating, this will adjust the scale and possibly angles of the `spritetransform`. The angles are adjusted except if it's `isplayer` and we are dashing, but the scaling is always adjusted to `truescale` except in the z axis if `flip` is true where that part is 0.0 - `truescale`.z. If the angles gets adjusted, only the y axis get adjusted. It will be changed to the lerp of the current y angle to 180.0 degrees when `flip` is true or 0.0 degree when it is false using the GetFlipSpeed method to get the value. This method normally returns the `flipspeed` (which is 0.2 normally), but it can be overridden to 1.0 with an instant flip if it has an `npcdata` with a `startlife` less then 50.0. Essentially, this progressively rotates the `spritetransform` towards the desired angle creating the flip effect.
