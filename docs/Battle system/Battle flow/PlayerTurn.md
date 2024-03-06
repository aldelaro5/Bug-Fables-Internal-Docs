# PlayerTurn
This method processes any [controlled flow](Update.md#controlled-flow) during the [player phase](Main%20turn%20life%20cycle.md#player-phase) when applicable to the currently selected player.

- `actedthisturn` is set to true
- If `currentaction` is `BaseAction` (the main vine action menu):
    - [SetMaxOptions](../Player%20UI/SetMaxOptions.md) is called
    - [ItemList](../../ItemList/ItemList.md)'s `inlist` is set to false
    - For each `playerdata`:
        - If the actor isn't [IsStopped](../Actors%20states/IsStopped.md#isstopped), its battleentity.`overrideanim` is set to false
        - If battleentity.[animstate](../../Entities/EntityControl/Animations/animstate.md) is 24 (`Block`), [UpdateAnim](../Visual%20rendering/UpdateAnim.md) is called and the `playerdata` loop exited. NOTE: it means not all player party member might have their `overrideanim` set to false when not stopped, but in practice, this doesn't impact anything negatively under normal gameplay
- If `choicevine` doesn't exist, it is created via CreateVine:
    - `choicevine` is created as a new GameObject named `Vine`
    - `vineicons` is created as new array of length `maxoptions`
    - `vinebase` is created as a new GameObject named `Base` childed to `choicevine`
    - Each `vineicons` elements is initialised as the SpriteRenderer of a new instance of `Prefabs/Objects/Vine` childed to the `vinebase` (each element causes the `choicevine` angles to be decreased by (0.0, 360 / `maxoptions` floored, 0.0)):
        - position of `choicevine` position + (0.0, 0.0, -1.6)
        - angles of (-180, 275.0, 0.0)
        - scale of (2.5, 2.5, 2.5)
        - sprite of `guisprites[35 + i]` where i is the option index
        - with flipX
    - The `choicevine` gets childed to the `battlemap`
- The `choicevine` position is set to `playerdata[currentturn].battleentity` position + (0.0, 10.0, 0.0)
- If `turncooldown` expired, [GetChoiceInput](../Player%20UI/GetChoiceInput.md) is called which handles the player UI