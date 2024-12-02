# Map identification
There are 2 [configuration fields](Fields/Configuration%20fields.md) that identifies a map in the game:

|Name|Type|Description|Default|
|---:|----|----------|-------|
|mapid|MainManager.[Maps](../Enums%20and%20IDs/Maps.md)|The unique identifier of this map. Every map should have a different value set from the others|`TestRoom` (must be assigned to a unique value)|
|areaid|MainManager.[Areas](../Enums%20and%20IDs/librarystuff/Areas.md)|The area this map belongs to. On Start, if this field is different than instance.`areaid` (the current area), UpdateArea is called|`BugariaOutskirts` (should be assigned to the logically accurate value)|

## `mapid`
The `mapid` is required to be set. While its default value is `TestRoom`, that value already has a map defined in the game meaning it's invalid to use this value except for the actual test room. It identifies the map specifically and can reliably be used to tell maps appart or to simply know the current map.

## `areaid`
The `areaid` on the other hand is more complex. A map must belong to an area and just like the `mapid`, it is expected to always be set (defaults to `BugariaOutskirts`).

An area has the following properties:

- It takes a slot in `librarystuff` (more information on the [areas](../Enums%20and%20IDs/librarystuff/Areas.md) page). This will be used in the Map window of the PauseMenu to reveal the location (each areas have a hardcoded location on the map)
- It can contain specific logic in [AreaSpecific](Init%20methods/AreaSpecific.md) that always happen when initialising the map (it can also do more logic depending on `mapid`, but areas are also a way to group the logic to apply)
- The current area is stored in instance.`areaid` once the map is initialised which MapControl can use to detect an area change

### UpdateArea
The area is responsible for grouping maps in such a way that going to a map with a different area than the current one involves some logic. Most of it happens in MainManager.UpdateArea:

- If exiting area 15 (Giant's Lair) and [flag](../Flags%20arrays/flags.md) 596 is true (spoke to the Roach Elder in postgame), flag 597 is set to true (changed area after speaking to the Roach Elder in postgame)
- All [regionalflag](../Flags%20arrays/Regionalflags.md) are reset
- instance.`areaid` is updated to the new area
- The `librarystuff` flag of the area is set to true

There are also many other places in the game where the area affects the logic. It's essentially a way to identify groups of maps that belong together logically.
