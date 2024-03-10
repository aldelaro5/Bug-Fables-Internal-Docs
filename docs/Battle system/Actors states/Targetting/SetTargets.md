# SetTargets
This method updates `availabletargets` and `helpboxid` according to the `playerdata[currentturn].battleentity`.[animid](../../../Enums%20and%20IDs/AnimIDs.md)`.

- If `currentturn` is negative (no player is currently selected), `helpboxid` is set to -1
- Otherwise, `helpboxid` is set to `playerdata[currentturn].battleentity.animid` and [GetAvaliableTargets](GetAvaliableTargets.md) is called according to that [animid](../../../Enums%20and%20IDs/AnimIDs.md):
    - 0 (`Bee`): without onlyground and onlyfront with attackid of -1
    - 1 (`Beetle`): with onlyground and onlyfront with attackid of -2
    - 2 (`Moth`): with onlyground and without onlyfront with attackid of -3
