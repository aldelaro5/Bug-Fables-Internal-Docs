# DoCommand
This coroutine contains all the logic to process any action commands which are commands to be performed during a battle action. Its runtime is tracked by `doingaction` being true which allows the caller to yield until it goes to false when it's done.

```cs
private IEnumerator DoCommand(float timer, ActionCommands commandtype, float[] data)
```

## Parameters

- `timer`: A parameter to send to the action command. Its specific meaning varies depending on the command.
- `commandtype`: The [ActionCommands](ActionCommands.md) to process
- `data`: General purpose array that controls the action commands

## Procedure
There are 4 parts to the coroutine, 2 of them depends on the commandtype.

### MainManager.`mashcommandalt` override
If MainManager.`mashcommandalt` is true (meaning the game settings has Sequential Keys set), it overrides some parameters sent to the coroutine:

- timer: If it's 0 or lower, it's overriden to 300.0. NOTE: while the `timer` gets overriden regardless of the commandtype, in practice, this will only affect [TappingKey](Action%20commands/TappingKey.md) assuming correct usage because it is the only action command that supports a negative `timer` value as being valid. It does however mean that if a negative `timer` value is incorrectly sent, this logic will apply
- commandtype: If it's [TappingKey](Action%20commands/TappingKey.md) or [RandomTappingBar](Action%20commands/RandomTappingBar.md), it is overriden to [SequentialKeys](Action%20commands/SequentialKeys.md)
- data: This can be overriden depending on the commandtype (not overriden if it isn't in the list):
    - [TappingKey](Action%20commands/TappingKey.md): overriden to a new array with one element being `data[9]` if it exists or the value 6.0 if it doesn't.
    - [RandomTappingBar](Action%20commands/RandomTappingBar.md): overriden to a new array with 2 elements being 6.0 and 1.0 respectively

### Setup
This starts by having `doingaction` set to true and `commandsucess` set to false which signals to the rest of the battle system that an action command is in progress and its result has yet to be determined, but is assumed false for now.

Some local variables are also initialised here which can be used by the specific commandtype:

- internaldata: set to a new array of float of one element
- initialtimer: set to the timer parameter
- infinite: set to true only if timer is exactly -1.0 (false otherwise)
- letters: set to null (this is a TextMesh array)

What follows depends on the specific commandtype. Consult the [ActionCommands](ActionCommands.md)'s indvidual pages to learn more. This phase involves initialising action commands related fields.

### Execution
This phase entire depends on the specific commandtype. Consult the [ActionCommands](ActionCommands.md)'s indvidual pages to learn more. This phase involves the actual execution of the action command. When it is done, the command is finished and a result has been determined which will be stored in `commandsuccess`.

### Cleanup
No matter the command type, this phase occurs after its execution:

- A frame is yielded
- FinishAction is called which does the following:
    - `actionroutine` is stopped if it was present and set to null (this is usually this coroutine's instance)
    - All `commandsprites` that exists are destroyed
    - All `buttons` that exists are destroyed
    - `doingaction` is set to false which signal to the rest of the battle system that there's no longer an action command in progress

## Known issues with [GetBlock](Battle%20flow/GetBlock.md)
While action commands are used for player actions, some enemy actions makes use of it, but in an unexpected fashion that is worth to explain. When this happens, DoCommand gets started by [DoAction](Battle%20flow/Action%20coroutines/DoAction.md) during the [enemy phase](Battle%20flow/Main%20turn%20life%20cycle.md#enemy-phase).

Due to how Unity manages coroutines, for a given frame, this will be the order they will get to execute their part:

1. [GetBlock](Battle%20flow/GetBlock.md) (it is always first because Update always run before coroutines)
2. [DoAction](Battle%20flow/Action%20coroutines/DoAction.md) (it is started first)
3. DoCommand (it is started second)

Please note that this execution order assumes that both coroutines are set to execute on that frame (for example, both could yield a frame making them scheduled to run on the next one). The coroutines are in starting order because both are defined in the same script (BattleControl) and on the same instance (MainManager.`battle`).

This order is very problematic when started by DoAction because of how a typical action would flow. Here's how a typical timeline of events would proceed:

1. Update starts DoAction with the enemy party member, this puts the battle in an [uncontrolled flow](Battle%20flow/Update%20flows/Uncontrolled%20flow.md)
2. While the action is ongoing, GetBlock runs regularly meaning `blockcooldown` and `caninputcooldown` updates as they normally do with the blocking logic
3. Eventually, DoAction starts DoCommand. This will cause GetBlock to have its logic locked because `doingaction` goes to true and that makes GetBlock not perform any blocking logic during the duration of DoCommand. Effectively, `blockcooldown` and `caninputcooldown` are frozen: they will not change and report the same state than the frame DoCommand was called (GetBlock was called right before DoAction started DoCommand on this frame)
4. The action command executes, but GetBlock cannot executes its regular blocking logic while it's ongoing
5. Eventually, DoCommand is done and sets `doingaction` to false which unlocks GetBlock's logic. However, because DoCommand is the last coroutine in the ordering, this is the last interesting event that happens in this frame.
6. A new frame starts. GetBlock goes first, but because it's unlocked now, it will perform a stray blocking state update which has effects on the regular block timing as explained below
7. DoAction goes after and it is unblocked too due to `doingaction` going to false. If [DoDamage](Damage%20pipeline/DoDamage.md) gets called before the next yield (which is what commonly happens), then the blocking result will be locked in and nothing can influence it anymore as it will be used in the damage pipeline.

Step 6 in this timeline is crucial: it effectively splits the timing window of the block into 2 distinct ones:

1. The first are the ~18 frames (~3.0 for a super block) leading up to the moment DoCommand is called
2. The second is the exact frame that GetBlock gets called while DoCommand just finished the frame before

The first window is essentially the regular one, but with 1 frame removed because this frame is now in a distinct window that occurs AFTER the action command rather than right before. Due to `caninputcooldown` however, it is not possible to attempt to hit both windows: if the player tries to hit the first window and less than ~36 frames passes until DoCommand starts, the anti spam feature will prevent any attempt to hit the second window. In other words: the player may attempt the first OR the second window, but not both (if they do, the first window will be the effective results).
