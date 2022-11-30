* if the [languageid](../../languageid.md) is `Japanese`
  * return the input string, nothing is done on it
* Set the input string to ReplaceFunctions(text):
  * Make a copy of the input string called the result string
  * If the input string is null, set the result string to empty string, otherwise:
    * For each char in the input string
      * If it's not a command, append the char to the result string
      * If it is a command, gather the command text (the next iteration resumes after the command text) and if the command is:
        * [Sstring](../../Commands/Individual%20commands/Sstring.md): Append the flagstring at the index of the only parameter of the command
        * `Menu`:
          * If it has 1 parameter, append menutext at the index of this parameter
          * Otherwise, get the menutext at the index of the first parameter and switch on the second parameter:
            * `1`: Append the first char of the menutext + `-` + the entire menutext to the result string
            * `2`: Append the menutext uppercased to the result string
            * `3`: Append the first char of the menutext + `-` + the entire menutext uppercased to the result string
        * Anything other commands: Append the command text to the input string 
  * Set the input string to the result string
* Char loop (for each char in the input string)
  * Pseudo switch on the char:
    * LF: CONTINUE
    * `|`:
      * For [Next](../../Commands/Individual%20commands/Next.md), [Blank](../../Commands/Individual%20commands/Blank.md) or any commands with `line` in it (except if paused with `Unpauseline` and when a Shopkeeper isn't in the interact range of the player for a `Shopline`):
        * Act as if a ` ` was processed, but without adding a space width to the line accumulator.
      * For [Button](../../Commands/Individual%20commands/Button.md):
        * Add 0.7 to the line accumulator. Add another 0.7 if the keyboard input text is not empty and it doesn't have `Arrow`. Reset the word accumulator to 0.
      * For any commands with `size` (except `Battlesize`) where the first param isn't `multi`:
        * Change the size to the x size parameter of the command
      * For [Singlebreak](../../Commands/Individual%20commands/Singlebreak.md):
        * Enable single break mode and set the maxoffset to the one sent in parameter of the command.
    * ` `: 
      * Add 0.3 * size to the line accumulator width (0.3 is the width of a space gap)
      * If the line accumulator width + the word accumulator width exceeds maxoffset
        * Add an LF or a [Line](../../Commands/Individual%20commands/Line.md) command in singlebreak to the result string
        * Set the line accumulator width to the start of the line:
          * If [languageid](../../languageid.md) is `English`, unless map.englishbreakfix is true AND the [message](../../Global%20vars%20used/message.md) lock is grabbed, this is 0 which is broken (see [OrganiseLines Known Issues > Not counting a whole word's width after the first line](OrganiseLines%20Known%20Issues.md#not-counting-a-whole-word-s-width-after-the-first-line))
          * Otherwise, this is the word accumulator which is only slightly wrong as it is missing a space gap (see [OrganiseLines Known Issues > Not counting a trailing space's width after the first line](OrganiseLines%20Known%20Issues.md#not-counting-a-trailing-space-s-width-after-the-first-line))
      * Otherwise
        * Add the word accumulator width to the line accumulator width
      * Append the accumulated word and a space to the result string
      * Reset the current word to empty and current word accumulator width to 0
    * Anything else:
      * Add the width of the char to the current word accumulator width and add the char to the current word. The width is floor(offset * 25.0) / 25.0. This width is multiplied by 0.975 if in single break mode
* if the last line accumulator width + the last word accumulator width exceeds maxoffset
  * Add an LF at the end of the result string
* if single break mode
  * Normalize every LF and every double [Line](../../Commands/Individual%20commands/Line.md) to be single [Line](../../Commands/Individual%20commands/Line.md) commands without parameters
* Return the result string
