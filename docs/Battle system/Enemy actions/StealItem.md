# StealItem
A method that has an enemy party member attempt to steal an [item](../../Enums%20and%20IDs/Items.md) from the player's standard items inventory and returns a bool indicating if an item was actually stolen or not.

```cs
private bool StealItem(int enemyid, bool always, bool noanim)
```

There are 2 overloads that ends up calling the one above. The following one sends false as `noanim`:

```cs
private bool StealItem(int enemyid, bool always)
```

And this one one send false as `always` and false as `noanim`:

```cs
private bool StealItem(int enemyid)
```

## Parameters

- `enemyid`: The `enemydata` of the enemy party member that will attempt the stealing
- `always`: If true, the enemy party member will always attempt to steal if it can. If false, it will only do it if `commandsuccess` is false (meaning the last block or action command was failed)
- `noanim`: If true, the enemy party member won't have their animstate changed to 27 (`ItemWalk`) during the stealing if it happens

## Steal conditions
Here are all the conditions needed for the enemy party member to actually steal:

- Either always is true or `commandsuccess` is false
- `items[0]` contains at least one standard item
- The enemy party member doesn't already have a `holditem`

## Procedure

- If the `SecurePouch` [medal](../../Enums%20and%20IDs/Medal.md) is equipped, false is returned early and no stealing happens
- If the conditions needed to steal mentioned above aren't fufilled, false is returned. Otherwise:
    - If noanim is false, the enemy party member animstate is set to 27 (`ItemWalk`)
    - A random valid [item](../../Enums%20and%20IDs/Items.md) id among `items[0]` is generated which will be the item that will be stolen
    - CreateHeldItem is called with the item id which initialises the enemy party member's `helditem` to be a new SpriteRenderer corresponding to the `itemsprites` from the id generated childed to the enemy party member's `sprite` using the `spritemar` material on layer 14 (`Sprite`) with a FaceCamera component. It also sets the `holditem` of the enemy party member to the item id generated
    - The item element is removed from `items[0]`
    - `ItemStolen` sound plays
    - true is returned
