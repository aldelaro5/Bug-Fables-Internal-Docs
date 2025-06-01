# SetTalk
Since SetText in [dialogue mode](../Dialogue%20mode.md) needs to manage dialogues between entities, there's a method to update their talking state called SetTalk:

```cs
private static void SetTalk(bool dialogue, bool talkstate)
```
Changes the `tailtarget`'s EntityControl's `talking` to `talkstate`. If `talkstate` is true, their `backsprite` is also set to false.

This method does nothing if `dialogue` is false or if `tailtarget` is null.

This method is meant to be used in a SetText context where SetText can periodically control visually whether the `tailtarget` should be in their `t` animation or not.
