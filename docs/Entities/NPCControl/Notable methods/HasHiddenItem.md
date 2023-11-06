# HasHiddenItem
This method tells if the NPCControl pertains to an item that is discoverable by the Detector [Medal](../../Enums%20and%20IDs/Medal.md). It only returns true if all of the following are true (it returns false otherwise):
- The `entitytype` is `Object`
- The player has the Detector equipped
- The `interacttype` isn't `CaravanBadge` or `Shop`
- The `NDTCT` [Modifier](../../EntityControl/Modifiers.md) isn't enabled
- Either the `DDIST` [Modifier](../EntityControl/Modifiers.md) isn't enabled or it is, but the player's z position is less than 20.0 away from this transform z position
- The `objecttype` must belong to a specific set on top of an additional condition for the method to return true. Consult each's documentation to learn more:
  - [BeetleGrass](../ObjectTypes/BeetleGrass.md)
  - [DigSpot](../ObjectTypes/DigSpot.md)
  - [Item](../ObjectTypes/Item.md)

