# Update and FixedUpdate
This pages talks about the Update and FixedUpdate Unity events of MainManager.

## Update
MainManager.Update is the main global update method of the game. It only does anything if `basicload` is true (LoadEverything completed).

Here is a summary of the tasks it handles in order:

- Handles dialogue mode SetText updates to process textbox inputs as well as prompts, numberpompts and letterprompts inputs
- Handles setting the Time.timeScale when cooking when the player requests to fast forward the cutscene TODO: detail the conditions for this
- Handles every ItemList inputs updates and handles most of their processing
- Calls GetJoystick

## FixedUpdate
MainManager.FixedUpdate is much simpler in comparision.

If `basicload` is true ([LoadEverything](Boot%20and%20reset%20process.md#loadeverything-part-12) ran to completion), the following happens:

- RefreshCamera is called
- LoopMusic is called
