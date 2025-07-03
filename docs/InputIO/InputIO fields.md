# InputIO fields
Here are all the fields declared in InputIO. All of them are static:

|Name|type|Is Private?|Description|
|---:|----|-----------|-----------|
|isconsole|bool|No|A convenience field that has the value of Application.isConsolePlatform intiially, but is never set|
|IsEditor|bool|No|A convenience field that has the value of Application.isEditor intiially, but is never set|
|keys|Keycode\[10\]|No|All the keyboard input bindings indexed by input id. Only the values contained in `bindingkeys` are allowed to be in this array|
|joykeys|Keycode\[6\]|No|All the buttons controller input bindings indexed from input id 4 to input id 9. Any values in this array is meant to be a joystick related KeyCode value|
|steam|SteamManager|Yes|The Steamworks.NET object meant to communicate with Steam's API dll. It is only present on the Steam version of the game and it is only initialised on InputIO.StartUp to the SteamManager component of a newly added GameObject named `Steam Manager` rooted to the scene|
|bindingkeys|List<KeyCode>|No|A list of values of KeyCode allowed to be in `keys`. It is initially set to a list that is never modified later and it contains all standard keys present on a full sized keyboard (including the numpad) except for any F keys and the Apple / Windows / meta keys|

## Unused fields
The following fields are never used:

|Name|type|Is Private?|Description|
|---:|----|-----------|-----------|
|neutrals|float\[10\]|Yes|All the 10 virtual axises values obtained on the last call to GetNeutrals. This is effectively unused because GetNeutrals is only called as part of the unused controller bindings window from the PauseMenu (that feature was replaced with the rebinding wizard). Since the method is never called, all `neutrals` have a value of 0.0. While they are read in GetJoystickRaw, they don't impact anything since it's used as "neutral" axis position and 0.0 is the canonical value for this|
|platform|RuntimePlatform|No|A convenience field that has the value of Application.platform intiially, but is never meant to be set|
|currentbuild|Builds|No|Indicates the current PC store platform used between STEAM, GOG and ITCH, but the value is never read or set|
