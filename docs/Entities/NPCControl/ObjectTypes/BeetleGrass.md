# BeetleGrass
A grass that can be cut by Kabbu's Horn Slash. Can either be bound to a [crystalbfflags](../../../Enums%20and%20IDs/crystalbfflags.md) slot which will be dropped, or a list of [items](../../../Enums%20and%20IDs/Items.md) that can be dropped alongside the activation of a [flag](../../../Flags%20arrays/flags.md) and [regionalflag](../../../Flags%20arrays/Regionalflags.md) slots. 

## Data Arrays
- `data[0]`: The id of the grass sprite to use (NOTE: any other value than the ones below when `data[1]` doesn't apply will cause an exception to be thrown):
  - 0: a standard green grass
  - 1: a branch looking grass
  - 2: an orange grass
  - 3: a brown grass
  - 4: a dense dark green grass
  - 5: a purple grass
  - 6: a beige and gray grass
- `data[1]`: If it's not negative, the [crystalbfflags](../../../Enums%20and%20IDs/crystalbfflags.md) id that will be dropped when the bush is cut. This is optional: no Crystal Berry is dropped if it doesn't exist
- `vectordata`: The list of [items](../../../Enums%20and%20IDs/Items.md) ids (in the x component after flooring) to potentially drop. Only applicable if `data[1]` doesn't apply

## Additional data
- `boxcol`: Required to be present with valid data
- `regaionalflag`: The [regionalflag](../../../Flags%20arrays/Regionalflags.md) slot turned to true when the bush is cut while `data[1]` isn't present or is negative (if an item is dropped, it's attached to the item created).
- `activationflag`: The [flag](../../../Flags%20arrays/flags.md) slot turned to true when the bush is cut while `data[1]` isn't present or is negative (if an item is dropped, it's attached to the item created).

## HasHiddenItem conditions
If `data[1]` is present and not negative, the [crystalbflags](../../Enums%20and%20IDs/crystalbfflags.md) whose id is that value must not have been obtained yet.

## Setup
A few adjustements occurs:
- The entity.`alwaysactive` is set to true
- The gameObject's isStatic is set to true
- The entity.`rigid` is placed in kinematic mode without gravity
- `nointeract` is set to true
- The layer is set to 0 (default)
- The `scol` is disabled

If `data[1]` is present and not negative and the crystal berry corresponding to its id has been obtained, the entity.`iskill` is set to true which ends this object setup as the grass will not appear.

Otherwise:
- The layer is set to 8 (Ground) 
- The `boxcol`'s material is set to the defaultpmat. 
- The entity.`rotater` tag is set to `Hornable` which allows PlayerControl to get a green ! emoticon when getting 2.5 or lower distance from the grass for 5 frames
- The entity.`rigid` has all constraints frozen
- entity.`overrideanim` is set to true
- entity.`sprite` is enabled and set to the corresponding sprite from grasssprite mentioned above and its shadowCastingMode is set to TwoSided
- Unless nowindeffect is true (meaning Wind Effects in the settings is Off), the entity.`sprite` material is set to the windShader and RefreshWind is called on the entity.`sprite` which sets the shader property to be:
  - `_ShakeDisplacement`: random betweent half the map.`windspeed` and it's actual value
  - `_ShakeBending`: random betweent half the map.`windintensity` and it's actual value
  - `_ShakeTime`: random betweent 0.075 and 0.25

## Update
If the `timer` hasn't expired yet, it is decremented by the game's frametime clamped from 0.0 to infinity. Otherwise, if it is 0.0 and the entity isn't `dead`, a [Death](../../EntityControl/Notable%20methods/Death.md) coroutine is started with the entity.

## OnTriggerEnter
If the other gameObject tag is `BeetleHorn` or `BeetleDash` while `hit` is false and the distance between this and the other transform is less than 3.5, CutGrass is called (See the section below for details) followed by a RefreshPlayer call which forces the player.`ceiling` the player.entity.`hitwall` (when not `inevent` and not in `battle`) and all `playerdata` entity.`onground` to false as well as childing the player to the root. NOTE: it is unknwon why RefreshPlayer is called.

