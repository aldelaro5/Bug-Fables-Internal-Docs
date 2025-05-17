## BoardQuest

```cs
public static void AddQuest(int id)
```
Calls ChangeBoardQuest(`id`, 0) which essentially "moves" the quest to the open board, but in this case, it more adds it from nowhere to the open board. This is used in the AddQuest SetText command and it is part of CheckQuests when the quest's requirements are fufilled.

```cs
public static void CheckQuests()
```
Check every `questchecks` elements whose first requirements isn't 0 (no requirements) and whose quest id doesn't appear on any `boardquests`. For each of them, if all of the corresponding requirements are fufilled, AddQuest is called with the matching quest id.

A `questchecks` requirement is defined by the following (the value 0 is valid, but it can't be the first requirement in the list since that will count as no requirements):

- Greater than 0: The flag slot of the requirement number needs to be true
- 0 or below: The area whose id is the absolute value of the requirement number needs to have its corresponding `librarystuff[4]` flag set to true (meaning the area id was added to the world map)

```cs
private static void UpdateQuestBoard()
```
TODO: Document the QuestBoard handling separately as it seems complex

```cs
private void CloseQuestBoard()
```
TODO: Document the QuestBoard handling separately as it seems complex

```cs
private static int[] GetQuestsBoard(int type)
```
TODO: document the questboard stuff separately as it gets complex

```cs
public static bool HasQuest(int questid)
```
Returns true if `questid` appears in any `boardquests`.

```cs
public static IEnumerator OpenQuestBoard(EntityControl caretaker, NPCControl caller)
```
TODO: document quest board separately since this is complex

```cs
public static void ChangeBoardQuest(int id)
public static void ChangeBoardQuest(int id, int type)
```
If `boardquests[type]` doesn't contain the quest id `id`, the following happens:

- 0 (no quest) is removed from `boardquests[type]`
- `id` is added to `boardquests[type]`
- If `type` is 0 (open quests) and `id` is above 0 (not a no quest), flag slot 2 is set to false (indicates that new unchecked open quests are available)

For the overload without a `type` parameter, the value defaults to 0 (open quests).

```cs
private static string BoardString()
```
Returns a `,` seprated list of all `instance.boardquests[0]` quest ids in a string.

```cs
public static void CompleteQuest(int id)
```
TODO: document quest board separately since this is complex

```cs
private static int[] GetOverralQuests()
```
Return an array cointaining non zero elements in `boardquests[1]` (taken quests) as they appear followed by all non zero elements in `boardquests[2]` (completed quests), but each elements is a negative number of the value.
