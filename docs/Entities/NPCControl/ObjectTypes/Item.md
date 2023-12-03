# Item
A collectable [item](../../../Enums%20and%20IDs/Items.md), [medal](../../../Enums%20and%20IDs/Medal.md) or crystal berry bound to a [crystalbfflags](../../../Enums%20and%20IDs/crystalbfflags.md) slot whose item id is the entity.`animid` field. This extends the logic of an [item entity](../../EntityControl/Item%20entity.md).

## Data Arrays
- `data[0]`: The [item type](../../EntityControl/Item%20entity.md#item-types). NOTE: if the `animid` is a money item id (`MoneySmall`, `MoneyMedium` or `MoneyBig`), this should always be 0 or 1 (standard or key item) because if it's not, the collection logic won't work as expected
- `data[1]`: If it's not negative, an [event](../../../Enums%20and%20IDs/Events.md) id that will be processed in an [event command](../../../SetText/Individual%20commands/Event.md) at the end of the input string when collecting the item. This is optional, nothing happens if it doesn't exist
- `data[2]`: If it's not 0, the item cannot be caught using the player `beemerang`. This is optional, the item can be caught if it doesn't exist.
- `data[3]`: The [crystalbfflags](../../../Enums%20and%20IDs/crystalbfflags.md) slot if applicable (the `animid` is 3)

## Additional data
- `activationflag`: If `data[0]` isn't 3 (it's not a Crystal Berry) and the value is above 0, the [flag](../../../Flags%20arrays/flags.md) whose slot is the value will be set to true when collecting the item
- `regaionalflag`: If `data[0]` isn't 3 (it's not a Crystal Berry) and the value isn't negative, the [regionalflag](../../../Flags%20arrays/Regionalflags.md) whose slot is the value will be set to true when collecting the item

## HasHiddenItem
For an entity.`animid` of 1 or 2 (key item or medal), the return value is the `activationflag` being positive and the coresponding [flag](../../../Flags%20arrays/flags.md) slot being false.

## MapControl.CreateEntities
- entity.`animstate` is set to the entity.`animid`
- entity.`animid` is set to `data[0]`
- entity.`item` is set to true (this makes it an [item entity](../../EntityControl/Item%20entity.md))
- entity.`sprite` local position is set to (0.0, 0.5, 0.0)

## EntityControl.CreateItem
This is a special way to create an [EntityControl](../../EntityControl/EntityControl.md) that will also give it this object type and set it as an [item entity](../../EntityControl/Item%20entity.md). It's a public static method on EntityControl meaning it can be called by anyone to create an item object out of thin air. It returns the newly created NPCControl and it takes in the starting position, the item type, the item id, the starting velocity and the amount of time in frames this should be collectable.

