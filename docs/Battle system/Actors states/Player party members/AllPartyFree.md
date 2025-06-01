# AllPartyFree
This is a method on MainManager:

```cs
public static bool AllPartyFree()
```
It returns true if no `playerdata` has a `cantmove` of 1 or higher (no actor turns available on the current main turn), false otherwise.

This is mainly used to know if a party rotation or swap is still possible because the battle system requires that all player party member can act for this to be possible.
