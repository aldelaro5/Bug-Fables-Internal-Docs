# Insides
Maps can define specific locations within the map called an "inside".

An inside is a portion of the game (defined by its root GameObject) that should only be accessible through a [DoorSameMap](../Entities/NPCControl/ObjectTypes/DoorSameMap.md) where the inside can be "entered" or "exited" through those NPCControl. It shouldn't in practice be possible to access the inside or getting out of it without passing through a DoorSameMap because the current inside the player is in greatly impacts entities's Update logic meaning violating this rule will cause unexpected behaviors. Essentially, most entities aren't considered active when the current inside doesn't match the inside an entity is defined in. There's also transitions that happens when entering or exiting an inside. They are also treated differently visually where entering an inside will cause the outside to be faded and this is only reversed when exiting the inside.

It should be noted that a DoorSameMap acts as both an entry and an exit point, but an inside may have multiple DoorSameMaps leading in and out of it. As long as it's not possible to access the inside or to get out of it without going through a DoorSameMap, it will behave as expected.

Additionally, it's possible to configure a [JumpSpring](../Entities/NPCControl/ObjectTypes/JumpSpring.md) to use an existing DoorSameMap when jumped on, but in that case, it still ends up doing the same procedure than the player going through a DoorSameMap directly, it's just performed programmaically by the JumpSpring.

At the most basic level, an inside can be considered very close to the concept of "interior", but what makes them special is they are managed in a particular way by the game where it treats the insides differently than anything else (or the "outside").

This system involves multiple configuration fields to define the insides and to configure their behaviors:

|Name|Type|Description|Default|
|---:|----|-----------|-------|
|insides|GameObject[]|The list of GameObject in this map that is considered an interior part and will get special treatment during rendering and include special transition managed by [DoorSameMap](../Entities/NPCControl/ObjectTypes/DoorSameMap.md) map entities|Empty array|
|insidetypes|InsideType[]|This is meant to be a matching array for `insides` and each matching element corresponds to the sound that will play when entering or exiting the inside. This field's string value is the prefix to the sound's name: when entering the inside, the name of the sound that will play ends with `DoorEnter` and when exiting it, it will end with `DoorExit`. Concatenating this field's string value as the prefix gives the full sound name to play when entering or exiting the inside|Empty array, but if the length doesn't match `insides` on Start, it is reset to be an array with the same length, but with all values left to `Stretch`|
|tieinsidedoorentities|bool|If this is true when entering an inside, every [DoorSameMap](../Entities/NPCControl/ObjectTypes/DoorSameMap.md) in the map has their `vectordata[4]` and `vectordata[5]` set to the current `camoffset` / `camangleoffset` which means no matter how the inside is exited, the camera values will be restored consistently to the values when it was entered|false|
|hideinsides|bool|If this is true, all `insides` that doesn't match the current one will be disabled. This will also disable any entity with `hideinside` set to true whose `insideid` doesn't match the current one. This is only used in the `AntTunnels` [map](../Enums%20and%20IDs/Maps.md) to hide the break room part of the map|false|
|setinsidecenter|bool|If this is true, whenener RefreshInsides is called, instance.`camtarget` will be set to the current inside's transform after entering one and set back to MainManager.`player`'s transform when exiting one|false|
|fadingspeed|float|The fraction of TieFramerate to use in FixedUpdate when updating `fadeammount` when entering or exiting an inside|0.2|

## Defining an inside
In order to define an inside, the following must be done on the map's prefab:

