# DoBehaviour
This is a method that takes in an [ActionBehaviors](../ActionBehaviors.md) (as ref which allows to override it) and the corresponding frequency which despite its name serve as a general float argument. It is only called as part of the [Update](Update.md) event and it is responsible for setting up and executing the applicable behavior.

## Before behavior execution
Some disguise behavior specific logic occurs here. See [Disguise section](../ActionBehaviors/Disguise.md#dobehavior-behavior-no-longer-exists) for more information.

## Behavior execution
From there, the appropriate [ActionBehavior](../ActionBehaviors.md) occurs, check the logic of them to learn more.

## After behavior execution
After the behavior logic, entity.`ccol` center is set to (0.0, entity.`ccol` height / 2.0, 0.0).

If the entity has a defined [animid](../../Enums%20and%20IDs/AnimIDs.md), it isn't an [item entity](../../EntityControl/Item%20entity.md) and it's not a `fixedentity`, the entity.`rigid` rotation gets frozen and on top of this, if it had zero velocity, all movement except the y ones are frozen too.