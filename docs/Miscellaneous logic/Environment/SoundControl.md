# SoundControl
A component meant to be attached on any GameObject via Unity as long as the GameObject has an AudioSource component (it will be collected on the component's Start).

It will cause the AudioSource's volume to be set to 0.0 on Start until the first LateUpdate that instance.`globalcooldown` expired where it will be set to `startvolume` * MainManager.`soundvolume` (or multiplied by MainManager.`musicvolume` if `musicvolume` is true). Both the `startvolume` and the `musicvolume` bool value are configurable via the inspector.

This is essentially a way for any GameObject producing sounds to get their volume scaled by the in game volume settings from the [config file](../../External%20data%20format/Config%20File.md). On its own, this volume never changes after the first applicable LateUpdate, but if the settings changes during gameplay, MapControl.RefreshSoundVolume is called which will manually update the volume of all SoundControl in the scene to reflect the new settings.
