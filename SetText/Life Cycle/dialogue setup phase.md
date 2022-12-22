* MainManager.instance.`letterprompt` is set to -1
* MainManager.`define` is initialised with a default new constructor
* Initializes the [Text advance](../Related%20Systems/Text%20advance.md) and the [Backtracking](../Related%20Systems/Backtracking.md) current dialogue and history to a clean state. `tempdiag` is set to |[size](../Commands/Individual%20commands/size.md),size.x,size.y| where the size param values comes from [size](../Commands/Individual%20commands/size.md). 
* if [languageid](../languageid.md) is `Japanese`, prepend |[size](../Commands/Individual%20commands/size.md),0.8,0.9| to the input string and set the [Speed](../Commands/Individual%20commands/Speed.md) to 0.03 (0.02 if not Japanese).
* Release [waitinput](../Global%20vars%20used/waitinput.md).
* Hide the HUD and the discovery HUD.
* Apply an `inputcooldown` of 10 frames.
* if the `caller` isn't null
  * if the caller's `interacttype` is `Shop` or `CaravanBadge` and if its shopkeeper's `dialogues[1].y` isn't 1, show the current berry count HUD, hide it otherwise.
* Set `promptpick` to -1.
* Grabs the [message](../Global%20vars%20used/message.md) and `minipause` lock.
* If the player isn't null and we are not in an event
  * Make the player face the `caller` (or the `caller`'s shopkeeper if its  `interacttype` is `Shop` or `CaravanBadge`)
  * Set `initialflip` to the player's flip.
  * Make the player stop moving.
  * Set `tcons` to the player's rigid body's constraints
  * if the `caller` isn't null, teleport following players and chompy closer to caller? (2.55 max distance, 1.5 away?). After, yield for a frame.
* If the [Camoffset](../Commands/Individual%20commands/Camoffset.md) is effectively not Vector3.Zero, add it to MainManager.camoffset
* Initialize the [textbox](../Notable%20local%20variable/textbox.md) and its [tailtarget](../Notable%20local%20variable/tailtarget.md) to [Parent](../Commands/Individual%20commands/Parent.md)
* Set the bleep and the bleep pitch to the ones coming from the entity of [Parent](../Commands/Individual%20commands/Parent.md) if it exists.
* If MainManager.playerdata isn't null, take every non null and activeInHierarchy player entity EXCEPT VI (possible errata?) and have them each stop their forcemove and set their sprite to enabled.
* if the current map isn't null, stops the forcemove of every non null `tempfollowers`.
* Setup the [GlobalCommand](../Related%20Systems/GlobalCommand.md) system if the current map isn't null and supports it.
* Overrides the [textholder](../Notable%20local%20variable/textholder.md)'s parent to be the [textbox](../Notable%20local%20variable/textbox.md)'s parent.
* Sets [textbox](../Notable%20local%20variable/textbox.md) to the [textholder](../Notable%20local%20variable/textholder.md). This allows it to be globally accessed.
* Setup the [textbox](../Notable%20local%20variable/textbox.md) by setting parent to the GUICamera, localEulerAngle to Vector3.zero and localPosition to (0.0, 3.25, 10.0)
* If the [position](../Commands/Individual%20commands/position.md) is effectively not Vector3.Zero, sets the localPosition of the [textholder](../Notable%20local%20variable/textholder.md) to it again. Otherwise, it is set to (-5.5, 0.9, 0.0)
* Sets the [textbox](../Notable%20local%20variable/textbox.md)'s localScale to Vector3.zero
* Add a `DialogueAnim` to the [textbox](../Notable%20local%20variable/textbox.md)
* Initializes the `blinker`.
* Sets the localEulerAngle of the [textholder](../Notable%20local%20variable/textholder.md) to Vector3.zero
* If the current map isn't null, call StopMovingEntities on all entities of the map.
* If the player isn't null, yield for a frame as long as 
  the switchcooldown is above 0 (meaning it is fully expired)
