# UpdateStatusIcons

This update method is called on [LateUpdate](Unity%20events/LateUpdate.md). It handles the management of `statusicons`, `statuscooldown` and `statusid` for cycling the status icons display. This method is only applicable when `statusicons` is present and not empty.

This method may do one of 3 possible things (checked in order):

* If we are currently in a battle and we are selecting Chompy's action or it's currently the enemy's phase or we are in a battle event, then all the `statusicons` are disabled if any weren't with the `statuscooldown` reset to 0.0
* If `statuscooldown` hasn't expired yet, it is decreased by framestep and nothing happens after
* On the other hand, if it has expired, then `statusid` is incremented (with wrap around to 0 if needed). From there, all `statusicons` are disabled except for the one with the index `statusid` which basically cycles the icon shown each time the `statuscooldown` expires. After, `statuscooldown` is set to 60.0

Basically, this method will cycle every `statusicon` each second unless specific battle conditions mandates to not show any of them.
