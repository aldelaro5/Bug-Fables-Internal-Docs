# MainManager methods
Since entities is such a central concept in the game, MainManager also offers a wide variety of utility methods to work with entities (either with NPCControl or not).

This page documented them.

```cs
public static EntityControl GetEntity(int id)
public static EntityControl GetEntity(string id, EntityControl caller)
public static EntityControl GetEntity(string id)
private static EntityControl GetEntity(NPCControl caller, string args)
```
This returns the entity matching a certain id `id` with support for various format. The format of the id is documented in the entity id documentation. The `caller` parameter is returned if `id` is `caller` and the game isn't `inbattle`.

The overloads offer various levels of format support:

- First overload only supports numeric id format to address `map`.`tempfollowers`, `playerdata`, `map`.`entities` and `battle`.`extraentities`
- Second overload acts as the first after converting `id` to int when `inbattle`. Otherwise, it adds supports for an `id` of `this` and `caller` as well as supporting `defines` resolution where if resolved, the `id` is converted to int and used as the first overload. If no string `id` is provided and no `defines` exists for `id`, the overload acts as the first after converting `id` to int
- Third overload is the same as the second, but without support for `caller` (most likely, an exception will be thrown if `id` is `caller` in such case)
- Fourth overload acts as the first if `caller` is null. If it's not, it acts as the second overload

```cs
public static bool EntitiesAreNotMoving(EntityControl[] entities)
```
Returns true if no entity in `entities` are in a `forcemove`, false otherwise.

```cs
public static Vector3[] GetEntitiesPos(EntityControl[] e)
```
Returns an array where each element is the position of the matching `e`'s entity.

```cs
public static bool ForceMoving(EntityControl[] e)
```
Returns true if any entity in `e` is in a `forcemove`, false otherwise.

```cs
public static void ForceAnim(EntityControl entity)
```
Calls Play on the `anim` of `entity` with a clip that matches its current `animstate`.

NOTE: This only supports standard animations, extra animations and the `f` argument if the `height` is above 0.1.

```cs
public static void SetEntityLastPos(bool setit)
```
If `setit` is true, all EntityControl present in the scene will have their `lastpo` set to their position. If `setit` is false, all EntityControl present in the scene will have their position set to their `lastpos` and if their animid is -1 (None), their `rigid` gets their gravity disabled in kinematic mode.

```cs
public static void StopEntitiesMove()
```
Calls StopForceMove on all EntityControl present in the scene.

```cs
public static void ResetEntitySpeed()
```
Sets the `anim`.speed of all the following entities to 1.0:

- All `map`.`entities` that aren't null and have their `anim` not null
- All `playerdata`'s `entity`

```cs
public static void ResetEntitySpeed(bool all)
```
If `all` is false, calls the parameterless overload of this method.

If `all` is true, sets the `anim`.speed of all EntityControl present in the scene that satisfies all of the following conditions:

- Not `iskill`
- `originalid` that isn't negative (defined animid)
- `anim` not null

```cs
public static EntityControl FindEntity(int id)
```
Returns the first EntityControl present in the scene whose animid is `id` or null if no such entity exists.

```cs
public static bool AllActive(EntityControl[] e)
```
Returns true if all entities in `e` either doesn't have an `npcdata` or it has one with a `hit` of true, returns false otherwise.

```cs
public static void FixEntities()
```
Does the following on all EntityControl present on the scene except those whose `npcdata` is an NPC or Enemy with an `item`:

- Reset `overrideflip` to false
- Reset `overrideanim` to false
- Set `rigid` to have gravity without kinematic
- Set `onground` to true
- Reset `rigid`.velocity to Vector3.zero
- If `anim` isn't null, `anim`.`speed` is reset to 1.0

```cs
public static bool CheckActiveEntities(int[] ids)
```
Returns true if all `map`.`entities` whose index is in `ids` have an `npcdata`.`hit` of true, false otherwise. It is possible for an `ids` element to have this check inverted where an `npcdata`.`hit` of false is considered active by specifying the negative version of the index (this implies this can't be done with id 0).

```cs
public void ForceLoadSprites()
```
Start an FLS coroutine.

```cs
private static IEnumerator FLS()
```
Forces a reference to be held for one frame of every `map`.`entities`'s `sprite`.`sprite` by creating new GameObject with SpriteRenderer childed to the `map` at the `player` position. Once all the GameObject are created, a frame is yielded. After, all the GameObjects that were created are immediately destroyed.

This method is meant to be a loading optimisation done during the map loading process.

```cs
public static EntityControl[] GetEntities(int[] ids)
public static EntityControl[] GetEntities(int[] ids, EntityControl[] ads)
```
Return the array of entities where each element is a resolved entity id from `ids` using GetEntity(int). If `ads` isn't null, the `ads` entity are appended to the return array.

For the overload not taking a `ads` parameter, the value defaults to null.

```cs
public static bool ObjectsAreActive(int[] objects, bool any)
public static bool ObjectsAreActive(NPCControl[] objects, bool any)
```
If `any` is true, returns true if any `objects` have a `hit` value of true, returns false otherwise.

If `any` is false, returns true only if all `objects` have a `hit` value of true, returns false otherwise.

In the `int[]` overload, the integers reprents the map entity ids of the NPCControl to use so they must be valid map entity for the current map or an exception will be thrown.

```cs
public static void IgnoreRadius(Collider col, bool ignore)
```
This is UNUSED, but remains functional. If `ignore` is true, causes all NPCControl present in the scene to have their `scol` ignore `col`. If `ignore` is false, they are unignored instead.

```cs
public static bool AllOnGround(EntityControl[] entities)
```
UNUSED: No one calls this, but it remains functional.

If every entity in `entities` is `onground`, this returns true and it returns false otherwise.