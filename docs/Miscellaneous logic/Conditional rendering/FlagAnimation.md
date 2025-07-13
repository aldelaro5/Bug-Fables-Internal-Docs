# FlagAnimation
A component meant to be attached on any GameObject via Unity as long as the GameObject has an Animator component (it will be collected on the component's Start).

This components will cause the Animator to play an animation depending on the value of some [flags](../../Flags%20arrays/flags.md). It has 2 configurable fields via the inspector:

- `flags`: The reverse ordered list of flags slot to test for where the first that is true will be selected as the `anims` index to use. The first element MUST be negative for the component to work
- `anims`: The list of AnimationClip names to use whose indexes needs to match the ones in `flags`. The first element is the falback one where if no `flags` is true, it will be the clip selected instead

The check occurs every LateUpdate, but the animation is only played if the selected index from the 2 arrays above changes. The falback element counts as its own index so it counts in this changed check.

This is only used on a monitor screen at the `RubberPrisonSecurity` map to change the animation displayed on it depending on flag 550 (raised the cafeteria tables) and 553 (turned on the computer in the room).
