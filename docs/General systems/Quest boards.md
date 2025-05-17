# Quest boards
The quest board system consists of all the facilities the game needs to interact with quests. At its heart is the `boardquests` 2 dimensional array which contains all BoardQuests ids (second dimension) grouped by the board in which they appear (first dimension).

There are 3 boards that a quest can be in:

- 0: Open quests which haven't been started, but could be if they get moved to taken. They will in the overall quests listtype so they only appear in the open quests listtype which only appears as part of a QuestBoard NPC interaction
- 1: Taken quests which are started, but unfinished quests. They will in the overall quests listtype
- 2: Completed quests which are finished quests that also appears in the overall quests listtype. A quest in this board normally stays there indefinetely.

This system implies that all quests aren't visible by default because they must be added to the open board or taken board for them to be visible. This is typically done by the game as the player progresses through various events.

There is a special BoardQuest element being id 0 called the `NoQuest` element. It's a placeholder that should only appear if the board would be empty were it not for this element. This implies that any of subarrays in `boardquests` shouldn't be empty and instead, contain this one element. The system automatically manages the presence or absence of this element. It is only there for UI presentation purposes.

## AddQuest and ChangeBoardQuest
The simplest way to add a BoardQuest to the open board is via the AddQuest method:

```cs
public static void AddQuest(int id)
```
`id` is a BoardQuest id and it's meant to be called on a BoardQuest that isn't in any boards. It's used in the addquest SetText command which means it's possible to add any quests to the open boards via SetText.

The process of adding a quest to the open board actually involves a different method: ChangeBoardQuest. What AddQuest actually does is it calls ChangeBoardQuest(`id`, 0) which is what does the actual adding:

```cs
public static void ChangeBoardQuest(int id)
public static void ChangeBoardQuest(int id, int type)
```
If `boardquests[type]` doesn't contain the quest id `id`, the following happens:

- 0 (no quest) is removed from `boardquests[type]`
- `id` is added to `boardquests[type]`
- If `type` is 0 (open quests) and `id` is above 0 (not a no quest), flag slot 2 is set to false (indicates that new unchecked open quests are available)

For the overload without a `type` parameter, the value defaults to 0 (open quests).

ChangeBoardQuest is also the method used to move open quests to the taken boards when the player starts it. The process of taking a quest is documented further below this page.

## CheckQuests
Several quests in the game automatically manages their requirements before they are placed in the open board rather than being added manually. This is handled by the CheckQuests method which has the following signature:

```cs
public static void CheckQuests()
```
It involves a different multidimensional array called `questchecks`. The format of this array is documented in the QuestChecks data documentation, but basically, it contains a list of prerequesite for BoardQuests to be added to the open board. This data comes from a TextAsset which is also documented.

CheckQuests is the method that will enforce these. It will go over every `questchecks` whose quest id doesn't appear in any `boardquests` and whose first requirement isn't 0 (no requirements). If all of the requirements are fufilled, AddQuest is called with the matching BoardQuest id.

The game calls the method on MapControl.Start (which covers every map load) and towards the end of event 3 (cooking).

## Accessing the quest board
Quest boards in the game are accessed through a complex system involving NPCControl interaction, ItemList and SetText.

It starts with the QuestBoard interaction. When interacting with an NPC that has this interaction, it will load to an OpenQuestBoard coroutine:

```cs
public static IEnumerator OpenQuestBoard(EntityControl caretaker, NPCControl caller)
```
The coroutine is documented in more details inside the QuestBoard interaction documentation and it essentially sets up some essential fields for the quest board:

- instance.`boardcaller`: The caller which is the NPC that had the QuestBoard interaction. This will be used later when accessing the caretaker NPCControl.
- instance.`questboardobj`: The GameObject containing all the quest board UI. It is created here with a complex arrangement of UI objects and a container for an ItemList.
- The game enters a `minipause`

Crucially, it ultimately sets up an ItemList with listtype 14 (open quests) and calls ShowItemList which is how the open quests can be accessed through the custom `questboardobj` UI and the ItemList within it.

These listtype uses a method to generate their `listvar` options which is GetQuestBoard:

```cs
private static int[] GetQuestsBoard(int type)
```
This method is necessary because not all the BoardQuest can be listed for the respective board. Some requires the `map` to be `UndergroundBar` and some aren't listed in these listtypes at all. Check the quest board listtype group documentation for more details.

The last step the OpenQuestBoard coroutine does is call UpdateQuestBoard which is detailed in the section below.

## Quest board UI handling
Once OpenQuestBoard is done, this is where some special inputs handling kicks in in Update. It happens during the process of handling the listtype 14, 15 and 16 which are respectively the open, taken and completed listtype. By pressing the left and right inputs, the player can cycle between these listtype to change the board to consult. Doing this involves calling ResetList, adjusting `listtype` and calling ShowItemList again so the entire ItemList is remade with the new listtype.

