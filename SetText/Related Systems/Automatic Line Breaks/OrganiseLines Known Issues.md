# [OrganiseLines](OrganiseLines.md) known issues

The line breaking logic has some known issues that are important enough to be mentioned in its own page. These issues impact any public versions of the game from 1.0.0.

## Not counting a whole word's width after the first line

While a partial fix is applied to this one, it was intentionally disabled for the `English` language and is also explains why `Japanese` has no auto line breakers. Since this issue is still present in some ways and is historically relevant, an explanation is warranted.

After processing the first line and inserting the first line break, there was an error present from early development when resetting the line accumulator width. It was reset to 0 and that failed to take into account the first word of the following line. This frequently lead to overflow because the line breaker thought there was more space than there actually was. The cause of the issue wasn't known for a very long time during development and what was done instead was to tune the default maxoffset to 9.75 instead of what it ended up being which is 10.5. This was a partial workaround to the problem, but the issue still persisted.

It wasn't a huge problem until efforts to localize the game happened. When it came time to implement the `Japanese` language, it was clear that the current state of the auto line breaker would not work. As the cause of the issue wasn't known yet, it was decided to manually insert [Line](../../Commands/Individual%20commands/Line.md) commands in the `Japanese` script for every single point where a line break was needed. This is why this language's dialogues files have a very high amount of [Line](../../Commands/Individual%20commands/Line.md) command, but it also explains why [OrganiseLines](OrganiseLines.md) does nothing on `Japanese`; every line break was done by hand instead.

After a while, the issue was uncovered and a fix was ready, but applying it posed problems for both `Japanese` and `English` which were already implemented. The former would potentially break the manual [Line](../../Commands/Individual%20commands/Line.md) commands while the later was completely written with the assumption that [OrganiseLines](OrganiseLines.md) stayed broken which made fixing the issue risky.

Ultimately, it was decided to apply the fix for every language except `English` which already was acceptable with the issue and leave `Japanese` without an auto line breaker. Hence why this issue is still present on `English`, but since the whole script was written with this broken logic in mind, it isn't a huge issue with the script the game ships with.

## Not counting a trailing space's width after the first line

This is similar to the issue above, but this one affects every languages other than `Japanese` which doesn't have the auto line breaker. When inserting a line break, the new line contains the current word with a space gap. The fix for the above issue accounts for the word, but it fails to account for the space gap. This can make the difference where a small word is barely overflowing the max offset due to this as a space isn't counted when accumulating the line's width.

## The default messagebreak is too low

For every other languages than `English` and `Japanese` (which doesn't have the auto line breaker), there can be situations where a word barely doesn't fit on the line despite the fact that it would visually look fine with the word included. While it doesn't sound like a huge issue, it can push relatively large words below leaving a wide gap at the end of the line which can make the text less dense. The default value of 10.5 is slightly too low compared to what can visually fit in one line.
