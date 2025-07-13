# FollowerLite
A component meant to be programmatically added to an existing GameObject and have the SetUp method called immediately upon attaching it:

```cs
public void SetUp(float dist, Transform following, float spd)
```
On Start, the GameObject is set to layer 9 (`Follower`), a RigidBody component gets added with rotation frozen and a BoxCollider component gets added with a center y of 0.5.

Every 3 LateUpdate including the first one where `following` isn't null, the following happens:

- If `following` is disabled, the GameObject is disabled too
- If the Vector3.Distance between the GameObject and `following` is greater than 5.0, the GameObject position is set to `following` position
- The GameObject is set to be in range of `following` if Vector3.Distance between the GameObject and `following` is greater than `dist`

At the end of every LateUpdate, if the GameObject is in range of the `following`, the GameObject position is set to a lerp between its position and `following` position with a factor of TieFramerate(`speed`).

This component is essentially a lighter version of the EntityControl's follow system, but can be attached to any GameObject. It is only used during the UpdateAnimSpecific of a `Ruffian` animid for its ball and chain to follow them.
