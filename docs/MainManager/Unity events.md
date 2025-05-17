# Unity events

```cs
private void Start()
```
The first method from the game that gets called by Unity. This is due to MainManager being one of the only component attached to the main scene of the game on the main camera GameObject meaning once the engine starts the game, this will be called first (this is also due to the fact MainManager has high execution priority).

It is possible this method gets called multiple times. The first call is detected by `basicload` being false which will additionally causes InputIO.StartUp to run. On Steam, this method creates the SteamManager and calls Init on Steamworks.NET, but the logic is platform specific.

Overall, Start aims to initialise core fields to sane defaults, set the current culture to `en-US` (enforces save files (de)serialisation) and to start a LoadEverything coroutine which does most of the startup work.

```cs
private void Update()
```
The main global update method of the game. It only does anything if `basicload` is true (LoadEverything completed).

Here is a summary of the tasks it handles in order:

- Handles dialogue mode SetText updates to process textbox inputs as well as prompts, numberpompts and letterprompts inputs
- Handles setting the Time.timeScale when cooking when the player requests to fast forward the cutscene TODO: detail the conditions for this
- Handles every ItemList inputs updates and handles most of their processing
- Calls GetJoystick

```cs
private void LateUpdate()
```
TODO: very complex, might need to be documented separately

```cs
private void FixedUpdate()
```
If `basicload` is true (LoadEverything ran to completion), the following happens:

- RefreshCamera is called
- LoopMusic is called
