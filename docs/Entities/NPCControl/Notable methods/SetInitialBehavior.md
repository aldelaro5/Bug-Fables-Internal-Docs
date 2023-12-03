# SetInitialBehavior
This method first applies a special case where for any `BeeGuard` [AnimId](../../../Enums%20and%20IDs/AnimIDs.md), it overrides the `tattleid` to -97 (common [dialogue line id](../../../SetText/Common%20commands%20id%20schemes/Dialogue%20line%20id.md) 96) if it had -1 already.

Some specific logic are performoned depending on the default [ActionBehavior](../ActionBehaviors.md), check their documentation to learn more.