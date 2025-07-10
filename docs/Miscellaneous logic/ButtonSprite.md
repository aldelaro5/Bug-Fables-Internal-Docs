# ButtonSprite
This component controls a input glyph rendering. The component renders a glyph that matches the physical input acording to current input bindings and input scheme. It is constantly monitoring changes in either to render the one matching the current input state. It can even be rendered using a button SetText command, but it's available to be used outside of it as well. A glyph is composed of a sprite and optionally, some text inside rendered by SetText in non dialogue mode.

It it meant to be programmatically attached to a component with a SpriteRenderer and have the SetUp method called upon adding it:

```cs
public ButtonSprite SetUp(int buttonid, int onlytype, string description)
public ButtonSprite SetUp(int buttonid, int onlytype, string description, Vector3 position, Vector3 iconsize, int sortorder, Transform parentobj)
public ButtonSprite SetUp(int buttonid, int onlytype, string description, Vector3 position, Vector3 iconsize, int sortorder, Transform parentobj, Vector3 label_offset)
public ButtonSprite SetUp(int buttonid, int onlytype, string description, Vector3 position, Vector3 iconsize, int sortorder, Transform parentobj, Color startcolor)
public ButtonSprite SetUp(int buttonid, int onlytype, string description, Vector3 position, Vector3 iconsize, int sortorder, Transform parentobj, Color startcolor, Vector3 label_offset)
```
Here are the defauilt values for the unsent parameters except for the first overload:

- `label_offset`: Vector3.zero
- `startcolor`: Color.white (note: the third overload does NOT use the parameter value and sends Color.white anyway)

The first overload doesn't use any unsent parameters so their associated fields are left to their default values.

Here's the parameters to fields mapping (all of these are private fields):

- `buttonid`: `id`, the input id of the glyph to render
- `onlytype`: `onlyone`, 
- `descriptions`: `labeltext`, 
- `position`: `tposition`, 
- `iconsize`: `size`, 
- `sortorder`: `overridesortorder`, 
- `parentobj`: `parent`, 
- `startcolor`: `basecolor`, 
- `label_offset`: `labeloffset`, 

Additionally, there's some exposed public fields:

- `basesprite`: 
- `centerx`: This field does nothing
- `textsize`: This field does nothing
- `layer`: 
- `buttonname`: 
- `tridimentional`: 
- `shrunkkey`: 

The component only has a Start which setups the initial render and a LateUpdate that updates the sprite to render and the text to render with a non dialogue SetText call if needed.
