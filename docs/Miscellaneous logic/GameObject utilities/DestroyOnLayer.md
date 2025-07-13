# DestroyOnLayer
A component meant to be programmatically added to an existing GameObject and have the SetUp method called immediately upon attaching it:

```cs
public void SetUp(string deathparticle, float particletime, int targetlayer, Vector3 partoffset, Vector3 partangle)
public void SetUp(string deathparticle, float particletime, int targetlayer, Vector3 partoffset, Vector3 partangle, bool actuallydontdestroy)
```
The default value of `actuallydontdestroy` on the first overload is false.

This component will destroy or move offscreen the GameObject when an OnTriggerEnter happens where the other collider is on the `targetlayer` layer. It is destroyed if `actuallydontdestroy` is false and moved -9999.0 offscreen if it's true (more specifically, the parent is moved if it exist and the GameObject is moved if it doesn't). If `deathparticle` isn't null, when the trigger collision happens, PlayParticle is called with the particle being `deathparticle` without sound at the GameObject position + `partoffset` with an angle of `partangle` for a time of `particletime`.
