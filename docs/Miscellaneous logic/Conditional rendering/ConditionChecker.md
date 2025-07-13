# ConditionChecker
A component meant to be attached on any GameObject via Unity.

This is a general purpose component to essentially disable or move offscreen a GameObject that doesn't fufill certain conditions regarding [flags](../../Flags%20arrays/flags.md) slots and [regionalsflags](../../Flags%20arrays/Regionalflags.md). It implements this using MainManager.[CheckIfCanExist](../../MainManager/Methods/Game%20utilities.md#condition-checking) which is used throughout the game and it is the system [NPCControl](../../Entities/NPCControl/NPCControl.md) also uses for disabling themselves when their flags requirements aren't met. The purpose of using this component is it allows to have the same logic, but done completely standalone from any GameObject in Unity. It has a Start where most of the logic happen and a LateUpdate that may move the GameObject offscreen after the fact.

It has several public fields to configure its behavior:

- `requires`: The require parameter to send to MainManager.CheckIfCanExist which is a list of flags slots that must be true for the GameObject to exist (can be empty or have its first element be negative to have no required flag slots)
- `limit`: The limit parameter to send to MainManager.CheckIfCanExist which is a list of flags slots where any among them must be false for the GameObject to exist (can be empty or have its first element be negative to have no limit flag slots)
- `regionID`: The regionalflag parameter to send to MainManager.CheckIfCanExist which is a regionalflag slot of the current area that must be false for the GameObject to exist (can be negative to have no regionalflag slot requirement)
- `spriteflagchange`: A flag slot that if true on Start, it will cause the GameObject's SpriteRenderer (if it exists)'s sprite to change to `spritetochange`. If the value is negative, this feature is disabled
- `activepos`: The local position (or position if `worldpos` is true) to set the GameObject on LateUpdate if it fails the MainManager.CheckIfCanExist requirements. If the magnitude of this is 0.1 or below, LateUpdate checks effectively are disabled
- `startpos` (not editable via the inspector): This contains the GameObject position it had on Start, but it is exposed programmatically (meant for reading)
- `startangle` (not editable via the inspector): This contains the GameObject angles it had on Start, but it is exposed programmatically (meant for reading)
- `spritetochange`: The sprite to change the GameObject's SpriteRenderer to if the `spriteflagchange` check is fufilled and the feature is enabled
- `dontdelete`: If true, the GameObject won't be disabled on Start if it fails to meet the MainManager.CheckIfCanExist requirements
- `worldpos`: This determines what the `activepos` will do if the MainManager.CheckIfCanExist requirements during LateUpdate. If it's true, the GameObject's position is set to `activepos` and if it's false, the GameObject's local position is set to `activepos`
- `pauseactive`: If this is true, the MainManager.CheckIfCanExist requirements are never disabled on LateUpdate, even when the game is paused after 20.0 frames

Essentially, there's 2 parts divided in the Start and LateUpdate:

- Start does 2 things:
    - If `dontdelete` is false, MainManager.CheckIfCanExist is called (this feature is disabled otherwise). If it succeeds, nothing happens, but if it fails, what happens depends on if the GameObject where if it has an NPCControl, its `entity`.`iskill` is set to true which will effectively cause NPCControl to disable it. If it's not an NPCControl, it's disabled and moved offscreen at 9999.0 in y with the Animator component destroyed if it exists
    - If `spriteflagchange` isn't negative and the GameObject has a SpriteRenderer, its sprite is set to `spritetochange`
- LateUpdate only manages one thing if `activepos`'s magnitude is above 0.1 (otherwise, this feature is disabled). It involves the GameObject to `activepos` (by position or local position depending on `worldpos`) if another MainManager.CheckIfCanExist check fails. This is only checked every 5 LateUpdate cycles, but after 20.0 frames, checks are disabled if the game is paused (unless `pauseactive` is true in which case checks happens even if the game is paused). It's essentially a way to move the GameObject out of the way after the fact if it's not destroyed by then
