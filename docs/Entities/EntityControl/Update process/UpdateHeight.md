# UpdateHeight
This update method is called by [LateUpdate](Unity%20events/LateUpdate.md). It will adjust the `spritetransform`'s local position unless `overrideheight` is true.

First, prechecks are done if the adjustment is needed which only happens if the entity is not the player AND that it's not an `npcdata` of type Object.

From there, 3 outcomes are possible (only one happens, they are checked in order):

* `height` is higher than 0.1: the position is adjusted by taking the existing position in x and z, but having the y be `height` + Sin(Time.time * `bobrange`) * `bobspeed` and the resulting vector has `extraoffset` (by default zero) added to it. This will oscillate the entity up and down.
* It is not an `item`: the position is set to the existing one, but with a y of 0.0 and the resulting vector gets `extraoffset` (by default zero) added to it
* Since it's an item with the `height` being too small, nothing happens on this cycle.
