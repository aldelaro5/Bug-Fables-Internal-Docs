Seems to be a function that is called right at the end of the [dialogue cleanup phase](../Life%20Cycle/dialogue%20cleanup%20phase.md) of [SetText](../SetText.md). It seems to handle very specific logic with [flags](../../Flags%20arrays/flags.md):

* if flags 347 (start of ch6) is true, sets the corresponding badgeshop's all bought flags to true (from 587 to 589) if the corresponding shop has 0 medals in stock. NOTE: There's a third one???
* If flags 470 (Examined the view of Termite Capitol for the first time at the Forsaken Lands) is true and the Termite Kingdom (NEED TO RECHECK!) discovery isn't unlocked, unlock it (why?)
* if flags 351 (Talked to the person in front of the Termacade for the first time) is true and the Termacade (NEED TO RECHECK!) discovery isn't unlocked, unlock it.
* if flags 605 (Completed the Lost Books quest) is true and the Lost Book quests isn't in the completed array of boardquests, complete it.
* if the map isn't null, sets map.currentline to -1
* Check the achievements completion.
