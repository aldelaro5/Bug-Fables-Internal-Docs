# Giveitem

Give an [items](../../../Enums%20and%20IDs/Items.md), [Medal](../../../Enums%20and%20IDs/Medal.md), Crystal Berry or berries to the player where its identification or amount is specified directly or is retrieved from a [flagvar](../../../Flags%20arrays/flagvar.md) with proper animations, presentations and checks that this action does not exceed the maximum amount of standard [items](../../../Enums%20and%20IDs/Items.md) if it is one.

## Syntax

(1)

````
|giveitem,type,itemid,redirect|
````

(2)

````
|giveitem,type,itemid,redirect,entityid|
````

(3)

````
|giveitem,type,var,flagvar,redirect|
````

(4)

````
|giveitem,type,var,flagvar,redirect,entityid|
````

(5) (Only use for testing purposes)

````
|giveitem,all|
````

## Parameters

### `type`: int

The type of the item to add. This must be a valid int or an exception will be thrown.

Here are the different values available:
Value | Type
--------- | --------
-1 | Berries
0 | Standard item
1 | Key item
2 | Medal
3 | Crystal Berry

Any other value will cause undefined behaviors.

### `itemid`: int

The item id whose meaning depends on `itemtype`. This must be a valid int or an exception will be thrown.

* If `itemtype` is berries, this corresponds to the amount of berries to add to the current amount (it can be negative which will decrease the amount).
* If `itemtype` is standard or key [Items](../../../Enums%20and%20IDs/Items.md), this corresponds to the [Items](../../../Enums%20and%20IDs/Items.md) id to give and it must be a valid [Items](../../../Enums%20and%20IDs/Items.md) id or an exception will be thrown.
* If `itemtype` is a [Medal](../../../Enums%20and%20IDs/Medal.md), this corresponds to the medal id to give and it must be a valid [Medal](../../../Enums%20and%20IDs/Medal.md) id or an exception will be thrown. (Note:  `itemid` is overridden to the next medal in [flagstring](../../../Flags%20arrays/flagstring.md) 13 (the MYSTERY? [Medal](../../../Enums%20and%20IDs/Medal.md) id list) if [flags](../../../Flags%20arrays/flags.md) 681 (MYSTERY? is active) is true).
* If `itemtype` is a Crystal Berry, this corresponds to the [crystalbfflags](../../../Enums%20and%20IDs/crystalbfflags.md) slot to enable and it must be a valid [crystalbfflags](../../../Enums%20and%20IDs/crystalbfflags.md) slot or an exception will be thrown.

### `redirect`: int

The [Dialogue line id](../Dialogue%20line%20id.md) to continue processing at after giving the item if it is given. This must be a valid [Dialogue line id](../Dialogue%20line%20id.md) or an exception will be thrown.

### `entityid`: int

