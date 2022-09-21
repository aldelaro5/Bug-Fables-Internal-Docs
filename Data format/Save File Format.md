# Bug Fables Save Format
A save file in Bug Fables is a XOR encrypted regular text file. It is a very simple
symetric encryption scheme where each bytes is XORed with a number called the key.
This key is present in the TextAsset located at `Ressources/Data/Key` in the asset tree.

The operation to encrypt or decrypt are identical.

The file is composed of exactly 18 lines. There are 3 types of line format:
- Fields line: contain fields of various meaning, separated by ",".
- List line: contain a list of items, separated by "," unless specified otherwise.
- Array line: contain an array, each element is separated by "@" and each element is either a list or fields.

## Format differences between 1.0.x and 1.1.x
There were 2 changes done to the format in the 1.1.0 version of the game. Specifically, the amount of items in the flagvar and flags line have increased. When a 1.0.x save file is loaded into the game, it will load normally, but with the new items getting a default value (in this case, `false`). The game uses a specific flag (see document on [glabal flags](../Flags/Global%20flags.md) to determine if the file is in 1.0.x format or 1.1.x format. If it detects it's a 1.0.5 file, it will perform some directives to adapt it as some logic changes and flags changes impacted it. Once it is done, it will set the flag checked earlier to true which will prevent the game from performing the conversion twice. This works because the flag used is not used in 1.0.x so it will never be `true` until the conversioon is done in 1.1.x. This will only persist once a save in the game is performed at which point, the save file is permanently converted to a 1.1.x format.

## Sections of a save file
### Line 1 (Fields line) Header

This line serves as a header.

Position | Description | Type | Additional information
----------------------------|---------------|--------------|--------------------------------
0-2 | Save Position | Vector3 (float x3) | In the order X,Y,Z
3 | RUIGEE | Bool | Same as flag 613
4 | HARDEST | Bool | Same as flag 614
5 | FRAMEONE | Bool | Same as flag 615
6 | PUSHROCK | Bool | Same as flag 616
7 | MOREFARM | Bool | Same as flag 656
8 | MYSTERY? | Bool | Same as flag 681
9 | Filename | String | Same as flagstring 10

### Line 2 (Array line) Party memeber stats
This line corresponds to each party members stats (their BattleData).

Element | Description
---------|----------------
0 | Vi
1 | Kabbu
2 | Leif

If a party member is not present, its element is empty.

An array element contains the following fields:

Position | Description | Type | Additional information
---------------|----------------|--------------|---------------------------------------------------
0 | trueid | Int | [AnimID](../Enums%20and%20IDs/Anim%20IDs.md) of the party member, 0 - Vi, 1 - Kabbu and 2 - Leif
1 | HP | Int
2 | Max HP | Int
3 | Base HP | Int
4 | Attack | Int
5 | Base Attack | Int
6 | Defense | Int
7 | Base Defense | Int

### Line 3 (Field line) Global information and party information
This line contains global information about the party and the state of the game.

Position | Description | Type | Additional information
------------------|-------------------|------------------------------------------------------------------------|-----------------------------------
0 | Rank | Int
1 | Exp | Int
2 | Needed Exp for next rank | Int
3 | Max TP | Int
4 | TP | Int
5 | Berries count | Int
6 | Name of the current map | String | This is a string version of the [MapIDs](../Enums%20and%20IDs/Maps%20IDs.md)
7 | Current area's ID | Int | Refer to the [AreasIDs](../Enums%20and%20IDs/Areas%20IDs.md) document
8 | MP | Int
9 | Max MP | Int
10 | Number of items allowed in inventory | Int
11 | Number of items in storage allowed | Int | Normally set to 35 and never set again
12 | Hours of the play time clock | Int
13 | Minutes of the play time clock | Int
14 | Seconds of the play time clock | Int
15 | Save progress icons | Int | 0 - no icons, 1 - Ancient Mask, 2 - Ancient Tablet, 3 - Ancient Key, 4 - Ancient half, 5 - Elizant II, 6 - Flame Brooch and 7 - Wasp King's crown

### Line 4 (Array line) Medals shops pool

This line corresponds to the pool of medals each shop currently offers.

Element | Description
--------------------------|------------------------
0 | Medal shop at Ant City
1 | Shades's shop at the Underground Bar

Each array element contains a list:

Position | Description | Type | Additional information
-----------------|--------------|------------------|------------------------
ALL | Medal ID | Int | -1 - empty slot, Refer to the [Medals](../Enums%20and%20IDs/Medals%20IDs.md) document

The way it works is this list is ordered in such a way that the first medals are going
to be the ones displayed at the shop. For the Ant City medal shop, it's the first 3
and for Shade's shop, it's the first 2. An empty slot means there will be no medals
(which occurs if there isn't enough available medals).

### Line 5 (Array line) Medals available in medal shops

This line corresponds the different medals available in each medal shops for purchase.

Element | Description
-------------------------|-------------------------
0 | Medal shop at Ant City
1 | Shades's shop at the Underground Bar

Each array element contains a list:

Position | Description | Type | Additional information
------------------|-----------|-------|------------------------
ALL | Medal ID | Int | Refer to the [Medals](../Enums%20and%20IDs/Medals%20IDs.md) document

Unlike line 4, this is a list to simply indicate the medals IN STOCK for purchase.
It does not represent the current pool offered by the shop which is in line 4.

### Line 6 (Array line) Quests
This line contains information about the quests on the quests boards

Element | Description
---------------|--------------
0 | Open quests
1 | Taken quests
2 | Completed quests

Each array element contains a list:

Position | Description | Type | Additional information
------------------|-------------|-----------------|------------
ALL | Quest ID | Int | Refer to the [Quests](../Enums%20and%20IDs/Quests%20IDs.md) document

If the element is empty, there should only be one item in the list being the number 0 (No Quests).
> This includes ALL quests including main story ones.

### Line 7 (Array line) Items
This line contains information about the items in possession and in storage.

Element | Description
----------|----------------
0 | Items
1 | Key items
2 | Stored items

Each array element contains a list:

Position | Description | Type | Additional information
---------------|---------|---------|---------------------------
ALL | Item ID | Int | Refer to the [Items](../Enums%20and%20IDs/Items%20IDs.md) document

### Line 8 (Array line) Medals
This line contains the medals in possession as well as how it is equipped.

Each array element contains fields:

Position | Description | Type | Additional information
--------------------------|--------------|------------------------------------------|---------------------------
0 | Medal ID | Int | Refer to Appendix A
1 | Equip target | Int | -2 - Unequipped, -1 - Equipped to the party, everything else is the [AnimID](../Enums%20and%20IDs/Anim%20IDs.md) of whom has it equipped

### Line 9 (Array line) Samira songs
This line corresponds to all the songs available for Samira as well as their status.

Each array element contains fields:

Position | Description | Type | Additional information
---------------------------|--------------|-----------|-----------
0 | Music ID | Int | Refer to [Musics](../Enums%20and%20IDs/Music%20IDs.md) document
1 | Unlock status | Int | -1 - Not bought, 1 - Bought

### Line 10 (Array line) Stats bonuses
This line corresponds to all the stats bonus applied to the party.

Each array element contains fields:

Position | Description | Type | Additional information
------------|-----------------|----------------------------------------|--------------------
0 | Bonus type | Int | 0 - HP, 1 - Attack, 2 - Defense, 3 - TP, 4 - MP
1 | Bonus amount | Int
2 | Target | Int | -1 - Party, 0 - Vi, 1 - Kabbu, 2 - Leif

### Line 11 (Array line) Library and map locations
This line correspond to each element of the library menu as well as the mmap locations.

Element | Description | Additional information
---------------|-------------|----------------------------------------
0 | Discoveries | Refer to the [Discoveries](../Enums%20and%20IDs/Discoveries%20IDs.md) document
1 | Bestiary | Refer to the [Bestiary](../Enums%20and%20IDs/Bestiary%20IDs.md) document
2 | Recipes | Refer to the [Recipes](../Enums%20and%20IDs/Recipes%20IDs.md) document
3 | Records (acheivements) | Refer to the [Records](../Enums%20and%20IDs/Records%20IDs.md) document
4 | Seen map locations | Refer to the [Areas](../Enums%20and%20IDs/Areas%20IDs.md) document

Each array element contains a list:

Position | Description | Type | Additional information
------------------|------------|-----------------|------------------------------------
0-255 | Unlocked | Bool | Refer to the appropriate document for the IDs

Each unused ID has the value `false`.

### Line 12 (List line) Flags
This line correspond to the flags array.

Position | Description | Type | Additional information
------------------|--------------|-------------------|--------------------
0-749 | Value | Bool | Refer to the [global flags](../Flags/Global%20flags.md) document
> Saves before 1.1.x have 700 items making the position 0-699.

### Line 13 (List line) Flagstring
This line correspond to flagstring array.
> The separator used here is "|SPLIT|", NOT ",".

Position | Description | Type | Additional information
---------------------------|-------------|---------------|----------------
0-14 | Value | String | Refer to the [flagstrings](../Flags/Flagstrings.md) document

### Line 14 (List line) Flagvar
This line correspond to flagvar array.

Position | Description | Type | Additional information
------------------|------------|----------------|-------------------------
0-69 | Value | Int | Refer to the [flagvars](../Flags/Flagvars.md) document
> Saves before 1.1.x have 65 items making the position 0-64.

### Line 15 (List Line) Regional flags
This line correspond to the regionalflags array.

Position | Description | Type | Additional information
-----------------|------------|--------------|-----------------
0-99 | Value | Bool | Refer to the [regionalflags](../Flags/Regional%20flags.md) document

Each unused ID has the value `false`.

### Line 16 (Field line) Crystal berries
This line to whether or not each crystal berry has been collected.

Position | Description | Type | Additional information
----------------|-------------|---------------|----------------
0-49 | Collected | Bool | Refer to the [crystal berries](../Enums%20and%20IDs/Crystal%20Berries%20IDs.md) document

### Line 17 (List line) Followers
This line corresponds to the extra followers of the party.
> Each follower ID won't do anything unless the game allows them to follow the party at the current map.

Position | Description | Type | Additional information
-----------------|-----------|----------------|----------------
ALL | Follower ID | Int | Refer to the [AnimIDs](../Enums%20and%20IDs/Anim%20IDs.md) document

### Line 18 (Array line) Enemy encounters
This line corresponds to enemy encounters information for each enemies.

The index of the array corresponds to each enemy ID, refer to the [Enemies](../Enums%20and%20IDs/Enemies%20IDs.md) document for details.

Each array element contains fields:

Position | Description | Type
-----------------|----------|-----------
0 | Number seen | Int
1 | Number defeated | Int

The array size is always 256, if the ID is unused, each fields contain 0.
