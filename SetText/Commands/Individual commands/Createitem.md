# Createitem

Create an item [Entity data](../../../TextAsset%20Data/Entity%20data.md) of a specific [Items](../../../Enums%20and%20IDs/Items.md), [Medal](../../../Enums%20and%20IDs/Medal.md) or a Crystal Berry with configurable position and timer before disappearing.

## Syntax

(1)

````
|createitem,itemtype,itemid|
````

(2)

````
|createitem,itemtype,itemid,xposition,yposition,zposition|
````

(3)

````
|createitem,itemtype,itemid,xposition,yposition,zposition,timer|
````

## Parameters

### `itemtype`: int

The type of the item. This must be a valid int or an exception will be thrown. Here are the defined types available:

* `0`: Standard item
* `1`: Key item
* `2`: Medal
* `3`: Crystal Berry
  The type influences the specific behaviors when the item is collected and how to render it. Any other value has undefined behaviors.

### `itemid`: int

The [Items](../../../Enums%20and%20IDs/Items.md) or [Medal](../../../Enums%20and%20IDs/Medal.md) id. This must be a valid [Items](../../../Enums%20and%20IDs/Items.md) or [Medal](../../../Enums%20and%20IDs/Medal.md) id or an exception will be thrown when rendering the [Entity data](../../../TextAsset%20Data/Entity%20data.md).

### `xposition`: float

The x position of where to place the [Entity data](../../../TextAsset%20Data/Entity%20data.md). This must be a valid float or an exception will be thrown. The default value is the x position of the player.

### `xposition`: float

The y position of where to place the [Entity data](../../../TextAsset%20Data/Entity%20data.md). This must be a valid float or an exception will be thrown. The default value is the y position of the player.

### `xposition`: float

The z position of where to place the [Entity data](../../../TextAsset%20Data/Entity%20data.md). This must be a valid float or an exception will be thrown. The default value is the z position of the player.

### `timer`: int

The amount of frames before the item disappear. This must be a valid int or an exception will be thrown. The default value is 300.

## Remarks

This will create the item without a velocity and it will also prepend the name with `Fixed` which gives the [Entity data](../../../TextAsset%20Data/Entity%20data.md) the Fixed modifier. Additionally, the RigidBody's constraints of the [Entity data](../../../TextAsset%20Data/Entity%20data.md) are frozen and the insideid field set to the current one.
