# Insides

## MoveInside

- If `samira` exists, SamiraStop is called on MainManager.`events` using `samira` with instant followed by `samira` being set to null
- `player` entity's `emoticon` is removed by playing the `-1` clip on it with `emotioncooldown` set to 0.0
- `player`'s `npc` list is reset to a new list
- dynamicFriction of the `player`'s `cool` material is set to 0.0 (the old value is saved for restore later)
- CancelAction called on `player`
- Enter a `minipause`

If we aren't in an inside, it means we are entering an inside, otherwise, we are existing it. What happens next depends if we are entering or exiting an inside (the cleanup section happens regardless at the end).

### Entering an inside

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

### Exiting an inside

- If we aren't `inevent`, the corresponding sound is played which has the name `XDoorExit` where `X` is the string value of the matching `insidetypes` (the inside id is fetched from the caller's `data[0]`)
- If `music` isn't empty, ChangeMusic is called with `music[0]` using a 0.2 fadespeed
- The matching `insides`'s Animator gets the `Close` clip played on it if it exists
- instance.`insideid` is set to -1 marking that we aren't in an inside anymore
- RefreshInsides called without inside and with the caller
- If move is true, `player`'s entity MoveTowards the caller's `vectordata[1]` with 1.0 speed, 1 (`walk`) state and 0 (`Idle`) stopstate
- instance.`camoffset` and instance.`camangleoffset` are restored using the caller's `vectordata[4]` and `vectordata[5]` respectively
- Yield all frames until the `player`'s entity's `forcemoove` is done

### Cleanup
This happens regardless if we entered or exited an inside:

- Yield for a frame
- `player`'s `lastpos` and its entity's `lastpost` both set to its position
- dynamicFriction of the `player`'s `cool` material is set to the value it had before this coroutine
- DetectIgnoreSphere called on the `player`'s entity which makes their `detect` collider ignore all NPCControl's `scol`
- If a Caravan exists, Refresh is called on the first one found (it plays the `Sleep` or `Idle` animation on the Caravan's `snail` depending on the value of `issleep`)
- Yield all frames until MainManager.`musiccoroutine` is null (in case entering the inside caused a music change)
- The `minipause` is ended

## RefresInsides

- All GameObjects with `DelAftBtl` tag are destroyed TODO: this seems unecessary
- If the `player`'s `beemerang` exists, it is destroyed

### inside is true (entering an inside)

- If there is a caller:
    - All `entities` whose `npcdata`.`insideid` isn't -2 have their enability changed:
        - Disabled if thier `npcdata`.`inisdeid` doesn't match the caller's `data[0]` (the inside we are entering)
        - Otherwise, enabled if `hideinside` is true and thier `npcdata`.`inisdeid` match the caller's `data[0]` (the inside we are entering)
        - If none of the above applies, the enability doesn't change
    - If the caller's `vectordata` contains at least 6 elements (assumed to contain 7 if true), it means the caller wants to set camera limits when entering the inside:
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

### inside is false (exiting an inside)

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

### Cleanup
This happens regardless if we entered or exited an inside:

- `player`'s `pausecooldown` is set to 7.0
- RefreshEntities called without forceanim and with refreshmap
