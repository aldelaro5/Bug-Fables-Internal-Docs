# Face

Instruct the party or an [Entity data](../../../TextAsset%20Data/Entity%20data.md) to face towards another one and optionally have that [Entity data](../../../TextAsset%20Data/Entity%20data.md) also face towards back.

## Syntax

(1)

````
|face,party,entity2|
````

(2)

````
|face,entity1,entity2|
````

(3)

````
|face,entity1,entity2,true|
````

## Parameters

### `party`

This value indicates to operate with syntax (1) and will have each party's [Entity data](../../../TextAsset%20Data/Entity%20data.md) face towards `entity2`. Any other value will be interpreted as `entity1`.

### `entity1`: int | string

The [Entity data](../../../TextAsset%20Data/Entity%20data.md) to instruct to face towards `entity2`. The int form is a regular [Entity id](../Entity%20id.md) and its restriction for valid values applies. The string is only applicable if caller isn't null and it will be interpreted as the int form otherwise. This form can be one of the following:

* `this`: Refers to the [tailtarget](../../Notable%20local%20variable/tailtarget.md). This must not be null or an exception will be thrown.
* `caller`: Refers to the caller. This must not be null or an exception will be thrown.
* Anything else: if the string is in the [Define](Define.md) list, it will refer to it. Otherwise, the value will be interpreted as the int form.

### `entity2`: int | string

The target [Entity data](../../../TextAsset%20Data/Entity%20data.md) that `entity1` will face towards. The int form is a regular [Entity id](../Entity%20id.md) and its restriction for valid values applies. The string is only applicable if caller isn't null and it will be interpreted as the int form otherwise. This form can be one of the following:

* `this`: Refers to the [tailtarget](../../Notable%20local%20variable/tailtarget.md). This must not be null or an exception will be thrown.
* `caller`: Refers to the caller. This must not be null or an exception will be thrown.
* Anything else: if the string is in the [Define](Define.md) list, it will refer to it. Otherwise, the value will be interpreted as the int form.

### `true`

Indicate to have `entity2` also face towards `entity1`. Any other value will be ignored.

## Remarks

In syntax (2) and (3), `entity1` will only change its flip to face towards `entity2`, but on syntax (1) and on syntax (3) when `entity2` faces towards `entity1`, FaceTowards is used instead.