The [Entity id](../Entity%20id.md) used to perform the animation of getting the item and to determine where to render the item sprite. This must be a valid [Entity id](../Entity%20id.md) or an exception will be thrown. The default value is -1 (the first player's [Entity data](../../../TextAsset%20Data/Entity%20data.md) which is the current lead).

### `var`

The presence of this parameter indicates to operate in syntax (3) or (4) which allows to retrieve the `itemid` from a [flagvar](../../../Flags%20arrays/flagvar.md) instead. Any other value will be interpreted as `itemid` which may cause an exception to be thrown.

### `flagvar`: int

The [flagvar](../../../Flags%20arrays/flagvar.md) slot to get the `itemid` from. This must be a valid [flagvar](../../../Flags%20arrays/flagvar.md) slot or an exception will be thrown.

### `all`

The presence of this parameter indicates to operate in syntax (5) which will add all [Items](../../../Enums%20and%20IDs/Items.md) in the standard inventory and nothing else without any checks with amount of standard items allowed. Any other value will be interpreted as the `type` which may thrown an exception.

## Remarks

This is a broader version of [Additemtoss](Additemtoss.md), but unlike it, it will not prompt to toss a standard item if giving it would exceed the maximum amount allowed in the inventory. If this happens with this command, this command will do nothing instead which means that if the prompt is desired under that condition, use [Additemtoss](Additemtoss.md) instead. Otherwise, this command provides more flexibility.

This commands resumes processing at the start of the input string to accommodate its replacement if the item was given, otherwise, processing resumes normally.

## About the inventory limit of standard items

It is not safe to use this command if it is possible to exceed the maximum amount because while the command processing gets stopped, it's done after the description box is created if applicable which leads to improper rendering. It is recommended to check this using [Checkinvqtd](Checkinvqtd.md) before processing this command or use [Additemtoss](Additemtoss.md).

Syntax (5) bypasses this check however giving all [Items](../../../Enums%20and%20IDs/Items.md) in the standard items inventory allowing to go over the maximum amount. This is done for testing purposes: its only usage is in the TestRoom and it has no other logic than that unlike the other syntax forms.

## [flagstring](../../../Flags%20arrays/flagstring.md) and [flagvar](../../../Flags%20arrays/flagvar.md) used

This commands writes to the following:

* [flagstring](../../../Flags%20arrays/flagstring.md) 0: The [Items](../../../Enums%20and%20IDs/Items.md) or [Medal](../../../Enums%20and%20IDs/Medal.md) name if applicable or menu menutext 112 (`Crystal Berry`) if `itemtype` is set to Crystal Berry. This is used to render the item get text.
* [flagstring](../../../Flags%20arrays/flagstring.md) 1: The default article (`a`) or the [Items](../../../Enums%20and%20IDs/Items.md) or [Medal](../../../Enums%20and%20IDs/Medal.md) article if applicable which is used for rendering the item get text.
* [flagvar](../../../Flags%20arrays/flagvar.md) 10: The buying price of the [Items](../../../Enums%20and%20IDs/Items.md) or [Medal](../../../Enums%20and%20IDs/Medal.md) if applicable
* [flagvar](../../../Flags%20arrays/flagvar.md) 14: This is incremented if `itemtype` is set to Crystal Berry.
* [flags](../../../Flags%20arrays/flags.md) 31: if `itemtype` is set to Medal, this is set to true if it was false before because this command will give the medal tutorial if it wasn't given.

## Command's logic

This command's logic behaves very differently based on the `itemtype` and the resolved `itemid` because the different types and ids requires different presentations and different behaviors.

### Description box rendering

A description box will be created if `itemtype` isn't berries or Crystal Berries and the caller isn't a CaravanBadge or a Shop (for key items, it also needs to not be Game Tokens and not be in an interior)

* Create a pure white standard 9box of type 0 relatively positioned at (0.0, -4.4, 10.0) of the GUICamera with a size of (11.0, 3.0) with a sort order of -3 with a grow animation and assign it to descwindow
* Sets [flagvar](../../../Flags%20arrays/flagvar.md) 10 to the buying price price of the [items](../../../Enums%20and%20IDs/Items.md) or [medal](../../../Enums%20and%20IDs/Medal.md)
* Calls SetText in non [Dialogue mode](../../Dialogue%20mode.md) with the input string |[single](Single.md)\||[singlebreak](Singlebreak.md),`itemdescbreak`\| where `itemdescbreak` is 10.5 on `English` and 9.9 on any other language followed by the [items](../../../Enums%20and%20IDs/Items.md) or [medal](../../../Enums%20and%20IDs/Medal.md) description:
  * Font is `BubblegumSans`
  * No line breaks (this is already manually controlled via the input string)
  * No tridimensional
  * Position is (-5.2, 0.65, 1.0)
  * No camera offset
  * Size is (0.675, 0.675)
  * Parent is the descwindow created earlier
  * No caller

### Other rendering setup

* Setup a shrink animation to the textbox
* Stop the `entity` rigidBody's velocity
* Setup an array of 3 SpriteRender
* Assign the first element of the array to the sprite of the item named `tempitem` (the sprite is gui sprite 83 (the Crystal Berry icon) if `itemtype` is Crystal Berry and for berries, it's the berry sprite with the highest value that fits in the amount from `itemid`)
  * This is positioned at the `entity`'s location + (0.0, 2.75, -0.1) at the same angles than the camera.
  * The layer is set to 14
* Assign the second element of the array to a new object named `fauxmessage` at (0.0, 3.0, 10.0) from the GUICamera using the [Boxstyle](Boxstyle.md) 4
* Create a new SpriteRender named `back` childed to `tempitem` at (0.0, 0.0, 0.2) with no rotation using gui sprite 85 (this is the star shaped background rendered behind the item)
* Set the layer of `back` to 14 and its scale to Vector3.zero with a grow animation (this effectively setup a grow while the sprite is hidden at first).
* Set the color of `back` to 00B2B2 if `itemtype` is Standard item, FF4C66 if it's Key items and FF7F00 for everything else

### `itemtype` specific handling

The `itemtype` have their own specific logic:

* berries: show the berries counter HUD and increase the current amount by `itemid2` clamped from 0 to 999 and set [flagvar](../../../Flags%20arrays/flagvar.md) 0 to `itemid2`, otherwise, hide the HUD counter.
* Standard or Key items: set [flagstring](../../../Flags%20arrays/flagstring.md) 0 to the [Items](../../../Enums%20and%20IDs/Items.md)'s name and [flagstring](../../../Flags%20arrays/flagstring.md) 1 to the [items](../../../Enums%20and%20IDs/Items.md)'s article then add the [items](../../../Enums%20and%20IDs/Items.md) to its appropriate inventory.
* Medal: set [flagstring](../../../Flags%20arrays/flagstring.md) 0 to the [Medal](../../../Enums%20and%20IDs/Medal.md)'s name and [flagstring](../../../Flags%20arrays/flagstring.md) 1 to the [Medal](../../../Enums%20and%20IDs/Medal.md)'s article then add the [Medal](../../../Enums%20and%20IDs/Medal.md) as unequipped.
* Crystal Berry: Activate the [crystalbfflags](../../../Enums%20and%20IDs/crystalbfflags.md) slot of `itemid`, increment [flagvar](../../../Flags%20arrays/flagvar.md) 14 and set [flagstring](../../../Flags%20arrays/flagstring.md) 0 to menutext 112 (`Crystal Berry`).

### Presentation

* Call SetText in non [Dialogue mode](../../Dialogue%20mode.md) with the input string |[sort](Sort.md),1||[center](Center.md)\||[halfline](Halfline.md)\| followed by menutext 110 (`You got |currency,var,0|!`) if `itemtype` is Crystal berry or menutext 107 (`You got |string,1| |color,1||string,0||color,0|!`) otherwise at Vector3.zero and `fauxmessage` as the parent.
* Set the overrideanim of `entity` to true and its animstate to 4
* Yield for 0.45 seconds (this lets the grow animation plays).
* Assign the third element of the SpriteRenderer array to a new sprite called `fauxcursor` where the parent is `fauxmessage` using the standard cursor sprite without scaling at (6.0, -2.0, 1.0).
* Set up `fauxcursor` to bounce like a normal cursor would
* Grab the waitinput lock, yield one frame and then yield until the lock is released.

### Cleanup

* Destroy all the SpriteRenders created earlier.
* Set the textbox to grow.
* Reset the [Text advance](../../Related%20Systems/Text%20advance.md)'s `skiptext` to false.
* Set the `entity`'s animoverride to false and its anim state to 0

### Redirection

* If `itemtype` is Medal and [flags](../../../Flags%20arrays/flags.md) 31 is false (the player has yet to receive the medal tutorial):
  * Assign a string with the value |[tail](Tail.md),null||[Destroydescbox](Destroydescbox.md)\||[blank](Blank.md)\||[boxstyle](Boxstyle.md),4| followed by common dialogue 31 (the medal tutorial) followed by |[flag](Flag.md),31,true||[break](Break.md)\||[Gettail](Gettail.md),`tailid`\||[boxstyle](Boxstyle.md),0| where `tailid` is the current [tailtarget](../../Notable%20local%20variable/tailtarget.md).
* Assign the input string to an [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) version of `redirect` prepanded with |[Destroydescbox](Destroydescbox.md)\||[blank](Blank.md)\| all prepanded with the medal tutorial if it was assigned above.
