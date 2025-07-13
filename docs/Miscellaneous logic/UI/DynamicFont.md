# DynamicFont
A component meant to be added programmatically or via Unity to a GameObject. If done programmatically, the StartUp method needs to be called:

```cs
public static DynamicFont SetUp(bool nokerning, float frequency, int fonttype, int sortorder, Vector2 fontsize, Transform parent, Vector3 position)
public static DynamicFont SetUp(string displaytext, bool centralized, bool nokerning, float frequency, int fonttype, int sortorder, Vector2 fontsize, Transform parent, Vector3 position, Color color)
public static DynamicFont SetUp(string displaytext, bool centralized, bool nokerning, float frequency, int fonttype, int sortorder, Vector2 fontsize, Transform parent, Vector3 position, Color color, Vector2? small)
```
The default `small` parameter is null for the first 2 overloads and for the first overload, here are what the other unsent parameters defaults to:

- `displaytext`: Empty string
- `centralized`: false
- `color`: Color.white

It will create a GameObject named `DynFont - X` where `X` is `displaytext` with a DynamicFont component childed to `parent` (or MainManager.`GUICamera`.transform if it's null) with a local scale of 1.0x. The component is configured with the following fields (see below for details):

- `startpos`: `position`
- `text`: `displaytext`
- `center`: `centralized`
- `monospace`: `nokerning`
- `updatefrequency`: `frequency`
- `fontindex`: `fonttype`
- `fontcolor`: `color`
- `size`: `fontsize`
- `sort`: `sortorder`
- `smallersize`: `small`

This component is important because it serve as a secondary way to render text that doesn't features any of the versatility then SetText, but it does have one huge advantage over it: it is made for text that constantly changes. SetText has to manage a lot of ressources (and it's very inefficient at it) and it has a letter slot limit on top of having a lot of overhead to just render the text or process the commands, but DynamicFont doesn't have such limits.

This is because all DynamicFont does on Update is simply to change the text field of all the TextMeshes. There's no materials being created, no effects, just the text changes. Since Unity is particularily efficient at handling this, it makes DynamicFont really performant for dynamically rendering text that reacts to the game. A prime example of in game usage is rendering HP numbers in the HUD: since they are expected to change a lot, it's much more efficient to use DynamicFont. The rendering only happens once on Start where one GameObject is created per TextMesh and they aren't rendered again. Only changes to the text are monitored on Update, but it also has a configurable throttle to reflect those changes.

Here are the public fields:

- `center`: This field doesn't do anything
- `monospace`: This field doesn't do anything
- `dropshadow`: If this is true, 2 TextMeshes will be rendered per letter where the second one will have a color of pure black, but half transparent, be locally positioned at `dropoffset` and have a MeshRenderer's sortingOrder being the same as the first TextMesh - 1
- `tridimentional`: If true, the TextMeshes will render on layer 0 (`Default`) instead of `layer`, but `triui` can still take priority if it is true
- `triui`: If true, the TextMeshes will render on layer 15 (`3DUI`) instead of `layer`. This takes precedance over `tridimentional`
- `text`: The text to render. Any changes to this field will be reflected on Update to the TextMeshes
- `updatefrequency`: The amount of frames to wait before Update actually updates the TextMeshes's text to `text`
- `fontindex`: The font id of the text to render (expressed as a MainManager.SetFont's fontid parameter)
- `sort`: The MeshRenderer's sortingLayer of the TextMeshes are set to this value + 1 (+ 0 is for the secondary TextMeshes if `dropshadow` is true)
- `layer`: The layer to render the TextMeshes when `tridimensional` and `triui` are both false
- `size`: The local x and y scale of the TextMeshes to render. Specifically, the local scale on render is set to (`size`.x, `size`.y, 1.0) * 0.07
- `dropoffset`: The local position of each secondary TextMesh when `dropshadow` is true
- `startpos`: The initial local position of the GameObject
- `fontcolor`: The color of the TextMeshes to render

Some additional notes about Start:

- `fontindex` on Start is set to MainManager.FontID(`fontindex`) so it represents the actually resolved font id after
- If the parent of the GameObject is still null by then, it is set to MainManager.`GUICamera`.transform
- The local angles are set to Vector3.zero

And some additional notes about the rendering:

- All TextMeshes's GameObjects are childed to the attached GameObject
- All TextMeshes's anchor are set to LowerLeft
- All TextMeshes's tag are set to `Text`
- All TextMeshes's local angles are set to Vector3.zero
