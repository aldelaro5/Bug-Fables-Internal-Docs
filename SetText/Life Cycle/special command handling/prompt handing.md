* If `questboardpromp` is true, store into a local var called ad the string `((instance.promptpick != 0) ? "|break||loadcamera||end|" : ("|activateselectedquest||break|" + (instance.flags[64] ? string.Empty : ("|tail,null||blank||boxstyle,4|" + menutext[170] + "|flag,64,true||break|")) + "|loadcamera||end|"));` otherwise, leave it as empty string
  * if [flagvar](../../../Flags%20arrays/flagvar.md) 0 is -55, set the [textbox](../../Notable%20local%20variable/textbox.md) to unhide itself.
  * Override the input string to be the selected prompt's choice's text prepended with `|blank|`.
  * If `linebreak` is not null, [OrganiseLines](../../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) the input string + ad
  * Sets the `promptbox` to hide itself and be destroyed in 0.5 seconds
  * Set `prompttick` to -1
  * Sets the current char index to -1 which will process the first one on the next loop iteration
  * yield a frame
