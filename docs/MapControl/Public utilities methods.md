# MapControl public utitlities methods
MapControl contains multiple public methods that can be called outside of the map's logic. Some of there are used internally by MapControl, but may be used elsewhere such as by [events](../Enums%20and%20IDs/Events.md) who needs finer control on the map.

Finds a Transform directly childed to `mainmesh` which has the specified GameObject's name. This only covers the DIRECT childs of `mainmesh`. If no GameObject is found, null is returned

```cs
public Transform FindByName(string name)
```

More details can be found at the [Music method](Init%20methods/Music.md) documentation

```cs
public void Music()
```

Changes all SoundControl found on the map's `source`.volume to their `startvolume` * value. A SoundControl is a component that can be defined in a map on a GameObject with an AudioSource (or an external one may be defined in the `source` field) which will adjust its volume based on the game's volume settings. This method is meant to be used when those settings changes because it wouldn't otherwise be possible to adjust their volume otherwise

```cs
public void RefreshSoundVolume(float value)
```
An overload of the above where value is MainManager.`soundvolume`

```cs
public void RefreshSoundVolume()
```

More details on these 2 overloads can be found at the [MoveInside](Insides.md#moveinside) documentation.

```cs
public IEnumerator MoveInside(NPCControl caller, bool move)

```
```cs
public IEnumerator MoveInside(NPCControl caller)
```

Returns the childs of `mainmesh` whose child index matches the ones sent in the form of a Transform array. NOTE: This method assumes the child indexes sent exists. If any doesn't, an exception will be thrown.

```cs
public Transform[] GetChilds(int[] index)
```

This method allows to override the camera configuration saved to be restored later by [RefreshInsides](Insides.md#refreshinsides) when calling it programmatically (typically by [events](../Enums%20and%20IDs/Events.md)). The caller must be a [DoorSameMap](../Entities/NPCControl/ObjectTypes/DoorSameMap.md) and it will be used to override the `camoffset` and `camangleoffset` to save. It does the following:

- Sets `tcpos` to `camlimitpos`  
- Sets `tcneg` to `camlimitneg`
- Sets the caller's `vectordata[4]` to `camoffset`
- Sets the caller's `vectordata[5]` to `camangleoffset`

It is meant to be called right before RefreshInsides which overrides the camera configurations to restore when calling RefreshInsides.

```cs
public void SetTCLimits(NPCControl caller)
```

More details can be found at the [RefreshInsides](Insides.md#refreshinsides) documentation.

```cs
public void RefreshInsides(bool inside, NPCControl caller)
```

This will do the following on all entities except the one specified in exception (the value null means no exceptions is specified and this will operate on all entities):

- Call [StopMoving](../Entities/EntityControl/EntityControl%20Methods.md#stopmoving) on it where the targetstate is its [animstate](../Entities/EntityControl/Animations/animstate.md) if the state parameter. However, if the state parameter isn't negative, it will be used as the targetstate instead
- If the entity has an `npcdata`, [StopForceBehavior](../Entities/NPCControl/Notable%20methods/StopForceBehavior.md) is called on it

```cs
public void StopMovingEntities(EntityControl exception, int state)
```

An overload of the above where exception is null and state is -1.

```cs
public void StopMovingEntities()
```

Disables all `render`.
```cs
public void HideRenders()
```

Set `camlimitneg` to (-999.0, -999.0, -999.0) and `camlimitpos` to (999.0, 999.0, 999.0). If settemp is true, the values will be saved before being set to `tempcneg` and `tempcpos` respectively which can be restored later using a RestoreLimit(true) call.

```cs
public void RemoveLimit(bool settemp)
```

Sets `camlimitneg` to `originallimitneg` and `camlimitpos` to `originallimitpos` unless restoretemp is true where they are instead set to `tempcneg` and `tempcpos` respectively. The latter is meant to be used after a RemoveLimit(true) call.

```cs
public void RestoreLimit(bool restoretemp)
```

Appends the entity en at the end of `entities` and child it to the map. This is only used for the [medals shop](../Entities/NPCControl/Shop%20system.md#medals-shop-specifics) to add the shelved [Items](../Entities/NPCControl/ObjectTypes/Item.md) entities.

```cs
public void AddInEntity(EntityControl en)
```
