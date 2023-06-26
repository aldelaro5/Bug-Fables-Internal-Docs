# Addfollower

Add the caller as a follower to a new [entity](../../../Entities/Entity.md) or add a new one via an [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) or a [flagvar](../../../Flags%20arrays/flagvar.md) slot containing it.

## Syntax

(1)

````
|addfollower,this|
````

(2)

````
|addfollower,this,true|
````

(3)

````
|addfollower,follower|
````

(4)

````
|addfollower,var,flagvar|
````

## Parameters

### `follower`: int

The follower's [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) to add. This must be a valid [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) or an exception will be thrown.

### `var`

The presence of this parameter indicates to operate in syntax (4). Any other value will be interpreted as `follower` which may cause an exception to be thrown.

### `flagvar`: int

The [flagvar](../../../Flags%20arrays/flagvar.md) slot containing the follower's [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) to add. This must be a valid [flagvar](../../../Flags%20arrays/flagvar.md) slot or an exception will be thrown.

### `this`

The presence of this parameter indicate to operate in syntax (1) or (2) which adds the caller to the follower by replacing its entity entry in the process. Any other value will be interpreted as `follower` which may cause an exception to be thrown.

### `true`

Indicate to operate in syntax (2) which, after the caller is added as follower, will shrink the textbox, start to move the party and followers into their regular formation and grow the textbox again. Any other value is the same as not sending the parameter which will not do these additional steps.

## Remarks

The way followers work is there are 3 lists/arrays involved:

* `extrafollowers`: The list saved on the [Save File](../../../Save%20File.md) that dictates which followers to consider following the last member of the party.
* `map.canfollowID`: An array generated on a map's `Start` (specifically `AreaSpecific`) that acts as a whitelist to `extrafollowers`. This whitelist enforces who can follow the party on the first iteration of the map's `LateUpdate`. The TestRoom is however exempt from this.
* `map.tempfollowers`: The list of non party followers currently following the party. This represents all the non party active followers.

Adding a follower is done in 2 ways:

* Adding from an existing [entity](../../../Entities/Entity.md): This is what syntax (3) and (4) does. How it works is it destroys the caller's [entity](../../../Entities/Entity.md), but creates it back this time following the last member of the follower chain (in the order player, party members and existing `map.tempfollowers`). This also adds the follower to `extrafollowers` and `map.tempfollowers`. which makes the follower active immediately.
* Adding from [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md): This is what syntax (1) and (2) does. All this do is add the [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) to `extrafollowers`, but nothing else. This means the follower isn't going to be active until a map load is performed which implies the whitelisting of `map.canfollowID` needs to allow the follower. This is only used for troubleshooting purposes in the TestRoom.
