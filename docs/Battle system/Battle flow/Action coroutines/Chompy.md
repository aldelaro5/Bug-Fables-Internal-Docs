# Chompy
This action coroutine involves `chompy`'s special attack phase when she is present. It it is called during the [player phase](../Main%20turn%20life%20cycle.md#player-phase) after all player party members's actor turns are done. Whether or not this coroutine has ran is tracked by `chompyattacked` and whether or not it's in progress is tracked by the `chompyattack` coroutine (`chompyaction` also tells if the setup is complete). 

The actual flow is only switched once the setup is complete meaning this coroutine may stay in a [controlled flow](../Update%20flows/Controlled%20flow.md) for a little while before switching to an [uncontrolled flow](../Update%20flows/Uncontrolled%20flow.md)

## Setup
This section runs first and performs any waits or setup needed before switching to an [uncontrolled flow](../Update%20flows/Uncontrolled%20flow.md).

- MainManager.`listredirect` is reset to null which is a workaround of the [inlist issue](../../../ItemList/inlist%20issue.md)
- A frame is yielded if there's at least one `extraenemies`
- All frames are yielded while `checkingdead` is in progress, `summonnewenemy` is true or while any enemy party member's battleentity is in a `forcemove`
- [ReorganizeEnemies(true)](../../Actors%20states/Enemy%20party%20members/ReorganizeEnemies.md) is called which removes all dead enemy party members and orders them by battleentity.position.x
- RefreshEnemyPos is called which checks all `enemydata` whose `hp` is above 0, whose [cantfall](../../Actors%20states/Enemy%20features.md#cantfall) is false and whose [position](../../Actors%20states/BattlePosition.md#battleposition) is `Ground` or `Flying`. If the enemy battleentity.`height` is above battleentity.`minheight` + 0.5, the `position` is set to `Flying`, `Ground` otherwise
- `combo` is set to 1
- `chompyaction` is set to true informing that the setup is complete
- `action` is set to true switching to a [uncontrolled flow](../Update%20flows/Uncontrolled%20flow.md)
- A frame is yielded

## Vine menu setup
This section builds the vine menu.

- A new GameObject named `ChompyVine` is created and childed to `battlemap` with a position of `chompy`'s position + (0.0, 10.0, 0.0)
- A new GameObject named `Base` is created and childed to `ChompyVine` with a scale of to (1.0, 0.75, 1.0) and angles of (0.0, -40.0, 0.0)
- `coptions` is set to a new list with 2 elements being 0 and 1 (these are the attack and do nothing options)
- The list of the ribbons's [item](../../../Enums%20and%20IDs/Items.md) ids in `items[1]` (key items) is obtained locally via EventControl.ChompyRibbons. Only the `ChomperRibbon`, `PoisonRibbon`, `NumbRibbon`, `SleepRibbon` and `RedRibbon` items counts
- If the lost of ribbons isn't empty, 3 is added to `coptions` (this adds the switch ribbons option)
- If [flags](../../../Flags%20arrays/flags.md) 404 is true (chompy has a ribbon on her) and [flagvar](../../../Flags%20arrays/flagvar.md) 56 (the [item](../../../Enums%20and%20IDs/Items.md) id of the ribbon chompy has on her) isn't `ChomperRibbon` `RedRibbon` or 0 (which means unequipped here), 2 is added to `coptions` (this adds the the ribbon specific attack option)
- `maxoptions` is set to the final amount of `coptions`
- `option` is set to `chompyoption`. This means the default option will be the last one selected during the battle, but if that option index is no longer available by checking if it's equal or higher than `maxoption`, `option` is set to 0 instead (attack)
- For each option indexes available from 0 to `maxoptions` exclusive:
    - A `Prefabs/Objects/Vine` is instantiated to a new GameObject childed to the `Base` with a position of `ChompyVine`'s position + (0.0, 0.0, -1.0), a scale of (2.5, 2.5, 2.5) and a SpriteRenderer component
    - `ChompyVine`'s angles are decremented by (0.0, 360 / `maxoptions` converted to float, 0.0)
    - The sprite of the SpriteRenderer of the vine GameObject is set depending on the corrsponding `coption` matching the option index:
        - 0 (Attack): `guisprites[147]`
        - 1 (Do Nothing): `guisprites[148]`
        - 2 (Ribbon specific action): Depends on the [flagvar](../../../Flags%20arrays/flagvar.md) 56 (the [item](../../../Enums%20and%20IDs/Items.md) id of the ribbon chompy has on her). This also initialises the attack property of the bite specific attack (check the selected option handling for details). The sprite is `guisprites[158]` for the `PoisonRibbon`, `guisprites[159]` for the `NumbRibbon` and `guisprites[160]` for the `SleepRibbon`. Additionally, if instance.`tp` is less than 2, the material of the sprite is set to MainManager.`grayscale`
        - 3: (Switch Ribbon) `guisprites[217]`
    - The sprite of the vine GameObject's flipX is set to true
- [currentaction](../../Player%20UI/Pick.md) is set to `Chompy`
- [UpdateText](../../Visual%20rendering/UpdateText.md) is called

## Vine menu selection loop
This is an infinite loop that runs until an option has been confirmed in the vine menu. Everything in this section runs perpetually until explicitly said otherwise.

- [RefreshEnemyHP](../../Visual%20rendering/RefreshEnemyHP.md) is called
- As long as input 4 (confirm) isn't pressed, the following runs in an inner infinite loop:
    - If input 3 (left) or input 2 (right) is pressed, the `Scroll` sound is played followed by the `option` being decremented or incremented accordingly with wrap around to `maxoptions` - 1 followed by [UpdateText](../../Visual%20rendering/UpdateText.md) being called
    - `Base` y angle is lerped from the existing one to -`option` * (360.0 / `maxoptions`) - 40.0 with a factor of 1/10 of the game's frametime
    - `Base`'s y scale is lerped from the existing one to 1.0 with a factor of 1/10 of the game's frametime (the x and z scale are set to 1.0)
    - A frame is yielded
- Getting here means a main vine menu option was chosen. If it's 2 (ribbon specific action) while instance.`tp` is less than 2, the buzzer sound is played followed by a frame yield followed by perfoming another iteration of the selection loop
- If the confimed option isn't 3 (switch ribbons), the slection loop ends (switch ribbon is the only option that needs further handling once chosen)
- The `Confirm` sound is played if it wasn't already
- [currentaction](../../Player%20UI/Pick.md) is set to `BaseAction`
- [UpdateTect](../../Visual%20rendering/UpdateText.md) is called
- instance.`multilist` is set to the ribbons list obtained earlier except the one held in [flagvar](../../../Flags%20arrays/flagvar.md) 56 (the one chompy has on her)
- [SetText](../../../SetText/SetText.md) is called in [dialogue mode](../../../SetText/Dialogue%20mode.md#dialogue-mode) using the text `|`[hide](../../../SetText/Individual%20commands/Hide.md)`||`[pickitem](../../../SetText/Individual%20commands/Pickitem.md)`,35,-11,-11|` (which pops an [ItemList](../../../ItemList/ItemList.md) with the [chompy ribbons list type](../../../ItemList/List%20Types%20Group%20Details/Chompy%20Ribbons%20List%20Type.md)). The call has these properties:
    - [fonttype](../../../SetText/Notable%20states.md#fonttype) of 0 (`BubblegumSans`)
    - linebreak of `messagebreak`
    - No tridimensional
    - position of Vector3.zero
    - No cameraoffset
    - size of Vector3.one
    - No parent
    - No caller
- [RefreshEnemyHP](../../Visual%20rendering/RefreshEnemyHP.md) is called
- As long as the [message](../../../SetText/Notable%20states.md#message) lock is active:
    - `Base`'s y angle is set to a lerp from the existing one to -`option` * (360.0 / `maxoptions`) - 40.0 with a fractor of `framestep` * 0.1
    - `Base`'s y scale is lerped from the existing one to 1.0 with a factor of `framestep` * 0.1 (the x and z scale are set to 1.0)
    - A frame is yielded
- Getting here means the [ItemList](../../../ItemList/ItemList.md) handling and the dialogue has completed. If MainManager.`listcancelled` is true, it means the selection loop isn't done yet and the following is performed:
    - `option` is set to the value it had before the SetText call (this is because the ItemList system uses it so it restores the value to continue being used for the vine menu)
    - `maxoptions` is set to the amount of `coptions` (this is also because it was used as part of ItemList)
    - [currentaction](../../Player%20UI/Pick.md) is set back to `Chompy`
    - [UpdateText](../../Visual%20rendering/UpdateText.md) is called
    - Another iteration of the vine menu selection loop is performed
- The value of [flagvar](../../../Flags%20arrays/flagvar.md) 0 is saved locally (this corresponds to the [item](../../../Enums%20and%20IDs/Items.md) id of the selected ribbon)
- The vine menu selection loop ends here as the switch ribbon option was just confirmed

## Selected option handling
Getting here means that an option has been confirmed in the vine menu selection loop and the coroutine is ready to handle it.

- `chompyoption` is set to the selected `option`
- [currentaction](../../Player%20UI/Pick.md) is reset to `BaseAction`
- [UpdateText](../../Visual%20rendering/UpdateText.md) is called
- The `Confirm` sound is played
- The `ChompyVine` object is destroyed
- `hideenemyhp` is set to true

What follows depends on the `coption[option]` selected.

### 0 (Attack) or 2 (Ribbon specific action)

- If the `coption` is 2 (ribbon specific action), instance.`tp` is decreased by 2
- All player party members has DestroyConditionIcons called on their entity (not battleentity)
- All enemy party members has DestroyConditionIcons called on their entity if it exists (this is normally the same than battleentity)
- The enemy party member to target is obtained by getting the first one whose [position](../../Actors%20states/BattlePosition.md) is `Ground` or `enemydata[0]` if none are found
- MainManager.SetCamera is called with no target, the targetted enemy's battleentity's position as the targetpos, 0.03 as the speed and (0.0, 2.85, -7.5) as the offset
- The x/z position of `chompy` are saved locally (the y component is saved to 0.0)
- `chompy`'s `overrideflip` is set to true
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md) is called on `chompy` to move towards the targetted enemy's battleentity's position + (0.0 - the `size` of the targetted enemy, 0.0, -0.1) with a multiplier of 3.0
- All frames are yielded while `chompy` is in a `forcemove`
- `chompy`'s [animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 100
- The `Chew` sound is played with a pitch of 1.5
- If the targetted enemy's [position](../../Actors%20states/BattlePosition.md) is indeed `Ground` (always happen unless no grounded enemies existed earlier):
    - [DoCommand](../../DoCommand.md) is called with a timer of 60.0, a commandtype of `PressKeyTimer` and a data being a one element array containing a random integer between 4 and 6 converted to float
    - A frame is yielded
    - All frames are yielded while `doingaction` is true
- A base amount of damage to deal is determined. It's 2 + the amount of `HelperBoost` [medals](../../../Enums%20and%20IDs/Medal.md) equipped (under normal gameplay, there's only 1 medal so the value is 3 with the medal, 2 without)
- If the targetted enemy's [position](../../Actors%20states/BattlePosition.md) is indeed `Ground` (always happen unless no grounded enemies existed earlier) and `commandsuccess` is true:
    - `chompy`'s [animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 102
    - The `Bite` sound is played with a pitch of 1.25
    - [DoDamage](../../Damage%20pipeline/DoDamage.md) is called without attacker to the targetted enemy for a damage of the base amount determined earlier (with 1 added if [flags](../../../Flags%20arrays/flags.md) 404 is true and [flagvar](../../../Flags%20arrays/flagvar.md) 56 is `ChomperRibbon` meaning that ribbon is currently on Chompy) without block. The property depends on the ribbon:
        - `PoisonRibbon`: `Poison`
        - `NumbRibbon`: `Numb`
        - `SleepRibbon`: `Sleep`
        - Anything else: No properties
    - if [flagvar](../../../Flags%20arrays/flagvar.md) 56 (the [item](../../../Enums%20and%20IDs/Items.md) id of the ribbon chompy has on her) is `RedRibbon` and `lastdamage` is above 0:
        - instance.`tp` is incremented then clamped from 0 to instance.`maxtp`
        - The `Heal2` sound is played
        - [ShowDamageCounter](../../Visual%20rendering/ShowDamageCounter.md) is called with type 2 (TP) with an ammount of 1 starting from `chompy`'s position + (-1.0, 1.0, 0.0) and ending at Vector3.up
- Otherwise (meaning no grounded enemies existed or the action command was failed):
    - 0.1 seconds are yielded
    - The `AtkFail` sound is played
    - `chompy`'s [animstate](../../../Entities/EntityControl/Animations/animstate.md) is set to 103
- 0.4 seconds are yielded
- [SetDefaultCamera](../../Visual%20rendering/SetDefaultCamera.md) is called
- `chompy`'s `rigid` constraints are set to freeze everything except x/z position
- [MoveTowards](../../../Entities/EntityControl/EntityControl%20Methods.md) is called on `chompy` to move her to the position saved earlier with a multiplier of 2.0
- All frames are yielded while `chompy` is in a `forcemove`
- [StopForceMove](../../../Entities/EntityControl/EntityControl%20Methods.md#stopforcemove) is called on `chompy`
- `chompy`'s `rigid` gravity is enabled
- `chompy`'s position is set to the one saved earlier
- A frame is yielded
- `chompy`'s `flip` is set to true
- `chompy`'s `overrideflip` is reset to false
- `startdrop` is set to true (this allows enemies that requires dropping to finish their drops)
- A frame is yielded (this lets the drops finish)
- `startdrop` is set back to false (since it's no longer needed)
- The targetted enemy party member (if any existed) has their `blockTimes` reset to 0
- `checkingdead` is set to a new [CheckDead](CheckDead.md) coroutine
- All frames are yielded while `checkingdead` is in progress
- If there's still any enemy party member with an `hp` above 0, `action` is set to false switching to a [controlled flow](../Update%20flows/Controlled%20flow.md) (this means the flow isn't changed if all enemies are dead)

### 1 (Do Nothing)
The only thing that happens is `action` is set to false switching to a [controlled flow](../Update%20flows/Controlled%20flow.md)

### 3 (Switch ribbon)

- The `Switch` sound is played
- `chompy`'s `spin` is set to (0.0, -20.0, 0.0)
- 0.33 seconds are yielded
- The selected ribbon's [item](../../../Enums%20and%20IDs/Items.md) id is removed from `items[1]` (key items)
- If [flags](../../../Flags%20arrays/flags.md) 404 is false (Chompy doesn't have a ribbon on her), flags 404 is set to true. Otherwise, [flagvar](../../../Flags%20arrays/flagvar.md) 56 (the [item](../../../Enums%20and%20IDs/Items.md) id of the ribbon chompy has on her) is added to `items[1]` (key items) if it didn't existed in the list already
- [flagvar](../../../Flags%20arrays/flagvar.md) 56 (the [item](../../../Enums%20and%20IDs/Items.md) id of the ribbon chompy has on her) is set to the selected ribbon item id
- ChompyRibbon is called on `chompy` using their `extrasprites[0]` (their ribbon) which changes its color depending on [flagvar](../../../Flags%20arrays/flagvar.md) 56 (the [item](../../../Enums%20and%20IDs/Items.md) id of the ribbon chompy has on her):
    - `PoisonRibbon`: Lerp between pure red and pure blue with a factor of 0.65 (this means purple with a bit more blue)
    - `NumbRibbon`: Lerp between pure yellow and pure black with a factor of 0.2 (mostly yellow) 
    - `SleepRibbon`: Lerp between pure green and pure black with a factor of 0.3 (mostly green)
    - `RedRibbon`: Lerp between pure white and pure bluw with a factor of 0.5 (light blue)
    - Any other item id: Lerp between pure red and pure white with a factor of 0.5 (light red)
- 0.1 seconds are yielded
- `chompy`'s `spin` is zeroed out
- `action` is set to false switching to a [controlled flow](../Update%20flows/Controlled%20flow.md)

## End
This section contains all the cleanup steps needed to end the action coroutine.

- A frame is yielded if there's at least one `extraenemies`
- All frames are yielded while `checkingdead` is in progress, `summonnewenemy` is true or while any enemy party member's battleentity is in a `forcemove`
- [RefreshEnemyHP](../../Visual%20rendering/RefreshEnemyHP.md) is called
- `chompyaction` is reset to false
- `chompyattacked` is set to true
- `chompyattack` is set to null indicating the coroutine is done
