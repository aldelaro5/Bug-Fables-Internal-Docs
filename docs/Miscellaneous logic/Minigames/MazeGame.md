# MazeGame
This is a minigame at the Termacade. It is a 3D grid based dungeon crawler with a third person view and randomly generated floors. Since its logic is very complex, only the important details are mentioned here.

## Rules and goal
The game is segmented into 3 randomly generated floors layed out in a grid. The player always starts on the bottom left tile while the stairs is located on the opposite corner where it requires a key to gain access to it. The rooms in the floors are randomly placed and have random sizes with randomly positioned corridors that joins the rooms together. At the start of each floors, enemies and potions are placed at random places. The player has 6 HP and can be attacked by enemies or by magic balls that each deals 1 HP per attack. Collection a potion heals 1 HP.

The goal of the game is to reach the stairs of the 3rd floor within 5 minutes without having 0 HP at any point. The game is over if the time limit expires or if the player has 0 HP at any point. The score accumulated still counts if the game is over or the player quits.

## Controls
The player can move forward 1 tile with the up input which doubles as the attack action to an enemy. They can turn with the left and right input which also turns the view of the camera. They can also hold the confirm input to hold a shield which both prevents enemies or magic balls to deal damages and it also allows to move in any cardinal directions (up, down, left or right) without needing to turn allowing faster movement, but they cannot attack while shielding.

If no inputs is done within 120.0 frames, a compass arrow appears at the top pointing towards the key or the stairs if the key was collected already.

Finally, the pause input can be used to pause the game where if the confirm input is pressed, the game resumes, but if the cancel input is pressed, the game ends (the score obtained still counts if this is done).

## Enemies
There are 2 types of enemies:

- Ant: They wander randomly by moving 1 tile per movement periodically until the player gets close enough where they will chase them and attack them if they are next to them
- Wizard: They stay stationary, but can be moved back due to the player attacking. They don't directly attack, but they will throw a magic ball towards the player in a line if the player gets close enough which will also play a sound

All enemies need 2 attacks from the player before they are defeated.

## Floor generation
All 3 floors have at most 8 rooms surrounded by walls where the first 2 are always the starting one and the stairs located in opposite corners. The other 6 rooms are randomly generated, but if one takes more than 20 tries to generate such that it doesn't overlap with any other rooms, that room will not be generated.

A room is defined by a rectangle formed by its 2 opposite corners tiles's coordonate in grid space. The position and size are random, but they cannot overlap. Corridors are then generated to joins the rooms at random positions, but all tiles around the perimeters of the map except for the stairs room are guaranteed to be free.

Each floors has a different map size:

- 1: 25x25
- 2: 30x30
- 3: 35x35

Each room contains a potion and each floors also has an amount of enemies that is equal to the amount of rooms generated + a number that starts at 0 and increases by 1 on each floors. Additionally, there's is always 1 wizard enemy in the stairs room (2 if it's the 3rd floor). This means that if all of the 8 rooms are generated for each floors, the amount of enemies is predetermined to be the following according to the floor number:

- 1: 9 enemies
- 2: 10 enemies
- 3: 12 enemies

The type of the enemies other than the ones in the stairs room are always ant in the first foor. For the second and third floor, each has a 3/5 chance to be ant and 2/5 chance to be wizard.

## Scoring
As for the scoring, it's done in 2 parts. There's the subtotal part and there's the end game bonus part which is only granted if the stairs of the 3rd floor was reached.

Here's what adds to the subtotal:

- Collecting the key: +500
- Attacking an enemy: +10
- Defeating an enemy: +100
- Unlocking the door leading to the stairs: +400

Because there are a predetermined amount of enemies on each floors if all of the rooms were generated correctly, this means that the maximum subtotal possible under normal gameplay is:

3 * 500 + 3 * 400 + (9 + 10 + 12) * (10 + 10 + 100) = 6420

As for the endgame bonus, it applies by applying multipliers. There are 2 multipliers calculated separately:

- HP bonus: This is equal to 1.0 + (HP remaining as float / 3.0)
- Time bonus: This is equal to 1.0 + (seconds remaining / 150.0)

So the final score is equal to the following ceiled:

subtotal * HP bonus * Time bonus
