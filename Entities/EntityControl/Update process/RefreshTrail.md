# RefreshTrail

RefreshTrail is a method called by [LateUpdate](Unity%20events/LateUpdate.md) that manages the `trail` feature. Since this feature only implicates rendering, a high level explanation is provided rather than a series of steps.

When `trail` is turned on, the method will create the internal `traildata` if they weren't created beforehand. A TrailData is a very simple struct containing 3 arrays (one for the sprite renderers of the copies, one for their time left to render  in frames and another for their position) and 2 fields (one setting the delay in frames between the copies and another saying the last trails id to use in the arrays).

The trails object are created as child of the root object of the entity with the name "trailX" where X is the array index. The game sets the array size to a fixed 5 so this will create trail0 through trail4. They are set to render the same sprite as the entity, but half transparent and a little bit backward. with a delay of 5 frames. The delay and copies's time fields are decreased by framestep each time this method is called so they act as proper cooldowns.

The way the trail rendering works is these objects are enabled, until their time expires. They will not get reenabled again until the delay expires for the entire thing, The key thing that makes this effect works is once the `traildata` is initialised the first time, RefreshTrail will only deal with the rendering of a single copy at the time, the one in the id field.

This means that at the start when everything is initialised already, only the first copy at index 0 is enabled and positioned to where the entity is, only slightly forward before incrementing the id. It is only when the amount of frames in the delay field expires that the copy at index 1 has the same happen and so on. This means that as the entity would move, the copies progressively gets enabled, but the first ones will "lag behind" because they render at position the entity reached frames ago. Each copy will render for the amount of frames in the time field, but they won't get repositioned and enabled until the amount of frames in the delay field has expired.

The game fixes both of these cooldowns to be a delay of 5 frames and a time for each copies of 20 frames. This gives a trail effect of 5 copies of the sprite rendered 5 frames apart each for 20 frames. It does mean an overprision of the amount, but this covers framestep misshaps.

The trail can be reset to its starting state manually by calling `ResetTrail` which will expire all the hang times and reset the current copy index to 0. It only works if the `traildata` has already been initialised.
