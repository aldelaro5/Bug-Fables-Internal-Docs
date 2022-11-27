* If [Transfer](../Commands/Individual%20commands/Transfer.md) has a value and `caller` isn't null, set the caller.interactcd to 15.0
* Sets the [tailtarget](../Notable%20local%20variable/tailtarget.md) to stop talking if it isn't null
* Sets [waitinput](../Global%20vars%20used/waitinput.md) to true
* Sets `skiptext` to false
* Yield a frame as long as [waitinput](../Global%20vars%20used/waitinput.md) is true and [End](../Commands/Individual%20commands/End.md) is false
* Stops looping the bleep if it was playing and fade its sound.
* Free all the letter slots of the [textholder](../Notable%20local%20variable/textholder.md)
* Destroys the [textholder](../Notable%20local%20variable/textholder.md)
* If `tempevent` is false and we are not in an event, sets `minipause` to false
* Release the [message](../Global%20vars%20used/message.md) lock.
* if `cameraoffset` is effectively not Vector3.Zero, restores the `Camoffset` to its local value
* `overridefollower` is restored to `tempoverf`
* If the [textbox](../Notable%20local%20variable/textbox.md) isn't null, hide it with a shrink animation and Destroys the [textbox](../Notable%20local%20variable/textbox.md) in 1 second.
* If the player isn't null and we are not in an event
  * Restores the player rigid body constraints to `tcons`
  * Sets the `actioncooldown` to 20.0
  * Stops ignoring every collisions between the player's ccol and each entities's ccol, npcdata.pusher, npcdata.scol and npcdata.boxcol from `returnentityccol` (npcdata is null checked)
* Releases the [waitinput](../Global%20vars%20used/waitinput.md) lock
* If we are not in an event and battle is null, call [EndOfMessage](../Notable%20Methods/EndOfMessage.md)
