# HardMode
This is a helper method that returns a boolean telling whether or not hard difficulty scaling should apply.

It's used primarily in the damage pipeline and in the actions logic to determine the scaling to use.

It only returns true if one of the following is true (false is returned otherwise):

- The `DoublePain` [medal](../../Enums%20and%20IDs/Medal.md) is equipped
- [flags](../../Flags%20arrays/flags.md) 614 is true (HARDEST is enabled)
- [flags](../../Flags%20arrays/flags.md) 166 is true (EX Mode is active when using the B.O.S.S system)
