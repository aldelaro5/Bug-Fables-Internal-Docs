# Questbreak

Signal the char loop that processing is completed once this iteration ends under certain conditions (this command is broken, see the remarks section for details).

## Syntax

(1) (This will throw an exception, do not use)

````
|questbreak|
````

(2)

````
|questbreak,condition|
````

## Parameters

### `condition`: `true` | `false`

This must be a valid bool or an exception will be thrown.

## Remarks

The exact conditions depends if the `questboardobj` of the [Quest Board List Type](../../ItemList/List%20Types%20Group%20Details/Quest%20Board%20List%20Type.md) is active or not:

* If it's active, the signal to the char loop is sent if `condition` is true AND syntax (1) is used. Since both of these conditions are incompatible, the former's check will throw an exception.
* If it's inactive, the signal to the char loop is sent if syntax (2) is used (this will throw an exception if syntax (1) is used because it still checks for the value of `condition` which wouldn't exist).
