# SetMaxOptions
This method sets `maxoptions`, `availabletargets` and the `vineicons` meterials (greyscale if the choice is available, but disabled).

- `maxoption` is set to 3 (gives access to attack, skills and items options)
- If [flag](../../Flags%20arrays/flags.md) 15 is true (after combat tutorial), `maxoption` is incremented (gives access to the stategies option)
- If [flag](../../Flags%20arrays/flags.md) 16 is true (Leif is able to fight in battle), `maxoption` is incremented (gives access to the turn relay option)
- [SetTargets](../Actors%20states/SetTargets.md) is called
- If `choivevine` exists and `currentturn` isn't negative (a player is selected for acting), the materials of the `vineicons` are updated to MainManager.`grayscale` if the choice is disabled or MainManager.`spritedefaultunity` if the choice is enabled (only applicable if the element at the index exists):
    - `vineicons[0]` (attack) option is disabled if there's no `availabletargets`, enabled otherwise
    - `vineicons[1]` (skills) option is disabled if any of the following are true about the `playerdata` of `currentturn` (enabled otherwise):
        - It has no `skills`
        - Its `lockskills` is true
        - It has the `Inked` [condition](../Actors%20states/Conditions.md)
        - It has the `Taunted` [condition](../Actors%20states/Conditions.md)
    - `vineicons[2]` (items) option is disabled if instance.`items[0]` is empty (no standard items) or any of the following are true about the `playerdata` of `currentturn` (enabled otherwise):
        - Its `lockitems` is true
        - It has the `Sticky` [condition](../Actors%20states/Conditions.md)
        - It has the `Taunted` [condition](../Actors%20states/Conditions.md)
    - `vineicons[4]` (turn relay) option is disabled if there's only one `playerdata` or any of the following are true about the `playerdata` of `currentturn` (enabled otherwise):
        - Its `haspassed` is true
        - Its `locktri` is true
        - It has the `Taunted` [condition](../Actors%20states/Conditions.md)
- `vineoption` is set to `maxoption`
