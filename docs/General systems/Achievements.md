# Achvievements
Achievements are essentially what are backed by Logbook `librarystuff` entries. They have hardcoded requirements that when met, will unlock the matching logbook entry as well as perform the platform specific steps to unlock the achievement on the appropriate platform (Steam, GOG or console platforms).

## CheckAchievement
This method is what will perform the checks to see if Logbook entries conditions have been met and unlock them if that is the case. It has the following signature:

```cs
public void CheckAchievement()
```
All the conditions are hardcoded. If any are met to unlock an entry, UpdateJournal is called to unlock it which will set the `librarystuff` element to true and present the `discoverymessage` HUD element temporarilly. The unlocking only happens if the entry wasn't unlocked already. The method must be manually called and the game does it under the following circumstances:

- 0.5 seconds after the end of ReturnToOverworld after a battle that wasn't lost
- 0.5 seconds after EndEvent was called to end an event
- At the end of EndOfMessage once a SetText call in dialogue mode is about to end
- During MapControl.Start once a map gets loaded
- During PauseMenu.Start after loading a save file when pausing the game
- 0.5 seconds after PauseMenu.DestroyPause when unpausing the game

This method does nothing if the first LateUpdate of MainManager hasn't ran yet.

### Achivement requirements
Here are all the requirements that unlocks a logbook entry:

|Logbook Id|Requirements to unlock|
|---------:|----------------------|
|0|flag 610 is true (Beaten team Maki for the first time)|
|1|CrystalBerryAmmount returns at least 50 meaning `crystalbflags` contains at least 50 true slots|
|2|`badges` contains at least 120 elements|
|3|SamiraGotAll returns true meaning that PurchasedMusicAmmount is at least the amount of Musics enum values - 8 (the 8 that are excluded are documented in the music playback documentation)|
|4|instance.`partylevel` is at least 27|
|5|Quest id 11 (chapter 1 quest) is in `boardquests[2]` meaning the quest is in the completed board|
|7|`boardquests[2]`.count (amount of completed quests) is at least the amount of BoardQuests enum values - 1 (it excludes the NoQuest value)<sup>2</sup>|
|8|There are at least `librarylimit[2]` recipes entries unlocked in `librarystuff` meaning all used recipes entries slots are unlocked|
|9|There are at least `librarylimit[1]` bestiary entries unlocked in `librarystuff` meaning all used bestiary entries slots are unlocked|
|10|There are at least `librarylimit[0]` discoveries entries unlocked in `librarystuff` meaning all used discoveries entries slots are unlocked|
|11|flag 410 is true (Gave the `GBugRangerPlushie` to the child at Termite Capitol)|
|12|Quest id 12 (chapter 2 quest) is in `boardquests[2]` meaning the quest is in the completed board|
|14|Quest id 13 (chapter 3 quest) is in `boardquests[2]` meaning the quest is in the completed board|
|16|Quest id 14 (chapter 4 quest) is in `boardquests[2]` meaning the quest is in the completed board|
|18|Quest id 15 (chapter 5 quest) is in `boardquests[2]` meaning the quest is in the completed board|
|20|Quest id 16 (chapter 6 quest) is in `boardquests[2]` meaning the quest is in the completed board|
|22|Quest id 17 (chapter 7 quest) is in `boardquests[2]` meaning the quest is in the completed board|
|24|flag 341 is true (Hatched the Chomper Seed at Bee Kingdom Hive)|
|25|Quest id 30 (Vi's Request) is in `boardquests[2]` meaning the quest is in the completed board|
|26|Quest id 26 (Leif's Request) is in `boardquests[2]` meaning the quest is in the completed board|
|27|There are at least `librarylimit[3]` - 1 logbook entries unlocked in `librarystuff` meaning all used logbook entries slots except this one are unlocked<sup>1</sup>|
|28|flag 612 is true (Completed all bounties quest)|
|29|flagvar 28 (MazeGame high score) is higher than 9500 and flagvar 29 (FlappyBee high score) is higher than 4500|

1: The logbook entry isn't checked to know if this one was unlocked. Instead, flag 63 contains this information and it is set to true once this entry gets unlocked by CheckAchievement
2: Unlocking this one also sets flag 671 to true (Completed all quests)

### Discovery 36
For some reasons, this method also handles the unlocking of discovery entry 36 (Wizard Tower) despite not having anything to do with logbook or achivements. It is unlocked in the same way if it wasn't already with the condition being that flag 546 is true (examined the tower for the first time).

### Platform specific achievements
After going through all the requirements and unlocking logbook entries if applicable, InputIO.Achievement will be called on each of them that are unlocked in `librarystuff`.

This method is platform specific meaning its logic depends on the specific platform the game runs on. This is where the platform specific achievement will be unlocked if applicable as it receives the logbook entry id that needs to be unlocked or do nothing if it's already unlocked.

## Hard mode bosses logbook entries
The logbook entries 6, 13, 15, 17, 19, 21 and 23 are absent from the list above which are all the logbook entries obtained from beating a chapter boss with the `DoublePain` medal equipped or flag 614 (HARDEST) active.

This is because CheckAchievement doesn't handle these. They are all handled in specific events that matches their battle encounter events. The UpdateJournal call happens after winning the battle in the event with the `DoublePain` medal equipped or flag 614 (HARDEST) active.

Here are the events associated with these checks:

|Logbook entry Id|Event associated|
|---------------:|----------------|
|6|26 (Approaching the Ancient Mask in Snakemouth Den)|
|13|73 (Approaching Venus at Golden Hills)|
|15|99 (Talking to the Overseet in the core room at the Honey Factory)|
|17|117 (Approaching the Watcher at Ancient Castle)|
|19|137 (Approaching the Beast at Wild Swampslands)|
|21|193 (Approaching Ultimax at Rubber Prison)|
|23|200 (Approaching the Wasp King at Giant's Lair)|
