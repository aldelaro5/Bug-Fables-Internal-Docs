# GetDefense
This method serves as the way to get the defense of an enemy.

TODO: This very likely is part of the infamous def bug, recheck everything later when damage calc is documented. This only returns the base def taking into account hard mode, but nothing else

The method takes the `BattleData`'s `def` and if the `DouplePain` [medal](../../Enums%20and%20IDs/Medal.md) is equipped or [flag](../../Flags%20arrays/flags.md) 614 is true (HARDEST is active), `harddef` is added to it and then returned.