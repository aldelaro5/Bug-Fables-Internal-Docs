# Miscellaneous features
Maps have other features that aren't easy to categorise while they involve [configuration fields](Fields/Configuration%20fields.md). They will be described here.

They involve the following configuration fields:

|Name|Type|Description|Default|
|---:|----|-----------|-------|
|mainmesh|Transform|The transform containing the "main" mesh which is a major reference points for the map. It determines the default value of `mainrender` (the Renderer of `mainmesh`) and the value of `render` (all the Renderer childed under `mainmesh`)|The first child of MapControl if null|
|discoveryids|int[]|The list of [discovery](../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) ids that this map contains. The ones whose flags are still false becomes detectable by the `Detector` [medal](../Enums%20and%20IDs/Medal.md) when equipped (except if the `mapid` is the `TermiteIndustrial` [map](../Enums%20and%20IDs/Maps.md) while the player z position is less than 20.0 where no discoveries can be detected)|Empty array|
|readdatafromothermap|MainManager.Maps|From prefab|When set to any value other than `TestRoom`, it will cause Start to read the `dialogues`, `commandlines` and `entities` TextAsset data from that map instead of the ones from `mapid`|`TestRoom`|
|preloadobjs|GameObject[]|A dummy list of GameObject that allows to preemptively load prefabs so they can be cached if they get loaded again. This only holds references, the actual GameObjects aren't used from this field|Empty array|
|alivetime|float|The amount of frames (excluding the first one) before LateUpdate is allowed to restore the disabled, but initially enabled [Fader](Graphics%20configuration.md#fader-control) and also to update the enablement of [DoorOtherMap](../Entities/NPCControl/ObjectTypes/DoorOtherMap.md) and [NPC](../Entities/NPCControl/NPC.md) (including their `emoticon`) |20.0|
|cantcompass|bool|If this is true, the `AntCompass` [key item](../Enums%20and%20IDs/Items.md) cannot be used on this map|false|
|autoevent|Vector2[]|A list of [events](../Enums%20and%20IDs/Events.md) in the y component to automatically start the moment the [flag](../Flags%20arrays/flags.md) slot of the x component is false. This is checked on each LateUpdate after the first one and when the event starts, the flag slot of the x component becomes true so it doesn't start the event again|Empty array|

## `mainmesh`
Maps by convention all have a GameObject typically called `Base` which contains most of the map's meshes, colliders and geometry. The name is only a convention, what makes this GameObject special is it's the value of the `mainmesh` configuration field. It can either be assigned directly from prefab, but if not, it will default to the first child of the MapControl's GameObject (which also typically happens to be the `Base`).

This GameObject by itself isn't too special. The only thing it influences directly is the default value of `mainrender` which is the `mainmesh`'s Renderer, but that can be overriden by assigning one in the prefab and `mainrender` is UNUSED so this doesn't do anything.

What does something however is `mainmesh` influences the values of `render` since that value are all the Renderer childed under `mainmesh`. This is important because the `render` have a huge impact on [outside fading](Insides.md#outside-fading) as it determines what will be faded. It's also used as a key reference points by the rest of the game such as events. There's even some [utilities methods](Public%20utilities%20methods.md) that makes use of it as a reference point and its `render`.

## `discoveryids`
The `Detector` [medal](../Enums%20and%20IDs/Medal.md) needs to know the [discoveries](../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) that can be found in the map in advance because the game needs to find if it should have the medal beep or not when the map loads. While it could be possible to check some [NPCControl](../Entities/NPCControl/NPCControl.md) to see which ones provides discoveries and to check if those are unlocked, this unfortunately can't cover everything. Discoveries can be unlocked in a wide variety of ways including the [discovery](../SetText/Individual%20commands/Discovery.md) SetText command or by [events](../Enums%20and%20IDs/Events.md).

To remedy this, maps have a configuration field that contains the discoveries id. This is how maps can declare in advance which ids to check for the medal to beep or not. The downside to this approach however is this list isn't bound by the reality: it's possible this list doesn't match the actual list the maps provides so it's possible the medal ends up doing a false negative or a false positive.

It is still the field that controls the medals however so the elements contained within should logically match reality for the medal to work properly. All of this is enforced in the CheckDisc method which is invoked 1.0 seconds after Start.

There is an exception to this: in the `TermiteIndustrial` [map](../Enums%20and%20IDs/Maps.md), it has a unique feature where there are 2 portions to it, but each portions loads the same map. While this is done for entities processing performance, it interferes with the `Detector` medal feature because the check always happen on the map's Start so each of the 2 portions of the maps will check the discoveries for the entirety of the map including the portion the player isn't present on.

Due to this, on this map, the medal is only allowed to beep if the `player` position is less than 20.0 by the time CheckDisc runs which is the main portion of the map. This does involve the limitations that no discoveries can be placed on the portion near the Colosseum, but it does allow the feature to work consistently.

## `readdatafromothermap`

This field mainly affects [CreateEntities](Init%20methods/CreateEntities.md) and the [SetText configuration](SetText%20configuration.md) where it will cause the data to be read from another map than the one being initialised.

## `preloadobjs`
While the value of this configuration field is unused, in practice, it is used for performance reasons. It is meant to be a dummy GameObject array field to force Unity to preload them whenever the map's prefab gets instantiated.

Doing so will force Unity to cache since a reference is kept by the map. Further loads of any GameObjects in this array will go much faster than if it wasn't present.

## `alivetime`
This configuration fields delays some logic dispatched by LateUpdate where the value is the amount of frames (excluding the first LateUpdate where `latestart` happens) to delay these logic from happening.

It impacts the following logic:

- Any [Faders](Graphics%20configuration.md#fader-control)'s restoration on all disabled Fader that were initially enabled
- Any enablement update on [DoorOtherMap](../Entities/NPCControl/ObjectTypes/DoorOtherMap.md) or [NPC](../Entities/NPCControl/NPC.md) and their `emoticon` update

## `cantcompass`
This configuration field does what it says on the tin when set to true: prevents the usage of the `AntCompass` [key item](../Enums%20and%20IDs/Items.md).

However, the influence of this field is inconsistent because it alone won't influence this behavior. The field won't do anything under the following conditions because if any of these are true, it means the item cannot be used regardless of the value of `cantcompass`:

- The `player`'s `candig` is false
- The `player` is in an [inside](Insides.md)
- The `player` is on a `conveyor`
- [Flag](../Flags%20arrays/flags.md) 400 is true (this is the temporary flag that gets reset to false on each map load)
- [Flag](../Flags%20arrays/flags.md) 401 is true (this is supposed to be the stealh mode flag, but its management is inconsistent, see the [issue with CloseMove](Follower%20system.md#exploring-further-the-issue-with-closemove) documentation to learn more)
- [Flag](../Flags%20arrays/flags.md) 10 is false (this is both for having received the spy tutorial, but it's also used during the Overseer rescue section in chapter 3 where it becomes false only during the section so this conditions enforces that the item can't be used during it)

However, there is an additional case where the value can be overriden to true by [AreaSpecific](Init%20methods/AreaSpecific.md) when the [area](../Enums%20and%20IDs/librarystuff/Areas.md) is `WaspKingdom`. It happens on every maps of the area except the following ones:

- `WaspKingdomOutside`
- `WaspKingdomJayde`
- `WaspKingdomMainHall`
- `WaspKingdomPrison`
- `WaspKingdom5`
- `WaspKingdomQueen`
- `WaspKingdomThrone`

The value from the map's prefab of this configuration field only dictates that usage of the item is forbidden when all of the above cases don't happen.

## `autoevent`
Maps have a way to automatically trigger [events](../Enums%20and%20IDs/Events.md) relatively soon after the map loaded. This feature is called automatic events which are defined by the `autoevent` configuration field and they allow to conditionally trigger events on the first LateUpdate where all of the following conditions are fufilled:

- `latestart` happened (in other words, this is the second LateUpdate cycle or later)
- We aren't in a `pause`
- We aren't in a `minipause`
- We aren't `inevent`
- The [message](../SetText/Notable%20states.md#message) lock is released

To define an automatic event, an element must be defined in `autoevent` like the following (NOTE: while the type of the array is Vector2, they are used as Vector2Int by casting each component to int before usage so any decimal part will get truncated when used):

- x: A [flag](../Flags%20arrays/flags.md) slot that is required to be false for the event to trigger and that will be set to true once the event triggers
- y: The [event](../Enums%20and%20IDs/Events.md) id to trigger when the x component condition is satisfied and all of the above conditions of LateUpdate are satisfied