The very first thing this does is call [CreateNewEntity](../../EntityControl/EntityControl%20Methods.md#createnewentity) with the name `tempitem` and add an NPCControl to it.

The rest is logic specific to CreateItem:

- `objecttype` is set to `Item`
- `entitytype` is set to `Object`
- entity.`item` is set to true making it an [item entity](../../EntityControl/Item%20entity.md)
- entity.`animid` is set to the item type
- entity.`animstate`, entity.`itemstate` and entity.`basestate` are all set to the item id
- `touchcooldown` is set to 30.0 (prevents to pickup the item in the first 30.0 frames)
- All collisions between entity.`ccol` and player.entity.`ccol` are ignored (the item will rely on the `secondcoll` created later)
- The position and entity.`startpos` are set to the starting position
- The NPCControl gets childed to the map
- `insideid` is set to the current one
- `timer` is set to the timer sent
- `tempobject` is set to true (prevents the Crystal Berry SetUp logic from applying as this method already does it partially and EntityControl's startup will do the rest via [CheckSpecialID](../../EntityControl/Notable%20methods/CheckSpecialID.md)) 
- `data[0]` is set to the item type
- `bounces` is set to 0
- entity.`onground` is set to true
- entity.`startvelocity` is set to the starting velocity
- If the item type is 3 (Crystal Berry), `data[0]` is overriden to be the item id instead (which simulates how this happens normally with `data[3]`) and entity.`animid` is set to 43 (`CrystalBerry`)

## Start
Normally, any objects has their entity.`ccol` disabled on Start, but this one is an exception: it remains enabled.

Also, just like `SemiNPC` items, no changes to the entity.`rigid` mass happens so it is left to 1.0.

## SetUp
- entity.`onground` is set to false
- entity.`alwaysactive` is set to true
- entity.`ccol` bounciness is set to 0.75
- The layer is set to 0 (default)
- All collisions between entity.`ccol` and these entity's `ccol` are ignored:
    - Objects with tag `PFollower`
    - Any other Item objects (excludes `SemiNPC`)
    - Any playerdata except the first one

If the entity.`animid` is 3 (which means it's a crystal berry) and `tempobject` is false, then this is setup as a crystal berry which starts by setting `data[0]` to `data[3]` (meaning `data[0]` is now the Crystal Berry id which is fine because entity.`animid` already tells it's a Crystal Berry). If it was obtained (the id is contained in `data[3]`), then the entity.`iskill` is set to true. Otherwise, adjustements on the entity are performed:

- entity.`animstate` is set to the `regionaflag` NOTE: this is likely an error, but it doesn't matter because the [crystalbfflags](../../../Enums%20and%20IDs/crystalbfflags.md) slot is already saved in `data[0]` and it is that value that will be used
- entity.`sprite` local position is set to Vector3.zero
- AddModel is called with path `Prefabs/Objects/CrystalBerry` without offset
- entity.`spin` is set to (0.0, 2.0, 0.0)
- entity.`item` is set to true which makes it an [item entity](../../EntityControl/Item%20entity.md)
- entity.`rigid` gravity is enabled

AddPlayerTrigger is called which will initialise `secondcoll` to a new trigger CapsuleCollider with the same properties than the entity.`ccol` and ignores all collisions between the player.entity.`ccol` and the entity.`ccol`.

Just like `SemiNPC` items, the `colliderheight` is set to 1.0 and the entity.`ccol` center to (0.0, 0.5, 0.0). This reduces the height of the collider to more snuggly fit the item shape.

## Update (active and not `trapped`)
This section applies for any [item entity](../../EntityControl/Item%20entity.md), but it effectively apply to this object type. Unlike `SemiNPC` items however, this gets the full version of the logic.

If the `beerang` isn't null (meaning the [beemerang](Beemerang.md) caught this item), the following occurs:

- The `timer` is set to 300.0 if it was -1
- The absolute position is set to the `beerang` one + Vector3.Up
- The entity's Unifx gets called if it was a `fixedentity`

All collisions gets ignored between the `secondcoll` and the player.entity.`ccol` if the player is present and the `touchcooldown` hasn't expired yet. Otherwise, if the `touchcooldown` isn't exactly -9999.0, the collisions gets unignored and `toochcooldown` gets set to -9999.0 (so it doesn't unignore again). If neither happened, the entity.`ccol` is enabled.

If the entity.`onground` is true, [Jump](../../EntityControl/EntityControl%20Methods.md#jump) is called on it with the absolute value of its current `rigid` y velocity. It will also increment `bounces` if it hasn't reached 3 yet on top of playing the `ItemBounce0` sound effect (or `ItemBounce1` if it's a Crystal Berry) if the `startlife` is above 15 frames.

If by then, `bounces` has reached 3, [StopForceMove](../../EntityControl/EntityControl%20Methods.md#stopforcemove) is called on the entity.

If the entity.`sprite` is present, then the `timer` logic proceeds:

- If the `timer` has yet to expire and we aren't in a `minipause` or `pause`, the timer is decremented according to the frametime, but it is clamped from 0.0 to infinity. Otherwise, if the `timer` expired, this GameObject gets destroyed
- If the `timer` is between -1.0 and 100.0 exclusive and we aren't in a `minipause` or `inevent`, the entity.`sprite` enablement gets toggled. Otherwise, the entity.`sprite` is enabled. This logic blinks the sprite when less than 100 frames are left on the `timer`

## OnTriggerEnter if the other collider is the player
If we aren't in a `pause` or `minipause`, the `insideid` matches the current one and the `timer` is exactly -1.0 or above 1.0, a CheckItem coroutine is started and `collisionammount` is incremented.

## OnTriggerEnter main segment
The `beerang` is set to the other transform if all of the following are true (this makes the `Beemerang` catch the item):

- The other gameObject tag is `BeeRang` (meaning the other collider is the [Beemerang](Beemerang.md))
- `data[2]` is either not present or it is and it's value is 0 (meaning it's a standard item)
- `beerang` was null (the `Beemerang` didn't already caught an item)
- We aren't in a `pause`, or `minipause` and [message](../../../SetText/Notable%20states.md#message) is released
- The player `beemerang` is present and its `hit` is false (it's on the first half of its path)
- `insideid` matches the current one

## OnTriggerStay (when the NPCControl is a [JumpSpring](JumpSpring.md))
If the other collider is an [item entity](../../EntityControl/Item%20entity.md) and is an Object (which can only happen with this object type under normal gameplay), the item is allowed to move upward using the spring if its `data[0]` is 0 or its `data[1]` is 1 and the additional check it provides is fufilled.

## CheckItem
This is a private coroutine that is only involved with this object type. Its job is to setup and confirm that the item can be collected followed by a bunch of logic performing the actual collection of the item.

- The game determines if this is a special currency item. This is the case if entity.`animstate` (the item id) is 6 (`MoneySmall`), 7 (`MoneyMedium`) or 186 (`MoneyBig`). This will be used for many checks later
- All frames are yielded while we are in a `pause` or `minipause` and `hit` is false (the item isn't collected). This never happens under normal gameplay because the caller (OnTriggerEnter) already makes sure that we aren't in a `pause` or `minipause`
- CancelAction is called on the player for anything other than standard money items
- instance.`itempicked` is set to true (controls some logic related to [JumpSpring](JumpSpring.md) objects and [Shop](../Interaction/Shop.md) / [CaravanBadge](../Interaction/CaravanBadge.md) interactions)
- If it's not a money item, player.entity.`icooldown` is set to 0.0 (removes any invicibily frames)
- `hit` is set to true (indicating the item is being collected)
- `tossed` is set to false
- The current player.entity.`flip` is saved locally as it may be restored later
- All collisions between the item entity and the player.entity are ignored for any colliders
- The item collection occurs if the `touchcooldown` expired, the item isn't `trapped` and we aren't in a `minipause`. NOTE: this seems like a bug because if we don't collect here, the colliders aren't unignored, but in practice, MainManager.IgnoreColliders doesn't work correctly meaning this isn't an issue in practice. See the sections below for more details. The collection logic depends on if the item is a standard or key money item.
- instance.`itempicked` is set to false

### Standard or key money items collection
- The [regionaflag](../../../Flags%20arrays/Regionalflags.md) whose slot is `regionalflag` is set to true if it wasn't negative
- The [flag](../../../Flags%20arrays/flags.md) whose slot is `activationflag` is set to true if it was above 0
- `touchcooldown` is set to 999999.0
- instance.`money` is incremented by the amount the item is worth in currency (1 for `MoneySmall`, 5 for `MoneyMedium` and 20 for `MoneyBig`)
- instance.`showmoney` is set to 150.0 frames which shows the berry count HUD temporarily
- The `Money` sound is played
- entity.`spin` is set to (0.0, 30.0, 0.0)
- A BerryBounce coroutine is started. This just renders the visual effect of collecting the berry by making its sprite go up and scale down
- `dummy` is set to true (disables most update cycles logic as BerryBounce needs some time before destroying the item so most updates needs to not run while this happens)
- instance.`money` is clamped from 0 to 999
- entity.`iskill` is set to true

### Any non money items collection
This logic is quite complex and has different behaviors depending on the entity.`animid` and entity.`animsate`.

#### Hold the player until they get `onground`
The logic first starts to handle the case where the player is still in the air:

- player.`lockkeys` is set to true (disable most input processing on PlayerControl)
- As long as player.entity.`onground` is false:
    - player.entity.`rigid` x/z velocity is zeroed out
    - instance.`minipause` is set to true
    - The position of the item is set to offscreen at (0.0, 999.0, 0.0)
    - If we have been on this yield loop for 300.0 frames or more, we force exit it by setting the player position to its `lastpos` with DeathSmoke particle playing at the player position
    - A local framecounter is incremented by `framestep` which only takes effect once it reaches 300.0 as outlined in the step above
    - A frame is yielded
- player.`lockkeys` is set to false (unlocks most input processing of PlayerControl)

#### Item collection perparation
From there, the actual item collection can now take place:

- `beerang` is set to null (detaches the item off the [Beemerang](Beemerang.md) if it was attached)
- entity.`spin` gets zeroed out
- entity.`sprite` angles gets set to Vector3.zero
- CancelAction is called on the player
- entity.`ccol` gets disabled
- entity.`rigid` gets its gravity disabled with all constraints frozen
- The item gets childed to the player.entity.`sprite` with a local position of (0.0, 2.25, -0.1)
- [flagvar](../../../Flags%20arrays/flagvar.md) 0 gets set to entity.`animstate` (the item id)
- A `guisprites[85]` (star shaped back icon) gets created with name `back` childed to entity.`sprite` with the same material as entity.`sprite` without angles with local position (0.0, 0.0, 0.2) with scale of Vector3.zero with a DialogueAnim and layer 14 (`Sprite`)
- [flagstring](../../../Flags%20arrays/flagstring.md) 1 gets set to `menutext[125]` (`a`).

#### Item type specifics
What happens next depends on entity.`animid` (the item type) and it mainly sets the `back` material color, sets `flagstring` 0, `flagstring` 1 and other exclusive logic to an item type:

- 0 or 1 (standard or key [item](../../../Enums%20and%20IDs/Items.md)): 
    - `flagstring` 0 is set to the name and `flagstring` 1 to the prepender of the corresponding item defined in [items data](../../../TextAsset%20Data/Items%20data.md#items-data) using the entity.`animstate` as the item id. 
    - It also sets the `back`'s material color to 00B2B2 (cyan) if it's a standard item, FF4C66 (bright red) if it's a key item
- 2 ([medal](../../../Enums%20and%20IDs/Medal.md)):
    - If `flags` 681 (MYSTERY? is enabled) and entity.`animstate` isn't 59 (it's not `Extra Freeze`) or it is, but `flags` 696 (MYSTERY? file started on 1.1.x) is false:
        - The next mystery medal id is obtained and the value is set to entity.`basestate`, entity.`itemstate`, entity.`animstate` and `flagvar` 0
        - entity.`overridemovesmoke` is set to true and UpdateItem is called on the entity (to ensure the actual sprite gets updated)
    - The `back` material color gets set to FF7F00 (bright orange)
    - `flagstring` 0 is set to a string obtained from taking `menutext[268]` and replacing `i` by the corresponding medal name from [medal data](../../../TextAsset%20Data/Medals%20data.md#badgename-data) using entity.`animstate` as the medal id and by replacing `m` by `menutext[159]` (`Medal`)
    - `flagstring` 1 is set to the corresponding prepander from [medal data](../../../TextAsset%20Data/Medals%20data.md#badgename-data) using entity.`animstate` as the medal id
- 3 (Crystal Berry):
    - The `back` local position is set to (0.0, 0.5, 0.0) and its material color to cyan
    - entity.`model` y scale is set to 0.1
    - [crystalbfflags](../../../Enums%20and%20IDs/crystalbfflags.md) whose slot is `data[0]` (used to be `data[3]` which is the slot number) is set to true
    - `flagstring` 0 is set to `menutext[112]` (`Crystal Berry`)
    - `flagvar` 14 (amount of Crystal Berries obtained) gets incremented

After this, for any type other than Crystal Berry, [CreateDescWindow](../Notable%20methods/CreateDescWindow.md) without shop is called.

For any type, the `ItemGetX` sound is played where X is the entity.`animid` (the item type).

#### [SetText](../../../SetText/SetText.md) call
The next portion setups a SetText call to actually receive the item. The SetText string is built with the following concatenated in order:

- `|lockmovement||boxstyle,4||halfline||spd,0||anim,-1,4||center|`
- `menutext[2]` (`You found |string,1| |color,1||string,0||color,1|!`)
- `|stopskip||fwait,0.45||break|`
- If it's not a Crystal Berry and `activationflag` is above 0, `|flag,activationflag,true|` is appended
- If it's not a Crystal Berry and `regionalflag` is not negative, `|regionalflag,regionalflag,true|` is appended
- `|additemtoss,`
- entity.`animid` (the item type)
- `,var,0|`
- If `flags` 31 is false and it's a medal (haven't received the medal tutorial while getting one), `|flag,31,true||tail,null||center,true||destroydescbox||goto,-32,break,end|` is appended
- If `flags` 108 is false and it's a Crystal Berry (haven't received the Crystal Berry tutorial while getting one), `|flag,108,true||tail,null||center,true||destroydescbox||goto,-88,break,end|` is appended
- If `data[1]` exists and isn't negative, `|event,` + `data[1]` + `|` is appended (a [break](../../../SetText/Individual%20commands/Break.md) command appears before the event one if any of the tutorial strings were appended)

SetText is called with the generated string in [dialogue mode](../../../SetText/Dialogue%20mode.md):

- [fonttype](../../../SetText/Notable%20states.md#fonttype) of `BubblegumSans`
- linebreak of instance.`messagebreak`
- No tridimensional
- No position offset
- No camera offset
- Size of Vector2.one
- No parent
- This NPCControl as the caller

#### SetText yield
This is the final part part of the collection phase:

- `timer` is set to -1 (so it never times out)
- entity.`overrideflip` is set to true
- As long as the [message](../../../SetText/Notable%20states.md#message) lock is active:
    - player.entity.`flip` is set to the value it had before this entire collection phase at the start of CheckItem
    - player.entity.[animsate](../../EntityControl/Animations/animstate.md) is set to 4 (`ItemGet`)
    - A frame is yielded
- If the item isn't `tossed` (it is only when [additemtoss](../../../SetText/Individual%20commands/Additemtoss.md) concluded the player wanted to toss the item), this gameObject is destroyed

## EntityControl.LateStart
Any object of this type will have [CreateFeet](../../EntityControl/EntityControl%20Methods.md#CreateFeet) called which gives the entity a ground detector in the `feet` field.

## Hazards.OnObjectStay
If the NPCControl passed has this type, it returns true which allows it to stay in collision with the Hazards.

## [SetText](../../../SetText/SetText.md)
This object type affects SetText if it's the caller:

- In an [end](../../../SetText/Individual%20commands/End.md) command, if the caller is an `Item`, [Death](../../EntityControl/Notable%20methods/Death.md) is called on the entity
- In the [cleanup phase](../../../SetText/Life%20Cycle.md#cleanup), if the caller is an object of this type and its `hit` is true (meaning it's been collected), its gameObject is destroyed.