# DoBehaviour
This is a method that takes in an ActionBehaviors (as ref which allows to override it) and the corresponding frequency which despite its name serve as a general float argument. It is only called as part of the [Update](Update.md) event and it is responsible for setting up and executing the applicable behavior.

## Before behavior execution
<!-- First, if the behavior we are dealing with is `Disguise`, `DisguiseOnce` or `DisguiseOnceJumpForward`:
- TempSpin is called on the entity which sets its `spin` to (0.0, 20.0, 0.0) and then zerored out in 0.2 seconds
- The `disguisecooldown` is set to 120.0
- The entity `overridehright` is set to false
- The `disguiseobj` is disabled -->

<!-- Before the main behavior specific logic starts, the entity's `digging` is set to true if the behavior we are dealing with is `WanderUnderground` or `ChargeAttackUnderground` while the `behaviorroutine` isn't in progress. It is set to false otherwise. -->

## Behavior execution
From there, the appropriate ActionBehavior occurs, check the logic of them to learn more.

## After behavior execution
After the behavior logic, entity `ccol` center is set to (0.0, its `ccol` height / 2.0, 0.0).

Finally, if the entity has a defined [animid](../../Enums%20and%20IDs/AnimIDs.md), it isn't an `item` and it's not a `fixedentity`, the entity `rigid` rotation gets frozen and on top of this, if it had zero velocity, all movement except the y ones are frozen too.