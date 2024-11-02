# PlayerControl Start

- MainManager.`player` is set to this which assigns the PlayerControl singleton for the rest of the game to use
- `entity` is set to the EntityControl of this GameObject
- tag is set to `Player` with a layer of 11 (`Player`)
- `flycooldown` set to 240.0
- entity gets [CreateDetector](../Entities/EntityControl/EntityControl%20Methods.md#createdetector) called which adds a wall detector using a BoxCollider with size (0.35, 0.6, 0.1) and center (0.0, 0.75, `ccol` radius + 0.1)
- entity.`speed` set to `basespeed` which should always be 5
- entity.`rigid` mass is set to 0.01
- `npc` is reset to a new empty list
- entity.`overridejump` is set to false
- `bubbleshield` is initialised to be an instance of `Prefabs/Objects/BubbleShield` childed to the entity.`rotater` with a local position of (0.5, 1.25, 0.0) and scale of Vector3.zero
- `digicon` is set to 2 elements to form the UI when `digging` (either are initially disabled using the `spritedefaultunity` material on layer 15 which is `3DUI`):
    - 0: A new sprite object childed to this GameObject locally positioned at 0.5 in y using the `guisprites[3]` sprite (down arrow) with a scale of (0.35, 2.0, 1.0)
    - 1: A new sprite object childed to `digicon[0]` locally positioned at 1.5 in y using the `guisprites[6]` sprite (`Beetle`'s gui icon) with a scale of (0.8, 0.8, 0.8)
