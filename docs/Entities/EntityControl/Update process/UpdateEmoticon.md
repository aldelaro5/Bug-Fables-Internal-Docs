# UpdateEmoticon
This is an update method called by [LateUpdate](Unity%20events/LateUpdate.md), but it is able to be called externally via the `Emoticon` method. This updates the `emoticon` object.

Every 10 frames (which means every 5 update cycle as [LateUpdate](Unity%20events/LateUpdate.md) only calls this method every 2 frames), the angles, position and `emoticonsprite`'s enabled are updated. The angles are set at 0.0 except for the y being the camera's y angle, the position is set to `emoticonoffset` + Vector3.up * `height` + `extraoffset` and the sprite's enabled is set to `incamera`.

Then, the method ensures the `emoticon` has a runtimeAnimatorController and if it doesn't, it's loaded from `AnimationControllers/_Misc/Emoticon/emoticon` from the root of the asset tree.

Finally, the `emoticoncooldown` expiration logic ensures. If the cooldown hasn't expired yet, it is decreased by framestep and the `emoticonid` is played immediately on the `emoticon`.

It should be noted that if the cooldown has expired, there is an unused feature where if it was set to higher than -100.0 (which is not possible normally without externally doing so), the -1 (blank) animation will play on the `emoticon` while setting `emoticoncooldown` to -101.0 to prevent from reaching this code path again. 
