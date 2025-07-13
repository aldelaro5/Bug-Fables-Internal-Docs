# LilypadEffects
A component meant to be attached on any GameObject via Unity as long as the GameObject exists in a map in the `WildGrasslands` area.

This is in practice an UNUSED component because while it does feature some logic on Start that would instantiate `Prefabs/Particles/LilypadMove` particles and configure them, it only happens when the `self` field is false. While the field is configurable in the inspector, it never has a value of false except in a prefab that is itself UNUSED.