- The root GameObject of the inside must be listed in the `insides` configuration field. An inside can only contain a single root GameObject that will also includes its children. This means that if multiple GameObject needs to be part of the inside, they must share a common root GameObject even if it's an empty one. This GameObject must NOT be be located under the `mainmesh`'s GameObject because those are what the outside fading will affect so the inside can't be there. It has to sit outside of the `mainmesh`'s tree (it can be a sibling to it which is usually what's done)
- The index matched of the `insidetypes` configuration field of the `insides` element mentioned above must exists. The type of the value is a special enum called `InsideType` which informs the game which enter and exit sound should play when entering or exiting the inside. It's possible to leave the array empty or to not have its length match `insides`, but doing this will cause all applicable elements to be created and default to `Stretch`. The possible values for this enum are as follows as well as their enter/exit sound they refer to (they are named `Ressources/audio/sounds/XDoorEnter` and `Ressources/audio/sounds/XDoorExit` where `X` is the string representation of the enum value):
    - `Stretch`: `StretchDoorEnter` and `StretchDoorExit`
    - `Slide`: `SlideDoorEnter` and `SlideDoorExit`
    - `Twist`: `TwistDoorEnter` and `TwistDoorExit`

This is all that's needed to define the inside, but to actually leverage it, at least 1 [DoorSameMap](../Entities/NPCControl/ObjectTypes/DoorSameMap.md) must exist whose `data[0]` is bound to the inside's id. This id corresponds to its `insides` / `insidestypes` index. Also, as mentioned earlier, the inside shouldn't be able to be reached or escaped physically without going through a DoorSameMap so the colliders needs to be setup appropriately.

Multiple DoorSameMaps may be defined for the same insides, but if this is the case for at least one inside in the map, it is recommended to also set the `tieinsidedoorentities` configuration field to true because it allows the `camoffset` / `camangleoffset` restore function when exiting the inside to be consistent regardless of how the inside was entered. See the section below for more details.

## `tieinsidedoorentities`
This field allows to make the `camoffset` / `camangleoffset` restore feature when exiting an inside to work consistently even if the inside has multiple DoorSameMape bound to it.

If it's true when entering an inside, every DoorSameMap in the map has their `vectordata[4]` and `vectordata[5]` set to the current `camoffset` / `camangleoffset` which means no matter how the inside is exited, the camera values will be restored consistently to the values when it was entered

## `hideinsides`
If this field is true, all `insides` that doesn't match the current one will be disabled. This will also disable any entity with `hideinside` set to true whose `insideid` doesn't match the current one.

This is only used in the `AntTunnels` [map](../Enums%20and%20IDs/Maps.md) to hide the break room part of the map

## `setinsidecenter`
If this is true, it overrides the camera system more by setting instance.`camtarget` to the `insides`'s transform when entering it. It will be reset to the `player`'s transform when exiting the inside

## Outside fading
During the course of entering or exiting an inside, FixedUpdate can fade in or out RenderSettings.`skybox` and all `render` who will also end up being disabled using a tracking field called `fadeammount`. It is contained in a method called UpdateInsideColor which receives a targetalpha parameter which is the opacity targetted depending if inside or outside (1.0 if outside, 0.0 if inside). 

Normally, none of the `render` should be in any inside so it's effectively fading in and out the "outside". This involves the `fadingspeed` configuration field which is the fraction of TieFramerate to use in FixedUpdate when updating `fadeammount` when entering or exiting an inside.

This fading logic is relatively complex because it doesn't always happen and is incorrectly intertwined with other unrelated graphical features. For the fading to happen in general, all the following conditions must be fufilled (they don't do anything in practice however because the maps in which these would apply don't have any `insides` defined):

- `nocolorchange` is false. This is only set to true as part of [AreaSpecific](Init%20methods/AreaSpecific.md) in the following [areas](../Enums%20and%20IDs/librarystuff/Areas.md):
    - `BanditHideout`
    - `ChomperCaves`
    - `SandCastle`
- `overrideskybox` is false. This is unrelated because it's supposed to only prevent the skybox's `_Rotation` to change, but it's still required to be false for inside fading
- RenderSettings.`skybox` must exists. This is required because the `skybox` needs to be faded alongside the `render` objects

Assuming all the above are fufilled (normally always the case when it matters), there are 2 parts to the fading logic:

- The actual update of the fading (when entering or exiting)
- The cleanup of the fading when fully outside after exiting

Also, `render`'s GameObjects can have tags that changes the behavior of the fading:

- `NoMapColor`: The `render`'s materials's colors won't change when entering an inside and its sharedMaterials won't be restored to `ogmats` when during the cleanup part. However, the materials's enablement will still be updated depending on `fadeammount`
- `AlwaysShow`: The `render` will never have its materials's colors changed and it will always remain enabled during the fading, but its sharedMaterials will still be restored to `ogmats` during the cleanup part

### Fading update
The fading only updates whenever `fadeammount` gets further than 0.025 away from the value it should be which is 0.0 if in an inside and 1.0 if not (the targetalpha of UpdateInsideColor).

Essentially, `fadeammount` is what controls the materials's color to fade in or out. Everytime `fadeammount` changes, the materials's colors will be set to a color where all r, g and b components will be set to `fadeammount` (except for the skybox where it's clamped from 0.0 to 0.5).

It effectively means that the colors will change to pure black when entering the inside and back to pure white (which is normal coloring) when exiting the inside (except the skybox where it goes to pure gray).

The destination value (0.0 or 1.0) is constantly updated, but whether any changes happen depens on if `fadeammount` gets further away then 0.025 from it. It means that the moment instance.`insideid` changes, FixedUpdate will have the fading happen accordingly on its own.

As for the actual fading, it is done like the following:

- `fadeammount` is set to a lerp from the existing one to the destination value with a factor of TieFramerate(`fadingSpeed`)
- RenderSettings.`skybox` has its `_Tint` set to a color where r, g and b are `fadeammount` clamped from 0.0 to 0.5
- If `insidedum` is null, it is created as a new MaterialPropertyBlock
- Each `render` that isn't null and have an `AlwaysShow` tag will be enabled, but their materials color won't change
- Each `render` that isn't null and don't have a `AlwaysShow` tag will have the following happen:
    - The `render` enablement is set depending on `fadeammount`. It's enabled if it's higher than 0.15 and disabled otherwise
    - If the `render` doesn't have a `NoMapColor` tag and fufills other conditions, `insidedim` will have its `_Color` set to a color where r, g and b are `fadeammount` (the a is the sharedMaterial's color.a) followed by `insidedim` being set on the `render`'s porperty block. Here are the conditions for a `render` material to have its color changed:
        - The shader isn't `emptymat`'s shader
        - The shader isn't `outlinemain`'s shader
        - The shader isn't `fakelight`
        - The material has a property named `_Color`

## Inside transition
The procedure involved in entering or exiting the inside is done in 3 ways:

- By a [DoorSameMap](../Entities/NPCControl/ObjectTypes/DoorSameMap.md)'s trigger collider
- By a [JumpSpring](../Entities/NPCControl/ObjectTypes/JumpSpring.md) configured to change insides using an existing DoorSameMap in the map
- Manually by the game (usually done through [events](../Enums%20and%20IDs/Events.md))

The first 2 leads to the same path: a coroutine called MoveInside. This method is meant to manage the entire inside transition and it also calls another method called RefreshInsides which is meant to handle the change of instance.`insideid`. It requires a DoorSameMap to function because it performs the transition using the configuration the DoorSameMap defines.

However, the third way doesn't have to involve MoveInisde. It's possible for the game to take control of the transition such that it can manually set instance.`insideid` without going through MoveInside. This is because it might not be necessary to involve any DoorSameMap which MoveInside requires. To do this, RefreshInsides needs to be called right after instance.`insideid` is changed.

The sections below outlines the 2 methods as they are responsible for handling entering or exiting an inside.

### MoveInside
A coroutine that handles entering or exiting an inside via a [DoorSameMap](../Entities/NPCControl/ObjectTypes/DoorSameMap.md).

```cs
public IEnumerator MoveInside(NPCControl caller, bool move)
```
There's an overload without the `move` parameter where it default to false. The behavior is the same as it simply start a coroutine with the other one and then yield breaks immediately.

The `caller` parameter tells the DoorSameMap that causes this inside transition (it cannot be null otherwise, an excpetion will be thrown) and the `move` parameter tells if the party and its followers should move during the transition or not.

Here's what the coroutine does:

- If `samira` exists, SamiraStop is called on MainManager.`events` using `samira` with instant followed by `samira` being set to null
- `player` entity's `emoticon` is removed by playing the `-1` clip on it with `emotioncooldown` set to 0.0
- `player`'s `npc` list is reset to a new list
- dynamicFriction of the `player`'s `cool` material is set to 0.0 (the old value is saved for restore later)
- CancelAction called on `player`
- Enter a `minipause`

If we aren't in an inside, it means we are entering an inside, otherwise, we are existing it. What happens next depends if we are entering or exiting an inside (the cleanup section happens regardless at the end).

#### Entering an inside

- If we aren't `inevent`, the corresponding sound is played which has the name `XDoorEnter` where `X` is the string value of the matching `insidetypes` (the inside id is fetched from the caller's `data[0]`)
- If move is true:
    - TeleportFollowers called with 2.0 distance, `Center` direction and the `player` as the caller
    - Every `playerdata` entity gets rooted and their `rigid` velocity zeroed out
    - If `chompy` exists, they are childed to this map and their `rigid` velocity is zeroed out
