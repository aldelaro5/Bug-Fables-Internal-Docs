# FakeWall
A component meant to be attached on any GameObject via Unity, but it requires a trigger collider on the GameObject to function.

It will cause a call to [PlaySound](../../General%20systems/Sounds%20playback.md#playing-sounds) with the `Glow` sound with a pitch of 0.8 and a volume of 1.0 on OnTriggerEnter when the other GameObject is the MainManager.`player`. It will only do this on the first OnTriggerEnter or after an OnTriggerExit has been registered with MainManager.`player`.
