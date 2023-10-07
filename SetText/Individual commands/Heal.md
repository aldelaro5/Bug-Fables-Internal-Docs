# Heal

Heals the party fully or only heal the party's TP.

## Syntax

(1) (Fully heals the party)

````
|heal|
````

(2)

````
|heal,tp|
````

## Parameters

### `tp`

This value tells to operate in syntax (2) which heals only the party's TP and shows the HUD after the heal. Any other value will have the healing done like syntax (1), but it will still show the HUD after the heal.

## Remarks

Syntax (1) heals both the HP and TP of the party to their max value with particles and healing sound. Syntax (2) only heals the TP, but it does so without particles and healing sound.
