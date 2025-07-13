# ShakeHorn
A component meant to be attached on any GameObject via Unity, but it requires the following to function:

- Any trigger collider on the GameObject
- Any ParticleSystem in any children of the GameObject (optional, if not present, particles functionality won't work)

On Start, the GameObject gets some modifications:

- isStatic set to false
- If there's no RigidBody, one gets added without gravity in kinematic with position frozen

The component only does something on OnTriggerEnter in any of the 2 conditions:

- The other collider has the `BeetleHorn` or `BeetleDash` tag (`Beetle`'s overworld abilities). If this is the case, it will cause a call to MainManager.HitPart to the other collider's postion + 1.0 in y
- `playermoveshake` is true and the other collider is the MainManager.`player`

If any condition happens, a coroutine starts (stopped if it was currently ongoing) that will perform some visual effects on the GameObject to rotate it in a sporadic manner to simulate a shake effect. Here's the details:

- If the `sound` AudioClip is isn't null, MainManager.PlaySoundAt is called to play its name at 1.0 volume from the GameObject position
- If there was a child ParticleSystem detected on Start, Emit is called on it with a count of `emission`
- Over the course of 61.0 frames, the angles of the GameObject each frames changes to be their initial angles from Start + `axis` * Sin(Time.time * 30.0) * (1.0 - X) where X is a number from 0.0 to 1.0 that increases as it gets closer to 61.0 frames
