* If `listredirect` is null
  - Set the [tailtarget](../../Notable%20local%20variable/tailtarget.md) to be talking
  - Set `inlist` to be false
  - If the player is not null, sets its npc to a new list
  - CONTINUE
  * If `listredirect` isn't -2 or -3
    * if `listredirect` isn't -1
      * Override the input string with `OrganizeLines`(|[blank](../../Commands/Individual%20commands/Blank.md)\| + GetDialogueText(`listredirect`.Value), `linebreak`.Value, [size](../../Commands/Individual%20commands/size.md).x, [fonttype](../../fonttype.md));
      * Sets the current char index to -1 which will process the first one on the next loop iteration
    * Set the [tailtarget](../../Notable%20local%20variable/tailtarget.md) to be talking
    * Set `inlist` to be false
    * If the player is not null, sets its npc to a new list
    * CONTINUE
  * Enable the `caller`'s ccol
  * Enable gravity on the `caller`'s rigid
  * Set the constraints of the `caller`'s rigid to FreezeRotation
  * Set the parent of `caller` to the current map.
  * Set the `caller`'s bounces to 0
  * Start a LateVelocity (velocity applied after a frame) to RandomItemBounce(6.0, 12.0)
  * Destroys the first child of the `caller`'s entity's sprite
  * Force unfix the `caller`'s entity
  * Set the touchcooldown of the `caller` to 70.0
  * Set the tossed of the caller to true
  * Set the timer of the caller to 600.0
  * If `listredirect` is -2
    * Set the caller's entity's animstate, itemstate and basestate to [flagvar](../../../Flags%20arrays/flagvar.md) 1
    * Remove the item from [listtype](../../../ItemList/listtype.md) list whose id is in [flagvar](../../../Flags%20arrays/flagvar.md) 1
    * Add the item to [listtype](../../../ItemList/listtype.md) list whose id is in [flagvar](../../../Flags%20arrays/flagvar.md) 0
  * Set [End](../../Commands/Individual%20commands/End.md) to true
  * Set `listredirect` to null
  * Set the `actioncooldown` to 30.0
  * If `eventtoss` is above -1, set `eventcall` to it
