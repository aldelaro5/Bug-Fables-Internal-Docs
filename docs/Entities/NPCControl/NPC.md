# NPC
An NPC is an entity that supports [interactions](Interaction.md) and [ActionBehaviors](ActionBehaviors.md) with a dialogue system via [GetDialogue](Notable%20methods/GetDialogue.md) and emotes via [CheckEmoteFlag](Notable%20methods/CheckEmoteFlag.md).

## Data arrays
- `dialogues`: Check the [GetDialogue](Notable%20methods/GetDialogue.md) documentation to learn more
- `emoticonflag`: Check the [CheckEmoteFlag](Notable%20methods/CheckEmoteFlag.md) documentation to learn more

## Start / SetUp
The `radius` is incremented by 0.35 if it was less than 1.75.

## Update (Inactive)
For every 3 frames, if the entity is in a [forcemove](../EntityControl/EntityControl%20Methods.md#forcemove) without any [SetPath](ActionBehaviors/SetPath.md) or [StealhAI](ActionBehaviors/StealthAI.md) behavior, [StopForceMove](../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity.

## LateUpdate (Non `dummy`)
We decides if we should lock/unlock the entity `rigid` and call [StopForceMove](../EntityControl/EntityControl%20Methods.md#stopforcemove) only when the entity isn't `dead`, `iskill`, `deathcoroutine` isn't in progress and `activeonpause` is false. 

It depends on `inevent` and whether or not the `insideid` matches the current one:

- If we aren't `inevent`: 
    - If the `originalid` isn't -1 (the [animid](../../Enums%20and%20IDs/AnimIDs.md) isn't `None`), we ensure the entity `rigid` is locked via [LockRigid](../EntityControl/EntityControl%20Methods.md#lockrigid) if the `insideid` doesn't match the current one and unlocked if it does match. This is only performed when the current value of the entity `rigid.isKinematic` doesn't match the reality of the `insideid` matches or not.
    - If entity is in a `forcemove` and the `insideid` doesn't match the current one, [StopForceMove](../EntityControl/EntityControl%20Methods.md#stopforcemove) is called
- If we are:
    - If the entity isn't a `fixedentity`, the `originalid` isn't -1 (the [animid](../../Enums%20and%20IDs/AnimIDs.md) isn't `None`) and it's not in a `forcemove`, [LockRigid(false, false)](../EntityControl/EntityControl%20Methods.md#lockrigid) is called if the `insideid` matches the current one
    - Otherwise, [StopForceMove](../EntityControl/EntityControl%20Methods.md#stopforcemove) is called instead

## OnTriggerEnter
Only 2 things are handled here: the `specialinteract` if it's not `None` and some handling for the [StealthAI](ActionBehaviors/StealthAI.md) behavior.

The `specialinteract` part does the following if the other collider tag is `BeetleDash` (or `BeetleHorn` if the `specialinteract` is `AnyHorn`):

- Set `interactedwithhit` to true
- Calls [Interact](Notable%20methods/Interact.md) with null arguments
- Set `interactedwithhit` to false in 1 second

As for the `StealthAI` part, check its documentation to learn more.