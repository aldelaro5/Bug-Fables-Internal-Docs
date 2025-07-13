# EntityTie
A component meant to be attached on any GameObject via Unity.

It will cause on the first LateUpdate after `delay` amount of frames to have MainManager.`map`.`entities[entityid]` have the following happen to them (this component disables itself after):

- Gets childed to the GameObject
- `alwaysactive` set to true
- Locally positioned at `offset`
- Local angles set to `angle` (if the magnitude of the value is above 0.1)

The `delay`, `entityid`, `offset` and `angle` fields are configurable via the inspector.
