# MidPos
A component meant to be programmatically added to an existing EntityControl's `sprite` or via Unity to a `model` entity prefab.

This component implements the ability for multiple Transform to be linked in such a way that it simulates a chain where each nodes is in a list of Transform and they move together. The logic is very complex so for the sake of brevity, only the public fields will be documented. The component has a Start and a LateUpdate (where most of the logic happens).

Here are the public fields and their effects:

- `links`: The list of Transform to link together. If `getfromchild` is true, this will be set to all the child Transform on Start
- `start`: The position of the start of the chain. If `getstartandendfromlink` is true, this will be overriden to `links[0]` position. If `localpos` is true, this refers to the local position instead
- `end`: The position of the end of the chain.  If `getstartandendfromlink` is true, this will be overriden to the last `links` element's position. If `localpos` is true, this refers to the local position instead
- `middle`: If the magnitude of this is above 0.1, the chain will form a BeizierCurve3 from start to end with this value being the midpoint of the curve. Otherwise, the chain will move form a straight line using a lerp
- `localpos`: If true, `start` and `end` refers to the local position of the chain instead of the position
- `getstartandendfromlink` If true, `start` and `end` are overriden to `links[0]` position and the last `links` element position respectively (this is incompatible with `localpos` being true)
- `getfromchild`: If true, `links` will be set to all child Transform on Start
- `ismodel` If true and `parenttransform` isn't null, the related EntityControl (used to determined if their `sprite` has `flip`) is assumed to be in the grand grand parent of `parenttransform`. Otherwise, it's assumed to be int the `parenttransform` directly
- `parenttransform`: If this isn't null, the Transform where an EntityControl resides related to the chain (if `ismodel` is also true, it's assumed to be on its grand grand parent instead)
