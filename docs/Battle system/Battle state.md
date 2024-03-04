# Battle state
These fields describes the state of the battle. Some fields belongs to MainManager, but are mostly BattleControl related.

## BattleControl fields
This is all the BattleControl fields.

### Actions commands and blocking

|Name|Type|Public?|Description|
|----|----|---------|-----------|
|actionroutine|Coroutine|No|Tracks the progress of a [DoCommand](Action%20commands/DoCommand.md) coroutine|
|doingaction|bool|Yes|If true, an action command is in progress|
|successfulchain|int|No|The amount of sucessful inputs perfomred in a `LongRandomBar` action commands during [DoCommand](Action%20commands/DoCommand.md)|
|presskey|int|No|A general purpose input id for use in [DoCommand](Action%20commands/DoCommand.md)|
|barfill|float|No|A number between 0.0 and 1.0 that tells the ratio of an action command's bar's filled portion which is managed by [DoCommand](Action%20commands/DoCommand.md)|
|infinitecommand|bool|No|If true, action commands processed by [DoCommand](Action%20commands/DoCommand.md) will use their infinite variant TODO: define what this is|
|killinput|bool|No|Used for the `PressKey` action command that when set to true, [DoCommand](Action%20commands/DoCommand.md) will stop listening for inputs|
|commandsuccess|bool|Yes|Tells if the last action command succeeded|
|buttons|ButtonSprite\[\]|No|A general purpose array of ButtonSprites for use in [DoCommand](Action%20commands/DoCommand.md)|
|commandsprites|SpriteRenderer\[\]|No|General purpose SpriteRenderer array for use in [DoCommand](Action%20commands/DoCommand.md) or other action commands related needs|
|helpboxid|int|No|The id of the current action command whose description should be rendered in `helpbox`|
|helpbox|DialogueAnim|No|The 9box containing the action command description whose id is `helpboxid`|
|superblockedthisframe|float|No|A short lived frames cooldown that when not expired, it indicates the player very recently peformed a super block. Used in the damage pipeline and maintained by [LateUpdate](Visual%20rendering/LateUpdate.md)|
|blockcooldown|float|No|The amount of frames left that a block can be processed TODO: detail this and recheck|
|caninputcooldown|float|No|A general purpose input cooldown in amount of frames left before expiring|
|hasblocked|bool|No|If true, the damage pipeline detected the player blocked the attack with a valid block (meaning FRAMEONE rules applied). This is only used in HardSeedVenus, a coroutine specific to the `VenusBoss` [enemy](../Enums%20and%20IDs/Enemies.md)|
|overridechallengeblock|bool|Yes|Normally, standard blocks are disallowed when [flags](../Flags%20arrays/flags.md#flags) 615 and 15 are true (FRAMEONE is active while past the first tutorial fight), but if this field is true, it allows to override that behavior to allow them regardless of these conditions being fufilled or not|

### Actors information

|Name|Type|Public?|Description|
|----|----|---------|-----------|
|alldata|[BattleData](Actors%20states/BattleData.md)\[\]|Yes|All the `playerdata` followed by all the `enemydata` appended together. Set by RefreshAllData which is only called during [StartBattle](StartBattle.md)|
|enemydata|[BattleData](Actors%20states/BattleData.md)\[\]|Yes|The first enemies party member data with a length up to 4 (the extra ones are stored in `extraenemies`). Set to a new list with the same length as the sent enemyids on [StartBattle](StartBattle.md) (after it was truncated to the first 4) and then filled with the actual enemy data|
|extraenemies|List<int>|No|This contains the list of [enemy](../Enums%20and%20IDs/Enemies.md) that overflows the maximum amount of enemies allowed in `enemydata` which is 4. It is initialised as such on [StartBattle](StartBattle.md)|
|reservedata|List<BattleData>|No|The list of enemies who are no longer part of `enemydata` because they had died and were moved there by [CheckDead](Battle%20flow/Action%20coroutines/CheckDead.md) since their `deathtype` indicated they should be moved there. Doing so prevents a full destruction which allows the enemy to be revived later or to visually have a death animation without disappearing. Reset to a new list on StartBattle|
|enemyfled|bool|No|Whether at least one enemy party member fled the battle or not|
|lastaddedid|int|No|The last enemy party member index added via [AddNewEnemy](Actors%20states/AddNewEnemy.md)|
|tempslot|BattleData|No|The last enemy to be added in `enemydata` according to [NewEnemy](Actors%20states/NewEnemy.md) if the sent animation value isn't `None`|
|deadenemypos|List<Vector3>|No|The list of the positions of previously dead enemies as found by [CheckDead](Battle%20flow/Action%20coroutines/CheckDead.md). It is only used by CheckDead in case `extraenemies` needs to be summoned in the place of the dead ones at the same positions they were before|
|deadmembers|int\[\]|No|The array of player party members indexes whose `hp` is 0 or below as returned by GetDeadParty. Used during [RevivePlayer](Actors%20states/RevivePlayer.md) to decide whether to resucitate the player party member|
|partyentities|[EntityControl](../Entities/EntityControl/EntityControl.md)\[\]|No|Set to all the `playerdata` battleentity on [StartBattle](StartBattle.md)|

### Targetting

|Name|Type|Public?|Description|
|----|----|---------|-----------|
|avaliabletargets|BattleData\[\]|No|An ephemeral array of actors to track possible targets for an action which is frequently set by calling [GetAvailableTargets](Actors%20states/GetAvaliableTargets.md)|
|forceattack|int|No|The player party member's [animid](../Enums%20and%20IDs/AnimIDs.md) index that will be forced to be returned during [GetRandomAvaliablePlayer](Actors%20states/GetRandomAvaliablePlayer.md)|
|playertargetID|int|No|The player party member index whom is currently targetted by the enemy. If it's -1, the enemy isn't targetting anyone|
|playertargetentity|EntityControl|No|An actor to use for targetting purposes (NOTE: despite the name, it can be an enemy party member as enemies can sometimes target other enemy party members)|
|targetedenemy|int|No|Indicates the enemy party member index that `Beetle` will target when using his basic attack or the `HeavyStrike` skill during [DoAction](Battle%20flow/Action%20coroutines/DoAction.md)|
|target|int|No|The index of the selected target when selecting a player or enemy|

### Chompy informations

|Name|Type|Public?|Description|
|----|----|---------|-----------|
|chompy|[EntityControl](../Entities/EntityControl/EntityControl.md)|No|If [flags](../Flags%20arrays/flags.md) 402 is true(Chompy is with Team Snakemouth), this is initialised to a new entity created via [CreateNewEntity](../Entities/EntityControl/EntityControl%20Creation.md#createnewentity) with name `chompy` with the `ChompyChan` [animid](../Enums%20and%20IDs/AnimIDs.md) on [StartBattle](StartBattle.md)|
|chompyattack|Coroutine|Yes|The coroutine of [Chompy](Battle%20flow/Action%20coroutines/Chompy.md) if it's in progress (null if it's not)|
|chompyaction|bool|No|Tells if [Chompy](Battle%20flow/Action%20coroutines/Chompy.md) is in progress or not|
|chompyattacked|bool|No|Tells if [Chompy](Battle%20flow/Action%20coroutines/Chompy.md) has completed during the player phase when applicable|
|coptions|List<int>|No|The list of action options available during [Chompy](Battle%20flow/Action%20coroutines/Chompy.md): 0 is basic attack, 1 is do nothing, 2 is the ribbon specific attack and 3 is change ribbon|
|chompyoption|int|No|The last chosen `coption` the last time [Chompy](Battle%20flow/Action%20coroutines/Chompy.md) was called|
|chompylock|bool|No|If true, it prevents [Chompy](Battle%20flow/Action%20coroutines/Chompy.md) to be a part of the [player phase](Battle%20flow/Update.md#player-phase) even if it would normally be allowed. This is only used during [DoAction](Battle%20flow/Action%20coroutines/DoAction.md) specifically for the `Centipede` [enemy](../Enums%20and%20IDs/Enemies.md)|

### AI party informations

|Name|Type|Public?|Description|
|----|----|---------|-----------|
|aiparty|[EntityControl](../Entities/EntityControl/EntityControl.md)|No|The entity that was created as part of AddAI if one was created|
|aiattacked|bool|No|Tells if [AIAttack](Battle%20flow/Action%20coroutines/AIAttack.md) has completed during the player phase when applicable|

### Battle flow

|Name|Type|Public?|Description|
|----|----|---------|-----------|
|action|bool|Yes|If true, it means an action coroutine controls the battle flow making it an [uncontrolled flow](Battle%20flow/Update.md#uncontrolled-flow). Set to false on [StartBattle](StartBattle.md)|
|inevent|bool|Yes|Tells if an [EventDialogue](Battle%20flow/EventDialogue.md) is in progress or not. If one is, we enter an [uncontrolled flow](Battle%20flow/Update.md#uncontrolled-flow)|
|calleventnext|int|No|If not negative, the [EventDialogue](Battle%20flow/EventDialogue.md) whose id is this value will be done on the next [unctrolled flow](Battle%20flow/Update.md#uncontrolled-flow). This is only used in conjuction with an actor's `eventonfall` when it triggers|
|cancelupdate|bool|Yes|Whether updates are disabled which only happen when some kind of terminal event occurs that changes the flow to a [terminal flow](Battle%20flow/Update.md#terminal-flow). Set to false on [StartBattle](StartBattle.md)|
|alreadyending|bool|Yes|Tells when the battle is about to end via [GameOver](Battle%20flow/Terminal%20coroutines/GameOver.md) or [AddExperience](Battle%20flow/Terminal%20coroutines/AddExperience.md). Set to false on [StartBattle](StartBattle.md)|
|enemy|bool|Yes|Whether we're in the [enemy phase](Battle%20flow/Update.md#enemies-phase) or not (meaning we're in the [player phase](Battle%20flow/Update.md#player-phase)) of the turn of a [controlled flow](Battle%20flow/Update.md#controlled-flow) or processing a [hitaction](Battle%20flow/Update.md#enemies-hitaction). Set to false on [StartBattle](StartBattle.md)|
|mainturn|Coroutine|No|The [AdvanceMainTurn](Battle%20flow/Action%20coroutines/AdvanceMainTurn.md) coroutine if one is in progress (null if it's not)|
|delprojs|DelayedProjectileData\[\]|Yes|The delayed projectiles currently active|
|turns|int|No|The amount of completed main turns advanced by [AdvanceMainTurn](Battle%20flow/Action%20coroutines/AdvanceMainTurn.md). Set to 0 on [StartBattle](StartBattle.md)|
|demomode|bool|No|If true, indicates the battle operates in tutorial mode TODO: clearly define what this is|
|gameover|Coroutine|No|The current [GameOver](Battle%20flow/Terminal%20coroutines/GameOver.md) coroutine is one is in progress (null if it's not). Set to null on [StartBattle](StartBattle.md)|
|halfload|bool|Yes|Whether or not [StartBattle](StartBattle.md) roughly got done half of the starting process. This is set to true right after calling SetLastTurns and it's used by the game to yield until this goes to true|
|avaliableplayers|int|No|The amount of players that are considered free by [GetFreePlayerAmmount](Actors%20states/GetFreePlayerAmmount.md). Set to the current amount on [StartBattle](StartBattle.md)|
|currentturn|int|Yes|Determine the `playerdata` index currently selected for an action. -1 means no one is selected yet, being below `playerdata` length means that player index is selected and being at length or above means all players have taken their actions and the player phase is over. Set to -1 on [StartBattle](StartBattle.md)|
|lastturns|int\[\]|No|The state of the player index selection cycle which starts with an array of length being the amount of free players - 1. Advancing it means either assigning the first free slot (-1) to the player being selected or shift the elements such that it falls on the latest while the oldest is removed|
|receivedrelay|bool\[\]|No|An array indicating which player index got relayed to via [Relay](Battle%20flow/Action%20coroutines/Relay.md) which allows the `tiredpart` to be rendered whenever `tired` gets above 0 during [UpdateAnim](Visual%20rendering/UpdateAnim.md)|
|actedthisturn|bool|No|Tells if [PlayerTurn](Battle%20flow/PlayerTurn.md) was called at least once during the turn implying at least one player could act. Set back to false on [AdvanceMainTurn](Battle%20flow/Action%20coroutines/AdvanceMainTurn.md)|
|noaction|int|No|The amount of main turns in a row where `actedthisturn` was false. If it reaches 5 without game over, the [inactive failsafe](Battle%20flow/Action%20coroutines/AdvanceMainTurn.md#inaction-failsafe) triggers in [AdvanceMainTurn](Battle%20flow/Action%20coroutines/AdvanceMainTurn.md)|
|attackedally|int|No|The `playerdata` index that was attacked while the `FavoriteOne` [medal](../Enums%20and%20IDs/Medal.md) was equipped on them which will cause it to take effect during [AdvanceMainTurn](Battle%20flow/Action%20coroutines/AdvanceMainTurn.md). If this medal isn't applicable, the value is -1|
|firststrike|bool|No|Whether or not the enemy is currently acting as part of the enemy party getting the starting advantage during [StartBattle](StartBattle.md)|
|summonnewenemy|bool|No|If true, it means that the game is in the process of summoning an enemy from the `extraenemies` to `enemydata` via [SummonEnemy](Actors%20states/SummonEnemy.md), called duing [CheckDead](Battle%20flow/Action%20coroutines/CheckDead.md). This prevents SummonEnemy to set `checkingdead` to null once completed to not interfere with the ongoing CheckDead and it also allows CheckDead to wait the summon is over|
|selfsacrifice|bool|No|If true, it indicates that the enemy party member killed themselves during their [DoAction](Battle%20flow/Action%20coroutines/DoAction.md)|
|eatenkill|bool|Yes|If true, it indicates that a player party member was killed by a `Pitcher` [enemy](../Enums%20and%20IDs/Enemies.md) by draining their HP|
|lastdamage|int|No|The last final amount of damage done at the end of [DoDamage](Damage%20pipeline/DoDamage.md)|
|damagethisturn|int|No|The amount of damage the player party inflicted in the current main turn. If it gets higher than [flagvar](../Flags%20arrays/flagvar.md) 41 (highest damage in one turn), the flagvar value is set to it before resetting in [AdvanceMainTurn](Battle%20flow/Action%20coroutines/AdvanceMainTurn.md)|
|charmcooldown|int|No|The amount of main turns that needs to pass for [UseCharm](Actors%20states/UseCharm.md) to process the next charm even if one is available and it would have been processed otherwise. Set to a random integer between 3 and 7 inclusive after a charm has been processed and decremented on [AdvanceMainTurn](Battle%20flow/Action%20coroutines/AdvanceMainTurn.md)|

### UI naviguation

|Name|Type|Public?|Description|
|----|----|---------|-----------|
|currentaction|[Pick](Player%20UI/Pick.md)|Yes|The current menu being naviguated by the player. Set to `BaseAction` on [StartBattle](StartBattle.md)|
|currentchoice|[Actions](Player%20UI/Actions.md)|Yes|The current action being selected on the `BaseAction` menu. Set to `Attack` on [StartBattle](StartBattle.md)|
|itemarea|[AttackArea](AttackArea.md)|No|Tells what actors are selectable to perform the current action|
|maxoptions|int|No|The amount of available `option` on the main vine menu|
|lastoption|int|No|The last selected `option` on the main vine menu|
|selecteditem|int|No|The selected `listvar` option from an [ItemList](../ItemList/ItemList.md), set by [SetItem](Player%20UI/SetItem.md)|
|vineoption|int|No|Equivalent to `maxoption`, but for the main vine menu value|
|option|int|Yes|The current vine menu option. 0 is attack, 1 is skills, 2 is items, 3 is strategies and 4 is turn relay. Set to 0 on [StartBattle](StartBattle.md). This can also be the index of an enemy or player target being selected. NOTE: the vine order is reversed in game as left goes up and right goes down in the option index with wrap around|
|excludeself|bool|Yes|If true, it will cause the `currentturn` player to not be selectable in a `currentaction` of `SelectPlayer` in [GetChoiceInput](Player%20UI/GetChoiceInput.md)|
|tempskill|int|Yes|The [skill](../Enums%20and%20IDs/Skills.md) id that was confirmed from skills selection in [SetItem](Player%20UI/SetItem.md)|
|lastskill|int|No|The last [skill](../Enums%20and%20IDs/Skills.md) id used, set to `selecteditem` before its usage when confirmed in [GetChoiceInput](Player%20UI/GetChoiceInput.md)|
|lastaction|int|No|The last [action](Player%20UI/Actions.md) chosen by a player party member. Set to `option` on [AdvanceMainTurn](Battle%20flow/Action%20coroutines/AdvanceMainTurn.md) which keeps the last selected vine menu options since the last one chosen on the new turn|
|idletimer|int|No|The amount of frames since the player idled which is used in the [EXP counter update](Battle%20flow/Update.md#exp-counter-updates)|
|turncooldown|float|No|The amount of [FixedUpdate](Visual%20rendering/FixedUpdate.md) cycles left before [GetChoiceInput](Player%20UI/GetChoiceInput.md) calls are allowed during [PlayerTurn](Battle%20flow/PlayerTurn.md). Set to 5.0 in [SetItem](Player%20UI/SetItem.md) and [CancelList](Player%20UI/CancelList.md)|


### UI rendering

|Name|Type|Public?|Description|
|----|----|---------|-----------|
|vinebase|Transform|No|A GameObject named `Base` childed to `choicevine`|
|choicevine|Transform|No|A GameObject named `Vine` that is the root of the vines UI objects, childed to the `battlemap` and initialised on CreateVine as part of [PlayerTurn](Battle%20flow/PlayerTurn.md)|
|vineicons|SpriteRenderer\[\]|No|The vine icons of the vine menu, childs of `vinebase`|
|cursor|Transform|No|The selection leaf cursor. Initialised on CreateCursor which is only called on [StartBattle](StartBattle.md)|
|switchicon|Transform|No|The parent of the UI objects that composes the switch icon which is a GameObject named `switchicon` childed to the `GUICamera`. Initialised on [StartBattle](StartBattle.md)|
|fronticon|SpriteRenderer|No|The SpriteRenderer of the UI object named `attackicon` childed to the `GUICamera` that corresponds to the icon the front party member has. Initialised on [StartBattle](StartBattle.md)|
|hexpcounter|Transform|No|The EXP counter shown when idling for 200+ frames when `currentaction` is `BaseAction`. Created on the first [controlled update flow](Battle%20flow/Update.md#controlled-flow)|
|expholder|Transform|No|A GameObject named `expholder` childed to the `GUICamera` with tag `DelAftBtl` that holds the EXP orbs cumulated in the battle for visual rendering|
|lvicon|Transform|No|The EXP icon sprite used in the UI during [AddExperience](Battle%20flow/Terminal%20coroutines/AddExperience.md)|
|damcounters|List<Transform>|No|All of the currently rendered damage counters maintained by [CounterAnimation](Visual%20rendering/ShowDamageCounter.md#counteranimation), a sub coroutine of [ShowDamageCounter](Visual%20rendering/ShowDamageCounter.md). Set to a new list on StartBattle|
|hideenemyhp|bool|Yes|If true, the `hpbar` of every enemy party members's battleentity gets disabled on [RefreshEnemyHP](Visual%20rendering/RefreshEnemyHP.md)|
|actiontext|SpriteRenderer|No|The UI element for the game to indicate the current action being selected which is the SpriteRenderer of a new GameObject named `ActionText` childed to the `GUICamera`. Initialised on [StartBattle](StartBattle.md)|
|smallexporbs|Transform\[\]|No|The array of smaller orbs (one per 1 EXP below 10 remaining) which are instances of `Prefabs/Objects/ExpOrbGUI` rendered as child of the `expholder`|
|bigexporbs|Transform\[\]|No|The array of larger orbs (one per 10 EXP) which are instances of `Prefabs/Objects/ExpOrbGUI` rendered as child of the `expholder`|
|longcancel|bool|No|Whether the Cancel input is considered wider for rendering or not. Set to the return of InputIO.LongButton(5) on [StartBattle](StartBattle.md). This is involved during the rendering of the `switchicon`|
|cancelb|ButtonSprite|No|A ButtonSprite of the cancel input for use in [UpdateText](Visual%20rendering/UpdateText.md)|

### Battle setup information

|Name|Type|Public?|Description|
|----|----|---------|-----------|
|battlemap|GameObject|Yes|The parent of all the battles objects notably the battle map prefab. This is a game object named `Battle` created on [StartBattle](StartBattle.md)|
|caller|[NPCControl](../Entities/NPCControl/NPCControl.md)|No|The [Enemy](../Entities/NPCControl/Enemy.md) NPCControl whose encounters caused this battle. This is set to the calledfrom value sent to [StartBattle](StartBattle.md). If it's null, no encounter caused this battle|
|sadv|int|No|The starting advantage value. Set to the sent adv value of [StartBattle](StartBattle.md), but adv is set to this field if it's a retry|
|sdata|StartUpData|No|The [StartUpData](StartUpData.md) stored during [StartBattle](StartBattle.md) when `saveddata` is false. Restored on StartBattle for a retry|
|saveddata|bool|No|Whether the `sdata` have been saved and are ready for restoration. Set to true at the very end of [StartBattle](StartBattle.md)|
|canflee|bool|Yes|Whether fleeing is allowed. Set to the canescape value sent to [StartBattle](StartBattle.md)|

### Camera

|Name|Type|Public?|Description|
|----|----|---------|-----------|
|campos|Vector3|No|The `camtargetpos` that will be set when [SetDefaultCamera](Visual%20rendering/SetDefaultCamera.md) is called, normally Vector3.zero|
|camoffset|Vector3|No|The `camoffset` that will be set when [SetDefaultCamera](Visual%20rendering/SetDefaultCamera.md) is called, normally MainManager.`battlecampos`|
|oldcamoffset|Vector3|No|Set to instance.`camoffset` on [StartBattle](StartBattle.md) when it's not a retry. This is used for restore later on [ReturnToOverWorld](Battle%20flow/Terminal%20coroutines/ReturnToOverworld.md)|
|oldcamrotation|Vector3|No|Set to instance.`camangleoffset` on [StartBattle](StartBattle.md) when it's not a retry. This is used for restore later on [ReturnToOverWorld](Battle%20flow/Terminal%20coroutines/ReturnToOverworld.md)|
|oldcamspeed|float|No|Saved to instance.`camspeed` on [StartBattle](StartBattle.md) when it's not a retry. This is used for restore later on [ReturnToOverWorld](Battle%20flow/Terminal%20coroutines/ReturnToOverworld.md)|

### EXP and berries

|Name|Type|Public?|Description|
|----|----|---------|-----------|
|estimatedexp|int|No|The sum of all the calculated `enemydata`'s `exp`|
|expreward|int|No|The amount of EXP cumulated in the course of the battle that will be granted if the battle is won. Set to 0 on [StartBattle](StartBattle.md)|
|oldexp|int|No|The last value of `expreward` observed since the last [RefreshEXP](Visual%20rendering/RefreshEXP.md)|
|leveled|bool|No|Indicates that [AddExperience](Battle%20flow/Terminal%20coroutines/AddExperience.md) detected a rank up situation|
|moneyreward|int|No|The amount of berries that will be dropped after the battle. Set to 0 on [StartBattle](StartBattle.md)|

### Attack and flow modifiers

|Name|Type|Public?|Description|
|----|----|---------|-----------|
|dontusecharge|bool|No|If true, the `charge` of the enemy party member will not be reset to 0 on [EndEnemyTurn](Battle%20flow/EndEnemyTurn.md)|
|nonphyscal|bool|No|If true, indicate to the damage pipeline that the damages aren't physical which affects the effects of the `FrostBite`, `SpikeBod` and `PoisonTouch` [medal](../Enums%20and%20IDs/Medal.md#)|
|nolifesteal|bool|Yes|If true, disables the effects of the `LifeSteal` [medal](../Enums%20and%20IDs/Medal.md) during the damage pipeline|
|lockmmatter|bool|Yes|If true, prevents the `MiracleMatter` [medal](../Enums%20and%20IDs/Medal.md) to take effect. Set to false on StartBattle|

### Misc coroutines tracking

|Name|Type|Public?|Description|
|----|----|---------|-----------|
|checkingdead|Coroutine|Yes|Store diverse coroutines to track their progression (NOTE: despite the name, it's not just for [CheckDead](Battle%20flow/Action%20coroutines/CheckDead.md))|
|enemybounce|Coroutine\[\]|No|A general purpose coroutines array for use in [DoAction](Battle%20flow/Action%20coroutines/DoAction.md) that is made to track EnemyBounce and SummonArtifact|
|tryenemyheal|Coroutine|No|Tracks the progress of a TryHealEnemyItem and EnemyFlee coroutines|
|spitout|Coroutine|No|The SpitOut coroutine if one is in progress which is a coroutine used by the `Pitcher` [enemy](../Enums%20and%20IDs/Enemies.md). It is null if it's not in progress|
|gottaspit|bool|Yes|When calling SpitOut (a coroutine specific to the `Pitcher` [enemy](../Enums%20and%20IDs/Enemies.md)), this needs to be set to false whenever `Pitcher` should spit out the player party member trapped within them|
|startdrop|bool|Yes|When a [Drop](../Entities/EntityControl/Notable%20methods/Drop.md#drop) calls occurs, the coroutine will eventually wait for this field to become true so it can proceed with the drops|

### Visual effects

|Name|Type|Public?|Description|
|----|----|---------|-----------|
|wordroutine|Coroutine|No|Tracks the progress of a [ShowSuccessWord](Visual%20rendering/ShowSuccessWord.md) coroutine|
|commandword|SpriteRenderer|No|The word SpriteRenderer used in [ShowSuccessWord](Visual%20rendering/ShowSuccessWord.md)|
|combo|int|No|The amount of consecutive successful action commands performed which influences visual and audio effects informing of the input successes|
|defaultcounteroffset|Vector3|No|Always set to (0.0, 1.25, 0.0) for use in [DoDamage](Damage%20pipeline/DoDamage.md) TODO: find out what this represents|
|counterspriteindex|int\[\]|No|Indicates the `guisprites` indexes of the different counter used in [ShowDamageCounter](Visual%20rendering/ShowDamageCounter.md). This is always {8, 40, 101}|
|partymiddle|Vector3|No|Always set to (-4.5, 0.0, 0.0) for use in [DoAction](Battle%20flow/Action%20coroutines/DoAction.md) TODO: find out what this represents|
|tiredpart|Transform\[\]|No|Set to a new array of the transform of instances of `Prefabs/Particles/Tired` on [StartBattle](StartBattle.md) belonging to each `playerdata` battleentity (each is childed to their battleentity) The length is thus the same than `playerdata`|
|tskybox|Material|No|The RenderSettings.skybox saved on [StartBattle](StartBattle.md). This is used for restoring later in case of a retry|
|fogdist|float|No|The value of RenderSettings.fogEndDistance saved on [StartBattle](StartBattle.md) if the current one needs to change. Restored from this field during [ReturnToOverworld](Battle%20flow/Terminal%20coroutines/ReturnToOverworld.md) if it had a value above 0.0|
|dimmer|SpriteRenderer|No|A giant black colored sprite that covers everything behind the transition during load. Initialised on StartBattle after the fade in transition and disabled with a clear color just before the fade out one|
|charmdance|Sprite\[\]|No|The sprites Charmy uses during [UseCharm](Actors%20states/UseCharm.md). Always set to sprite 98 and 99 of `Sprites/Entities/moth0` on [StartBattle](StartBattle.md)|

### Music

|Name|Type|Public?|Description|
|----|----|---------|-----------|
|overworldmusic|AudioClip|No|On [StartBattle](StartBattle.md), this is set to MainManager.`music[0]`.clip if the map.`music[0]` exists and map.`musicid` is not negative. This is used for restoring the music on [ReturnToOverworld](Battle%20flow/Terminal%20coroutines/ReturnToOverworld.md)|
|overmusic|float|No|If `overworldmusic` was saved, the time of that music is saved on this field if MainManager.`keepmusicafterbattle` is true. This is also used for restore on [ReturnToOverworld](Battle%20flow/Terminal%20coroutines/ReturnToOverworld.md)|
|keepmusic|bool|Yes|Towards the end of [ReturnToOverworld](Battle%20flow/Terminal%20coroutines/ReturnToOverworld.md) when instance.`inevent` is true, it is possible to skip the default behavior of fading the current music to silence by having this field set to true before ReturnToOverworld ends which will leave the current music playing instead|

### Spying

|Name|Type|Public?|Description|
|----|----|---------|-----------|
|disablespy|bool|Yes|If true, spying is disabled and cannot be performed on any enemy|
|scopeequipped|bool|No|Whether or not the `Spy Specs` [medal](../Enums%20and%20IDs/Medal.md) is equipped. Set as such on [StartBattle](StartBattle.md)|

### General purpose

|Name|Type|Public?|Description|
|----|----|---------|-----------|
|tempdata|int|Yes|A general purpose integer for use in [DoAction](Battle%20flow/Action%20coroutines/DoAction.md) meant to be set externally. This is only used for the `Spuder` [enemy](../Enums%20and%20IDs/Enemies.md) TODO: recheck, it might have been moved to a different enemy during dev|
|extraentities|[EntityControl](../Entities/EntityControl/EntityControl.md)\[\]|Yes|A general purpose array of entities. This is only used when a `VenusBoss` [enemy](../Enums%20and%20IDs/Enemies.md) is involved|

### Unused fields
These fields are never referenced or never used in any meaningful ways.

|Name|Type|Public?|Description|
|----|----|---------|-----------|
|weakenemyfound|bool|No|UNUSED, this is only set to false at the start of [DoAction](Battle%20flow/Action%20coroutines/DoAction.md) and to true on TryHealEnemyItem when a heal occured, but the field is never read making it unused|
|actionid|int|No|UNUSED, this is only read during the damage pipeline, but never written to making it unused|
|oldmusicchannel|int|No|UNUSED|
|counter|int|No|UNUSED|
|selection|int|No|UNUSED|
|specialdefeat|bool|Yes|UNUSED|
|firstaction|bool|Yes|UNUSED|
|currententity|EntityControl|No|UNUSED|
|oldcamtarget|Transform|No|UNUSED, Set to instance.`camtarget` on [StartBattle](StartBattle.md) when it's not a retry|
|guicooldown|float|No|UNUSED (there is logic in Update implicating it was supposed to be a cooldown that exhausts only during an Update controlled battle, but it is never practically used anywhere else making it practically unused)|
|partypointer|int\[\]|Yes|UNUSED, This field is maintained to be the same value as `partypointer`, but by indexing `playerdata` instead. This means that `playerdata[X].pointer` is the same than `partypointer[X]` where `X` is a valid `playerdata` index. In practice, this field is only written into making it unused|

## MainManager fields
These fields belongs to MainManager, but they are mostly related to BattleControl's state and either have influences on the battle or they are reporting the result of the battle.

### Player party

|Name|Type|Static?|Description|
|----|----|---------|-----------|
|playerdata|BattleData\[\]|No|The battle data of the players. The game keeps track of them constantly even outside of battle, but [StartBattle](StartBattle.md) resets their fields to a default state|
|tp|int|No|The amount of TP the player party has (usually clamped from 0 to `maxtp`)|
|tpt|int|No|The displayed amount of TP the player party has in the HUD|
|maxtp|int|No|The maximum amount of TP the player can have (this is `basetp` + the medals effects from [ApplyBadges](ApplyBadges.md))|
|basetp|int|No|The base maximum amount of TP the player can have (this is normally 10 + all the bonuses applied from [ApplyStatBonus](ApplyStatBonus.md))|
|bp|int|No|The amount of MP the player party has, clamped to `maxbp`|
|maxbp|int|No|The maximum amount of MP the player party has|
|partylevel|int|No|The player party's rank, starts at 1|
|partyexp|int|No|The amount of EXP the player party has (this amount is only the one applicable for the current rank's progression)|
|neededexp|int|No|The amount of EXP needed for the player party to rank up (in other words, `partyexp` needs to reach this value to rank up). This starts at 100|

### Battle modifiers

|Name|Type|Static?|Description|
|----|----|---------|-----------|
|haltbattleload|bool|Yes|If this is set to true right after [StartBattle](StartBattle.md) starts by the caller, StartBattle will wait that it goes to false right after the fade in transition played|
|battlelossevent|bool|Yes|Tells if [ReturnToOverworld](Battle%20flow/Terminal%20coroutines/ReturnToOverworld.md) should be called without flee if [DeadParty](Battle%20flow/Terminal%20coroutines/DeadParty.md) happens|

### Battle state and results

|Name|Type|Static?|Description|
|----|----|---------|-----------|
|battle|[BattleControl](BattleControl.md)|Yes|The current battle (null if no battle is in progress). Set on [StartBattle](StartBattle.md) when it's null (meaning it wasn't a retry)|
|inbattle|bool|No|Whether we are in battle or not. Set to true on [StartBattle](StartBattle.md) after the fade out transition and set to false in a terminal case where we know the battle will end without retry|
|partyorder|int\[\]|No|The list of party members by their [animid](../Enums%20and%20IDs/AnimIDs.md) ordered by their formation|
|battlenoexp|bool|Yes|Whether the last battle yielded no EXP. Set to false on [StartBattle](StartBattle.md)|
|battlefled|bool|Yes|Tells if the battle ended by fleeing|
|battleenemyfled|bool|Yes|Whether the battle ended while `enemyfled` was true without `expreward`. Set to false on [StartBattle](StartBattle.md). This won't be set to true for an [Enemy](../Entities/NPCControl/Enemy.md) NPCControl encounter if any `GoldenSeedling` [enemy](../Enums%20and%20IDs/Enemies.md) were defeated|
|lastdefeated|List<int>|No|The list of [enemy](../Enums%20and%20IDs/Enemies.md) ids that were defeated in the last battle (amended when applicable by [CheckDead](Battle%20flow/Action%20coroutines/CheckDead.md))|
|lastitemuser|int|Yes|The last player party member's `trueid` who used an item via [UseItem](Battle%20flow/Action%20coroutines/UseItem.md)|
|lastdefeated|List<int>|No|The list of [enemy](../Enums%20and%20IDs/Enemies.md) ids that [CheckDead](Battle%20flow/Action%20coroutines/CheckDead.md) detected were killed due to their `hp` reaching 0 or below. Set to a new list on [StartBattle](StartBattle.md) and reset to a new list after processing drops during the [Death](../Entities/EntityControl/Notable%20methods/Death.md) of the [Enemy](../Entities/NPCControl/Enemy.md) NPCControl if the battle was caused by a regular encounter|
|battleresult|bool|Yes|Whether the battle ended by winning (fleeing don't count) Set to true on [StartBattle](StartBattle.md) and set to false during [TryFlee](Battle%20flow/Action%20coroutines/TryFlee.md) (when succeeding) and during [DeadParty](Battle%20flow/Terminal%20coroutines/DeadParty.md)|

### Uncategorised fields
These fields's semantics haven't been found yet. They will be moved out of this section as they are figured out.

|Name|Type|Static?|Description|
|----|----|---------|-----------|
|firstbattleaction|bool|No|??? TODO: this field is very strange in general|
