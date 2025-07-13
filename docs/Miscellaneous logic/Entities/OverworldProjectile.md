# OverworldProjectile
A component meant to be added programmatically by calling the NewProjectile method:

```cs
public static OverworldProjectile NewProjectile(NPCControl parent, int spriteindex, Vector3 startpos, Vector3 target, Vector3 spin, Vector3 angle, Vector3 size, string particleondeath, float arc, float time, float shadowsize)
```
It will create the GameObject with the component on it.

This component is specific to the ShootProjectile ActionBehavior or related behavioirs, check the documentation to learn more.