- If the caller's `data[1]` exists (a Music id is set to play while in this inside):
    - ChangeMusic is called with the string value of the matching Music with 0.2 fadespeed
    - If the music is not among the following list, CheckSamira is called with the Music's string value (this adds the Music to instance.`samiramusics` if it wasn't in it already):
        - `Calm`
        - `Title`
        - `Wind`
        - `Water`
        - `MachineHum`
        - `Breathing`
- The matching `insides`'s Animator gets the `Open` clip played on it if it exists
- MainManager.`lastinside` is set to -1 (this field is UNUSED)
- instance.`insideid` is set to the inside we are entering which is the caller's `data[0]`
- RefreshInsides called with inside and with the caller
- If move is true, `player`'s entity MoveTowards the caller's `vectordata[0]` with 1.0 speed, 1 (`walk`) state and 0 (`Idle`) stopstate
- The caller's `vectordata[4]` and `vectordata[5]` are set to instance.`camoffset` and instance.`camangleoffset` respectively
- If the caller's `vectordata[2]` magnitude is above 0.1, instance.`camoffset` is set to it (set to `defaultcamoffset` otherwise)
- If the caller's `vectordata[3]` magnitude is above 0.1, instance.`camangleoffset` is set to it (set to `defaultcamangle` otherwise)
- If `tieinsidedoorentities` is true, all other DoorSameMap in `entities` other than the caller have their `vectordata[4]` and `vectordata[5]` set to the caller's
- Yield all frames until the `player`'s entity's `forcemoove` is done
- Every `playerdata` entity gets rooted and their `rigid` velocity zeroed out
- If `chompy` exists, they are childed to this map and their `rigid` velocity is zeroed out

