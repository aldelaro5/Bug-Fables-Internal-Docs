# Tailextra

An alias of [Tail](Tail.md) where the int form of `entity` refers to a temporary follower index instead of an [Entity id](../Entity%20id.md) (which is redundant, see remarks why).

## Syntax

(1)

````
|tailextra,entity|
````

(2)

````
|tailextra,entity,transition|
````

## Parameters

Refer to the [Tail](Tail.md) documentation, but the only changes are for the `entity` parameter if the current map isn't null (otherwise, this command exactly like [Tail](Tail.md)):

* The int form refers to a temporary follower index and it must be a valid one or an exception will be thrown.
* `this` is not a valid value and it will throw an exception if specified anyway.

Any other value for this parameter functions the exact same way than [Tail](Tail.md).

## Remarks

This command should be redundant because [Tail](Tail.md) takes an [Entity id](../Entity%20id.md) which supports temporary followers. There are only 2 possible reasons one would use this command over [Tail](Tail.md):

* There is no null check performed on the map before accessing the temporary followers array for an [Entity id](../Entity%20id.md) meaning it will throw instead of interpreting the value as a regular [Entity id](../Entity%20id.md) like this command does (which is potentially not desired so this difference may not matter anyway)
* This allows to have more than 1000 [Entity id](../Entity%20id.md) being specified on the map without the followers because it is now possible to index the desired follower independently. While this is a possible usecase, it should be noted that most commands uses [Entity id](../Entity%20id.md) as a standard meaning should this ever happen, it is likely some other command will behave incorrectly making this usecase not as relevant.

Due to this redundancy and the simplicity of just using [Tail](Tail.md), usage of this command isn't recommended: use [Tail](Tail.md) instead.

This is one of the command supported by [Testdiag](Testdiag.md).
