# RefreshEXP
This method refreshes the rendered EXP orbs visually involving the `expholder`. It is mostly called by [Update](../Battle%20flow/Update.md).

- If the `expholder` doesn't exist, it is created as a new GameObject named `expholder` childed to the `GUICamera` with a tag of `DelAftBtl` (meaning it will get destroyed on [ReturnToOverworld](../Battle%20flow/Terminal%20coroutines/ReturnToOverworld.md))
- All existing `bigexporbs` and `smallexporbs` are destroyed
- `bigexporbs` is recreated with length `expreward` / 10 floored and each element is created as a `Prefabs/Objects/ExpOrbGUI` instance childed to the `expholder` with a scale of (0.35, 0.35, 0.35) and a local position of (8.5 - index * 0.65, -4.26, 10.0)
- `smallexporbs` is recreated with length `expreward` - (`expreward` / 10 floored) * 10 and each element is created as a `Prefabs/Objects/ExpOrbGUI` instance childed to the `expholder` with a scale of (0.25, 0.25, 0.25) and a local position of (8.5 - index * 0.5, -4.9, 10.0)
- `oldexp` is set to `expreward` which prevents the calling of this method by [Update](../Battle%20flow/Update.md) until `expreward` changes