#### Exiting an inside

- If we aren't `inevent`, the corresponding sound is played which has the name `XDoorExit` where `X` is the string value of the matching `insidetypes` (the inside id is fetched from the caller's `data[0]`)
- If `music` isn't empty, ChangeMusic is called with `music[0]` using a 0.2 fadespeed
- The matching `insides`'s Animator gets the `Close` clip played on it if it exists
- instance.`insideid` is set to -1 marking that we aren't in an inside anymore
- RefreshInsides called without inside and with the caller
- If move is true, `player`'s entity MoveTowards the caller's `vectordata[1]` with 1.0 speed, 1 (`walk`) state and 0 (`Idle`) stopstate
- instance.`camoffset` and instance.`camangleoffset` are restored using the caller's `vectordata[4]` and `vectordata[5]` respectively
- Yield all frames until the `player`'s entity's `forcemoove` is done

#### Cleanup
This happens regardless if we entered or exited an inside:

- Yield for a frame
- `player`'s `lastpos` and its entity's `lastpost` both set to its position
- dynamicFriction of the `player`'s `cool` material is set to the value it had before this coroutine
- DetectIgnoreSphere called on the `player`'s entity which makes their `detect` collider ignore all NPCControl's `scol`
- If a Caravan exists, Refresh is called on the first one found (it plays the `Sleep` or `Idle` animation on the Caravan's `snail` depending on the value of `issleep`)
- Yield all frames until MainManager.`musiccoroutine` is null (in case entering the inside caused a music change)
- The `minipause` is ended

### RefreshInsides
A method that handles updating the map's state after instance.`insideid` changes. It should always and only be called once that value changes. It is involved in MoveInside described above, but the game can call it manually to perform custom inside transitions.

```cs
public void RefreshInsides(bool inside, NPCControl caller)
```

The `inside` parameter tells if we are entering an inside (true) or exiting one (false). The `caller` parameter is optional, but it tells the [DoorSameMap](../Entities/NPCControl/ObjectTypes/DoorSameMap.md) being used if applicable.

Here's what the method does:

- All GameObjects with `DelAftBtl` tag are destroyed. This is only to destroy [BeetleGrass](../Entities/NPCControl/ObjectTypes/BeetleGrass.md) that are in the process of being faded and desatroyed
- If the `player`'s `beemerang` exists, it is destroyed

#### inside is true (entering an inside)

- If there is a caller:
    - All `entities` whose `npcdata`.`insideid` isn't -2 have their enability changed:
        - Disabled if thier `npcdata`.`inisdeid` doesn't match the caller's `data[0]` (the inside we are entering)
        - Otherwise, enabled if `hideinside` is true and thier `npcdata`.`inisdeid` match the caller's `data[0]` (the inside we are entering)
        - If none of the above applies, the enability doesn't change
    - If the caller's `vectordata` contains at least 6 elements (assumed to contain 8 if true), it means the caller wants to set camera limits when entering the inside:
        - If `vectordata[6]` magnitude is above 0.1, `camlimitpos` is set to it (value saved in `tcpos` for restore later)
        - If `vectordata[7]` magnitude is above 0.1, `camlimitneg` is set to it (value saved in `tcneg` for restore later)
- Otherwise (there's no caller):
    - All `entities` whose `npcdata`.`insideid` doesn't match the current inside are disabled
- If `hideinsides`, all `insides`'s enability changes to enable only the inside we are entering, the rest gets disabled
- If there's a caller:
    - All `insides` has the following happen to them:
        - If the inside doesn't match the caller's `data[0]` (inside we are entering), it is disabled
        - Otherwise, if the inside has a Fader, it is disabled
- If `setinsidecenter` is true, instance.`camtarget` is set to to the current `insides`'s transform

#### inside is false (exiting an inside)

- `canlimipos` is restored to `tcpos` (defaults to `originallimitpos` if it's null)
- `camlimitneg` is restored to `tcneg` (defaults to `originallimitneg` if it's null)
- What happends here depends on `hideinsides`:
    - If it's true:
        - All `entities` except DoorSameMap and DoorOtherMap have their enabilitiy changed to be enabled only when their `hideinside` is false and their `npcdata`.`insideid` is -1 (not bound to an inside)
        - All `insides` are disabled
    - Otherwise, if it's false:
        - All `entities` that aren't `iskill` have their enabilitiy changed to be enabled only when their `hideinside` is false and their `oldstate` set to -1 (if they have an `anim`, its speed is reset to 1.0)
        - All `insides` are enabled and if they have a Fader, it is enabled too
- If `setinsidecenter` is true, instance.`camtarget` is set to the `player`

#### Cleanup
This happens regardless if we entered or exited an inside:

- `player`'s `pausecooldown` is set to 7.0
- RefreshEntities called without forceanim and with refreshmap
