# WackaWorm
This component implements a minigame where worms will appear around a radius at random times and the goal is to hit the most of them using the [Beemerang](../Entities/NPCControl/ObjectTypes/Beemerang.md) within the time limit. The component represents both the minigame's manager and also each of the worms present in the minigame. At the end, the score will be in [flagvar](../Flags%20arrays/flagvar.md) 1 which is used temporarily for this.

## Startup
The minigame can be started at any time by calling the static New method on this component:

```cs
public static void New(int totaltime, int wormammount, int frequency, int endevent, float maxradius, Vector3 pos)
```
The method creates a rooted GameObject named `wackacontrol` with this component that has StartUp called on it. StartUp is public, but it's only used by New and it has the same signature other than its name and the fact it's not static. This component instance will serve as the controller.

StartUp is what actually configures the fields. The parameters are assigned to the following fields:

- `totaltime`: The time limit in seconds
- `wormammount`: The amount of worms to create which represents the maximum amount of worms that can be present at once
- `frequency`: `wfreq`, controls the range of time in frames that each worm needs to wait after revealing themselves before doing so again
- `endevent`: `eventtocall`, the [event](../Enums%20and%20IDs/Events.md) id to start when the time limit expires and the game ends
- `maxradius`: `radius`, determines the maximum distance from `pos` each worms may spawn
- `pos`: Set to the initial position of the controller which also serves as the center of the worms's spawns range

The controller has its `main` field set to true which allows Update and OnTriggerEnter to distinguish between the worms and the controller.

On StartUp, `wormammount` amount of worms are created which `Prefabs/Objects/Worm` prefab instances with a WackaWorm component childed to the controller. They will reveal themselves in a random amount of frames between 10 and `wfreq` / 2 (all are integer only operations). Some fields on the worm instances changes meaning such as `eventtocall` becomes the worm id for the controller instance.

## Worm reveal and hiding
The worms will reveal themselves (managed by the controller's Update) once the amount of frames elapsed and they will remain revealed for a random amount of frames between -`wfreq` and `wfreq` where they will hide themselves. However, if the player gets within 4.0 units they will instantly hide themselves for the same random amount of frames. The hidden check is only done every 2 Update cycles that MainManager.`player`.`beemerang` wasn't present which means that if it is, it will prevent any worms from hiding until it disappears.

Being revealed is what enables each worms's trigger BoxCollider. This is the only thing that is checked in OnTriggerEnter and it only detects collisions with MainManager.`player`.`beemerang`. When hit, the worm will hide itself, but their timer is still going and it will determine the next time they will reveal themselves. Getting hit increments [flagvar](../Flags%20arrays/flagvar.md) 1 which is used temporarily to keep track of the score.

## Ending
While all this is happening, the controller is tracking the timer and once it expires, the game ends and the `eventtocall` [event](../Enums%20and%20IDs/Events.md) starts.

The destruction isn't the responsibility of the component, but rather the ending event. To properly decomission the minigame, the DestroyThis publid method needs to be called by the event on the controller instance which will also destroys all the worm instances and other used GameObjects.
