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
- `onlytype`: `onlyone`, if this is 1 or higher, controller glyphs are always used even if MainManager.`joystick` is true. If this is -1, controller glyphs are only used if MainManager.`joystick` is true. Any other number will cause keyboard glyphs to always be used even if MainManager.`joystick` is true
- `descriptions`: `labeltext`, The test to render near the glyph via SetText on Start in non dialogue mode childed to the GameObject. If this is null or empty, no text will be rendered on Start
- `position`: `tposition`, The local position of the GameObject to set on Start. If this is null, it won't be changed
- `iconsize`: `size`, The scale of the GameObject to use in the component. If this is null, it defaults to Vector3.one
- `sortorder`: `overridesortorder`, The base sortingOrder of the glyph. Arrows and text inside the glyph are rendered with this value + 1 as the sortingOrder
- `parentobj`: `parent`, If this isn't null, the GameObject gets childed to this and their local angles are reset to Vector3.zero
- `startcolor`: `basecolor`, The color of the SpriteRenderer of the glyph set on Start, defaults to Color.white
- `label_offset`: `labeloffset`, If `labeltext` isn't null, an offset to apply to the local position of the text rendered near the glyph

Additionally, there's some exposed public fields:

- `basesprite`: The SpriteRenderer of the glyph
- `centerx`: This field does nothing
- `textsize`: This field does nothing
- `layer`: The layer to render the glyph, defaults to 5 (`UI`)
- `buttonname`: The text displayed inside the glyph that indicates the keybaord input
- `tridimentional`: If true, the layer of the GameObject won't be set to `layer` so it will stay at 0 (`Default`) and it also makes any SetText calls use this value as the tridimensional parameter. NOTE: This is incorrectly not honored for arrow glyphs which sets the layer anyway
- `shrunkkey`: If true, on every LateUpdate, the sclae of the GameObject gets set to MainManager.MultiplyVector((0.55, 1.0, 1.0), `size`)

The component only has a Start which setups the initial render and a LateUpdate that updates the sprite to render and the text to render with a non dialogue SetText call if needed.

There is also a public method called ChangeButton:

```cs
public void ChangeButton(int newid)
```
It allows to change the input id associated with the glyph's `id` and run Start again which will rerender everything.
