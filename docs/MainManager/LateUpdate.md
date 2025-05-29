# LateUpdate
MainManager.LateUpdate is a very large update events, just like its [Update](Update%20and%20FixedUpdate.md#update) counterpart. However, where it differs is Update is mainly concerned about inputs and UI naviguation updates while LateUpdate handles a lot of diverse update tasks. Most of its updates impacts visual with some state consistency logic. Most of the method requires `basicload` to be true meaning [LoadEverything](Boot%20and%20reset%20process.md#loadeverything-part-12) completed.

Here's what happens during LateUpdate:

- `messagebreak` (the default linebreak of [SetText](../SetText/SetText.md) mostly used in [dialogue mode](../SetText/Dialogue%20mode.md#dialogue-mode)) is set 10.5 unless [languageid](../SetText/languageid.md) is undefined or 0 (English) where it's 9.75 instead
- `itemdescbreak` (the default linebreak of [SetText](../SetText/SetText.md) mostly used when rendering items's description box) is set 9.9 unless [languageid](../SetText/languageid.md) is undefined or 0 (English) where it's 10.5 instead
- `framestep` is set to TieFramerate(1.0). This is a convenient, but partially out of date field to get number that scales with the smoothed average framerate based on 60 FPS (meaning 1.0 is equivalent to exactly 60 FPS). It's partially out of date because it's only updated on LateUpdate, but in practice, because TieFramerate is based on smoothDeltaTime, the variation should be minimal
- At this point, the method returns abrutply if `basicload` is false which means LoadEverything isn't done yet
- UpdateOutlines is called which sets the `GlobalOutline` global Shader float to the value that reflects the outlines setting
- `GUICamera` angles are reset to Vector3.zero
- `globalcamdir` has its angles reset to (0.0, `MainCamera` y angle, 0.0). For more information, check the [camera system](../General%20systems/Camera%20system.md) documentation
- `globalcooldown` decreases by TieFramerate(1.0)
- If `playerdata` isn't null or empty, [RefreshHUD](../General%20systems/HUD.md#refreshhud) and [RefreshDiscovery](../General%20systems/HUD.md#discoverymessage) are called
- If a `texttail` exists, the [tailtarget](../SetText/Notable%20states.md#tailtarget) gets updated
- If `started` is false (this is the first LateUpdate since boot) and `player` exists (meaning the game isn't [loading a map](../MapControl/Map%20loading.md)), [DetectIgnoreSphere](../Entities/EntityControl/EntityControl%20Methods.md#detectignoresphere) is called on `player`.`entity` and `started` is set to true
- If the game is `inevent`, but not `inbattle`, both `letterbox` elements has their color set to a lerp from the existing color to Color.black with a factor of TieFramerate(0.15)
- `inputcooldown` is decreased by `framestep`
- If the `switchicon` exists, its color is set to a lerp from the existing color to Color.clear with a factor of TieFramerate(0.01)
- `stickholdx` is updates such that it is only true if the absolute value of InputIO.JoyStick(0) or InputIO.JoyStick(2) (the left/right inputs) is higher than 0.5
- `stickholdy` is updates such that it is only true if the absolute value of InputIO.JoyStick(1) or InputIO.JoyStick(3) (the up/down inputs) is higher than 0.5
- Some constraints on max numbers are enforced on 2 [flagvar](../Flags%20arrays/flagvar.md):
    - 26 (berry bank balance): Set to 10000 if it is higher
    - 27 (game token count): Set to 9999 if it is higher
- For any completed [boardquests](../General%20systems/Quest%20boards.md), they are removed from the open and taken boardquests
- If `partylevel` is 27 (the max one), `partyexp` is set to `neededexp` (this essentially makes the EXP counter looks like `X/X` where `X` was the `neededexp` value set when rank 26 was reached)