### CutGrass
A number of steps occurs:
- The `rustling1` is played on entity
- The entity.`rotater` tag is set to `Object` (this removes its Hornable property).
- `hit` gets set to true (which prevents OnTriggerEnter to act again)
- The `boxcol` gets disabled
- The actual sprite of entity.`sprite` gets changed to the small base version of the grass according to `data[0]` and its material gets set to `spritemat`

From there, this is where the potential item drop and flag slots edits occurs.

#### Crystal Berry drop
If `data[1]` is present and not negative, [CreateItem](../../EntityControl/Notable%20methods/CreateItem.md) is called with the following:
- starpos of this transform + (0.0, 0.5, 0.0)
- itemtype of 3 (Crystal Berry)
- itemid of `data[1]`
- direction of RandomItemBounce(4.0, 12.0)
- No timer

#### Potential Item Drop
If `data[1]` isn't present or is negative, `vectordata` is checked to see if we are going to drop an item. The way this is done is selecting a uniform random valid index of a `vectordata` element. It is valid if it's not negative. If it is valid, CreateItem is called which creates an [Item](Item.md) object with the following:
- starpos of this transform + (0.0, 0.5, 0.0)
- itemtype of 0 (Standard item)
- itemid of `vectordata[i].x` floored where `i` is the random index generated earlier
- direction of RandomItemBounce(4.0, 12.0)
- timer of 600 frames

Once the creation is done, the `regionalflag` and `activationflag` of the new item are set to this object's corresponding values. This binds them to the item instead of the BeetleGrass object itself.

In the case where `vectordata` is empty or the generated index leads to a negative value of the element x component, the logic is limited to set the flag and regionalflag slots of this object's `regionalflag` and `activationflag` to true.

No matter which cases we land into, there is always a 13% chance to call CreateItem a second time with the following:
- starpos of this transform + (0.0, 0.5, 0.0)
- itemtype of 0 (Standard item)
- itemid of 6 (`MoneySmall`)
- direction of RandomItemBounce(4.0, 12.0)
- timer of 600 frames

#### Common ending logic
For any cases at the end of the method, if we dropped any item or Crystal Berry, all collisions between the item and this object's entity are ignored.

Finally, a GrassFade coroutine is started. The purpose of that coroutine is only for rendering the cut part of the grass which is done by creating a new gameObject named `grass` with a SpriteRenderer, SphereCollider, BoxCollider and RigidBody where the sprite is the cut grass version according to `data[0]`. For brevety, the full details of this effect won't be detailed, but a couple of things is worth to mention:
- The grass layer is set to 9 (Follower)
- A torque is applied on the grass of RandomItemBounce(5.0, 0.0)
- The starting velocity is (0.0, 10.0, 0.0)
- If an item or Crystal Berry was dropped earlier, all collisions between the item and the grass's colliders are ignored
- For each frames until 3 seconds passed, the `rustling2` sound is played if the y velocity is negative while it was positive before
- The fade is done by setting the material to `spritematlit`, its renderQueue to 3000 and to progressively fade the alpha of the grass until it reaches 0.1 or below by subtracting 1/20 of the game's frametime each frame
- The grass object is destroyed once the fading is completed

## PauseMenu.NearSomething
If the method finds out that an NPCControl of this object type exists less than 4.0 distance from the player, the return will be an array of 3 booleans with the first one being true. This will prevent usage of the Bed Bug [item](../../../Enums%20and%20IDs/Items.md).

## HasHiddenItem
This object type is allowed to return true here if `data[1]` is positive and the corresponding [crystalbfflag](../../../Enums%20and%20IDs/crystalbfflags.md) slot has not been obtained yet.

## Hazards.OnObjectStay
If the NPCControl passed has this type, it returns true which allows it to stay in collision with the Hazards.