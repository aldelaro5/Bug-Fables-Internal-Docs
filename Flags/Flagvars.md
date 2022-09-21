# Flagvar
A `flagvar` is an `int` which is a global variable the game can access for any purpose. They are saved on the save file and typically have one reserved use, but some can be used temporarilly notably in `SetText` operations. There are 70 flagvars available to the game in the `MainManager.instance.flagvar` array.
> Before 1.1.0, there were 65 slots instead

## Notes about the flagvars table
If a flagvar mentions "Prize medal", it means that the flagvar corresponds to the state of a medal given by Artis with Hard Mode on or bought in the medal shop without Hard Mode (not to be confused with the regular medal shops in the game). The possible values of a prize medal flagvar are as follows:

Value | Description
-------- | --------
0 | not available
1 | available from Artis 
2 | available from the caravan 
3 | obtained

A flagvar with the mention "TEMP" means its value in the save is only used for temporary usage by the game.

## Flagvars table
ID | Description
--------- | ----------
0 | TEMP
1 | TEMP
2 | TEMP
3 | Allows fast cooking if its value is 5353, 0 otherwise
4 | TEMP
5 | TEMP
6 | TEMP
7 | UNUSED
8 | UNUSED
9 | TEMP number of battles left at the current attempt of boss rush on the B.O.S.S system
10 | TEMP
11 | TEMP
12 | Number of big switches hit at Snakemouth Den (the ones that unlockes the large door)
13 | Prize medal: Quick Flea
14 | Amount of Crysal berries in possession
15 | Number of Lore Book on the library shelf
16 | Number of times Diana has been paid to build a tunnel
17 | Prize medal: Weak Stomach
18 | Prize medal: Spiky Bod
19 | Prize medal: Break
20 | Prize medal: TP Plus
21 | Prize medal: Life Stealer
22 | Number of charms left
23 | Marks the state of the I Wanna Get Better! quest, 0 - no progress, 1 - cooked the Yam Bread, 2 - cooked the Succulent Platter, 3 - cooked the Abomination, 4 - Won the battle against the Abomihoney
24 | Number of Factory Pass used on the door to the pump roomm at the Honey Factory
25 | Prize Empower
26 | Berry bank balance
27 | Token count
28 | Mite Knight high score
29 | Flower Journey high score
30 | Prize medal: Berry Finder
31 | Prize medal: Block Heal
32 | Whack-A-Worm high score 
33 | Prize medal: Hard Charge
34 | Prize medal: Enfeeble
35 | Number of crystal turned in to Doppel by completing a bounty
36 | Prize medal: Reverse Toxin
37 | TEMP? (used on DoClock and event 154)
38 | Number of completed Cave of Trials run
39 | Current streak count of completed Cave of Trials run
40 | Number of battles fought
41 | Highest damage in one turn
42 | Number of battles fled from
43 | Number of attempted Cave of Trials run
44 | Prize medal: Resist All
45 | Prize medal: Deep Taunt
46 | Prize medal: TP Core
47 | Number of quests completed (excluding the main story ones)
48 | Prize medal: Heavy Throw
49 | UNUSED
50 | Last generated price for a Longleg Summoner (0 if the last price generated was bought OR there was no price generated yet)
51 | Prize medal: Random Start
52 | Prize medal: Reflection
53 | Number of rewards obtained from Librem by turning in discoveries
54 | Last amount of discoveries turned in to Librem
55 | Number of prize medals available or obtained
56 | Item ID (refer to Appendix C) of the ribbon that Chompy has on (0 if no ribbon)
57 | Prize medal: HP Plus
58 | UNUSED
59 | UNUSED
60 | UNUSED
61 | Prize medal: Antlion Jaws
62 | Number of items bought from the leftmost Metal Island merchant
63 | Prize medal: Miracle Matter
64 | Prize medal: Triumph Buzz
65 | UNUSED
66 | Amount of medals bought at Shades's medals shop
67 | Seed used in MYSTERY?, 0 if not using MYSTERY?
68 | UNUSED
69 | UNUSED
