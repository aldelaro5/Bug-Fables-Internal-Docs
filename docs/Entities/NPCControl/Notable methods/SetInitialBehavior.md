# SetInitialBehavior

This method first applies a special case where for any `BeeGuard` [AnimId](../../Enums%20and%20IDs/AnimIDs.md), it overrides the `tattleid` to -97 (common [dialogue line id](../../SetText/Common%20commands%20id%20schemes/Dialogue%20line%20id.md) 96) if it had -1 already.

Then, some specific logic are performoned depending on the default (first) `behaviors` (nothing happens if it's none of them):
<!-- - `FacePlayer`, `FaceAhead`, `FaceDown`, `FaceBehind` and `FaceUp`: calls the method with the same name on the entity and also invoke it in 0.5 seconds and another time in 1 second. This ensures the entity faces the correct direction on startup. -->
<!-- - `TurnRandomly` and `TurnFixedInterval`: sets the `actioncooldown` to the first `actionfrequency`. -->
<!-- - `WanderUnderground`: calls InstantDig on the entity which immediately causes it to dig underground. -->