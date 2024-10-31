# EntityOnly
An EntityOnly is a GameObject with a `EntityOnly` tag that contains a collider. All these colliders are stored in `entityonly` on SetPlayerColliders which is only invoked by the first LateUpdate (`latestart`) in 0.2 seconds. If the GameObject contains multiple colliders, only the first one will be stored. It is intended for the GameObject to only feature a single collider.

These colliders have special properties:

- Their static and dynamic frictions are always 0.0
- They are ignored by all `playerdata` entities's colliders
- They are ignored by all `tempfollowers`. NOTE: This only includes the list initially generated on the first LateUpdate (`latestart`), but it won't include any followers added afterwards
- They are ignored by any [Beemerang](../Entities/NPCControl/ObjectTypes/Beemerang.md) NPCControl's `scol` and entity's `ccol`
- They ignored by the player's `icicle`'s BoxCollider

Essentially, the intent of these colliders is to only apply to every NPCControl except for a `Beemerang`. Other key colliders that the player can control are ignored so the player can pass through them. It's mainly intended for example to restrict passage of certain moving map entities.
