# DeadLanderOmega
There are 3 components that are closely related together: DeadLanderOmega, OmegaHand and DeadLanderZones. They together implement the stealth mechanic of the [maps](../../Enums%20and%20IDs/Maps.md) in Giant's Lair.

## Main component
DeadLanderOmega is the main component behind most of the logic here.

TODO: document this later

## OmegaHand
This component only implements the hand part.

TODO: document this later

## DeadLanderZones
This is a very simple component that only has the logic of detecting sight of the MainManager.`player`.

It's meant to be be attached via Unity and it only has 3 fields which are public:

- `onlyrock`: Only detects `PushRock` tagged colliders (see the [PushRock](../../Entities/NPCControl/ObjectTypes/PushRock.md) documentation to learn more) causing a call to DeadLanderOmega.GetOmega(`id`).ForceLook(`setoffset`)
- `setoffset`: If `onlyrock` is true, the parameter sent to ForceLook of the DeadLanderOmega
- `id`: If `onlyrock` is true, the index of the return of FindObjectsOfType<DeadLanderOmega\> that this zone refers to

The only method is the OnTriggerEnter Unity event. If `onlyrock` is true, this is explained above. If it's false, it only acts if the other collider is the MainManager.`player` in which case, DeadLanderOmega.`activeid` is set to `id` which activates them.
