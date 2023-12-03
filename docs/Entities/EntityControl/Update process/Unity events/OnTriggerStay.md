# OnTriggerStay
This event only does anything every 2 frames.

First, `flowerbed` is set if it's not an [item entity](../../Item%20entity.md) and the other collider has the `FlowerBed` tag. Its position will be (0.0, 0.65, 0.0).

Finally, if the other collider has the `IceRadius` tag, `inice` is set to true.
