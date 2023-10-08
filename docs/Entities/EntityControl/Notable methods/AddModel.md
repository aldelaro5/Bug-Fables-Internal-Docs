# AddModel

AddModel is a method in EntityControl that is called whenever a need to have a custom object structure occurs for [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) that needs it:

````cs
public void AddModel(string path, Vector3 offset)
````

The `path` is the path of a prefab in the asset tree and the `offset` is the desired local position. The actual object gets assigned to `model` with the parent being the `spritetransform`. The `modelscale` is set to the model's actual scale.

The way the animator is setup is different than normal. Normally, the animator is present on the [Entity](../../Entity.md)'s root object, but when a model is added, the prefab's root can have an animator already defined. In which case, this is the one that will be assigned to `anim` instead of the root's one. This allows the controller attached to target the [AnimIDs](../../../Enums%20and%20IDs/AnimIDs.md) specifically even if the structure isn't standard.

Finally, this method also makes sure to setup all the child when `hologram` is true. Specifically:

* All child's SpriteRenderer have their material assigned to the holosprite one and if `cotunknown`, with a color of opaque pure black on the sprite and material
* All child's SkinnedMeshRender and MeshRenderer have their material assigned to the holosprite one and if `cotunknown`, with a color of opaque pure black on material or the same color with 50% transparent otherwise
