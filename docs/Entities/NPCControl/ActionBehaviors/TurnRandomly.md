# TurnRandomly
Stops any force move on the entity and periodically causes the entity.`flip` to toggle which turns the entity using a somewhat random time interval.

## Frequency meaning
The middle of the time interval range between turns. The time interval is random between half of the frequency value to the double of the frequency value.

## SetInitialBehavior (Only if it's the default behavior)
Sets the `actioncooldown` to the frequency.

## DoBehavior
[StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity.

Then, this behavior only takes effect when the `actioncooldown` expires and if it hasn't yet, it is decreased by the game's frametime.

When it does take effect, the entity.`flip` is toggled and the `actioncooldown` is set to a random value between half of the frequency and the double of the frequency.