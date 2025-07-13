# FlappyBee
This is a minigame at the Termacade. It is a 2D side scrolling game made using 2 components: FlappyBee and FlappyBeePlayer.

## FlappyBee
The game has a FlappyBeePlayer that can flutter when pressing the confirm input where the game perpetually moves walls, enemies, combo pickup and honey items from right to left. The goal of the game is to survive as long as possible without hitting a wall or enemy without invulnerability which can be given when picking up a honey item. The collision detection part is done by FlappyBeePlayer which is explained in a section below. The game can go on infinetely until the game over condition is hit.

Score is given in various ways, but in general, being alive longer yields better score. Here's the exact ways to get points:

- Passing a wall: +15
- Passing an enemey: +30
- Grabbing a combo pickup: Increments `combo` and awards 10 * `combo` (see below for details)
- Touching an enemy while invulnerable: Increments `combo` and awards 10 * `combo` (see below for details)

When touching a honey item, invulnerabilty is given for the next 300.0 frames. Touching a wall or enemy while not invulnerable results in a game over.

The game has a `combo` system. The value starts at 0 and increments when picking up a combo item or touching an enemy while invulnerable. It goes back to 0 when when passing a combo item without touching it. Essentially, the fastest way to increase score is to keep picking up combo items in a row and killing enemies when possible.

> NOTE: There is a known issue with this system. When killing an enemy, their sprites gets disabled, but not necessarily the whole GameObject so their collider is still active. This is a convention the game follows elsewhere, but the problem is when checking if the player touched an enemy while invulnerable, it doesn't check that the enemy sprite is STILL enabled so it can result in colliding with an invisible, but active enemy over and over to increase `combo` multiple times during the invulnerable period.

The `combo` system also affects some spawning behaviors:

- At a `combo` above 3 after passing at least 8 walls, enemy starts to be able to be spawned. They can only spawn each 10 walls passed total (15 instead if the amount is above 100 or each 5 if invulnerable and there's more than 30.0 frames left of it) and the rate is 75% (100% instead if invulnerable and there's more than 30.0 frames left of it). An enemy can only spawn when one isn't present
- At a `combo` above 7, each 7 items (5 instead if an enemy is present), there is a 40% chance (60% instead if an enemy is present) that the item is a honey. NOTE: Because the "each X item" part is done using a modulo, it's possible that an enemy appearing interferes both of these checks such that no honey can spawn even if 7 items are passed so for example, if 6 items passes, but an enemy spawns, the 7th won't have the chance to be a honey because it isn't divisible by 5 and an enemy appeared

It is possible to pause the game with the pause [input](../../InputIO/Inputs.md) which allows to quit with the cancel input or to resume with the confirm input.

## FlappyBeePlayer
This is a simple component that holds a reference to the FlappyBee.

It only contains 2D collisions and trigger collisions detection for the player in the minigame. Trigger colliding with an item uses it on the FlappyBee side and colliding with anything else causes the death of the player which is handled by FlappyBee.
