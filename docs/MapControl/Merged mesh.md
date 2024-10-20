# Merged mesh
There is a feature MapControl has that is normally only enabled for the `BugariaResidential` [map](../Enums%20and%20IDs/Maps.md) and it involes merging all the meshes (including their MeshCollider) under every GameObject with the `Merge` tag. This process is done in CombineMesh which is invoked by Start in 0.1 seconds, but only if `mapid` is `BugariaResidential`. The feature should be functional for other maps, but it won't be used.

This feature is in place for performance reasons as having less individual meshes can help rendering performance. Here is what CombineMesh does:

- Gather all GameObjects with tag `Merge`, if none exists, the method does nothing
- Build an array of CombineInstance where each mesh is each `Merge` GameObject's MeshFilter and each transform is the GameObject's world transform. During the course of building the array, all `Merge` GameObject are destroyed except the first one, but it will have its MeshRenderer destroyed (its materials will be stored)
- Create a new GameObject named `merged mesh` with a MeshFilter containing a new Mesh then having CombineMeshes called on it with that new mesh using the CombineInstance array built earlier. The GameObject is childed to the first `Merge` GameObject with angles (-90.0, 180.0, 0.0) and uses layer 8 (`Ground`). It has a MeshRenderer using the materials stored from the first `Merge` GameObject. Fionally, it has a MeshCollider with a sharedMesh set to the MeshFilter's mesh
- The MeshRenderer just added is added to `render`