Changing the listtype also involves the method UpdateQuestBoard:

```cs
private static void UpdateQuestBoard()
```
This is called at the end of OpenQuestBoard, but also when the player changes the listtype. Its purpose is only to visually update the quest board UI so it shows the correct tab as the current one being browsed on top of the other 2.

The entire handling of the quest board UI is now exclusively controlled by Update and the listtype which has custom rendering to render the details about each quests as the player browses the ItemList.

From there, only 2 outcomes is possible: cancelling the ItemList or confirming a quest in the open quests listtype.

### Cancelling a quest board listtype
Cancelling happens as a regular ItemList, but specifically for quest boards, it involves a CloseQuestBoard call and a SaveCameraPosition(false) call. The latter will restore the saved camera position because it's possible the NPC earlier wanted to pan the camera temporarily before presenting the quest board UI.

As for the former, it's a method specific for this situation:

```cs
private void CloseQuestBoard()
```
The goal of this method is to tear down `questboardobj` leading to its destruction. It aims to undo what OpenQuestBoard did.

It can be summed up as doing the following:

- Calls DestroyText on all children of `questboardobj` so all letter slots used are freed
- All DialogueAnim in the children of `questboardobj` gets their `shrink` set to true
- `questboardobj` is destroyed in 0.1 seconds
- `player`.`actioncooldown` is set to 20.0 frames which prevents most inputs processing
- The game exits its `minipause`
- `inlist` is set to false (the ItemList gets destroyed later with a DestroyList after returning)

### Confirming an open quest
This is only possible with the open quest list type and it is the main way an open quest can be moved to the taken board. All options are confirmable except 0 (`NoQuest`).

When this happens, CloseQuestBoard is still called (details above), and DestroyList still happens, but the difference is there's a special SetText call that happens. It happens with the caretaker of the NPC that was interacted with earlier which is another NPCControl that can handle the confirmation of taking an open quest. Check the QuestBoard interaction documentation for more details on how this works. The dialogue line to use with SetText is also defined from the NPC that was interacted with earlier. Right when the SetText call starts, flagvar 0 is set to the confirmed BoardQuest id.

What's important in that SetText call is the dialogue line is prepanded with `|questprompt|` which is a special commands that marks all prompt command in the call as quest board prompts. They behave like regular prompts, but when an option is chosen, SetText will handle it in a special way by replacing the text to go to entirely. This implies that it's assumed that the caretaker dialogue line contains a prompt command otherwise, this whole interaction won't work.

Selecting the first `option` (which is assumed to be the confirm one) will cause SetText to eventually process a activateselectedquest command as part of the new text. This command is what will move the BoardQuest contained in flagvar 0 to the taken board via a ChangeBoardQuest call and a removal of the BoardQuest from the open board.

There's a lot more details involved not mentioned here for brevety on how SetText handles this. For more detail, check the questprompt command documentation.

## Completing a quest
Completing a quest involves the CompleteQuest method:

```cs
public static void CompleteQuest(int id)
```
This method will do all needed tasks to move the BoardQuest `id` from the taken board to the completed board. Alongside the movement of the BoardQuest, here's what the method does:

- Removes any 0 (`NoQuest`) elements from the completed quest as it's guaranteed to not be empty anymore
- If the taken board became empty, 0 is added to it
- flagvar 47 is incremented (amount of completed quests excluding the main story ones)
- instance.`discoveryhud` is set to 350.0 to temporarily show the `discoverymessage` HUD element
- The 2nd, 3rd and 4th child of `discoverymessage` have their enablement changed such that only the 3rd one is enabled, the other 2 are disabled
- The `Quest` AnimationClip is played on `discoverymessage`'s Animator

## SetText interface
SetText supports multiple commands that interfaces with BoardQuest in similar ways than the methods mentioned in this page. It allows to call AddQuest or ChangeBoardQuest in various ways through these commands. One such command even allows to move all open quests to the taken board.

Check the BoardQuest section of the SetText commands documentation to learn more.

## Utilities methods
The following methods operates on BoardQuests in various useful ways.

```cs
public static bool HasQuest(int questid)
```
Returns true if `questid` appears in any `boardquests`.

```cs
private static string BoardString()
```
Returns a `,` seprated list of all `instance.boardquests[0]` quest ids in a string.

This is used as part of the activateselectedquest SetText command where the return of this is set to flagvar 11 which has no known usage.

```cs
private static int[] GetOverralQuests()
```
Return an array cointaining non zero elements in `boardquests[1]` (taken quests) as they appear followed by all non zero elements in `boardquests[2]` (completed quests), but each elements is a negative number of the value.

This is only used as part of the overall quests listtype which doesn't restrict which quests can be listed as long as it's from the taken or completed boards.
