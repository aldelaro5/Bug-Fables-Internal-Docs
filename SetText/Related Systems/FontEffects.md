# FontEffects

Bug Fables contains a component called [FontEffects](FontEffects.md) which is meant to be applied to a TextMesh corresponding to a letter. It allows through a very simple state machine to render the letter with fancy visual effects.

This state machine simply offers boolean fields that enables or disable the effect on the letter with the ability to receive an integer value as variant. The idea of the variant is to offset the way the effect is applied so multiple letters looks nice together.

Here are what the effects this component supports via [SetText](../SetText.md) [Commands](../Commands/Commands.md) (the field name of the FontEffects is the same than the command name in lowercase):

* [wavy](../Individual%20commands/Wavy.md): Make the mesh move in wave using oscillation functions through time.
* [shaky](../Individual%20commands/Shaky.md): Make the mesh displace itself randomly in a range from its starting position every frame.
* [rainbow](../Individual%20commands/Rainbow.md): Render the mesh with a color that makes multiple meshes in a row look like a flowing rainbow.
* [glitchy](../Individual%20commands/Glitchy.md): Have the mesh change its text to a random letter among a set with 0.125% chance every frame. The set is `'@', '?', '!', '#', '*', '+', '-', '$', '&'`. Once a letter has changed, it has 10% chance to restore itself every frame. Additionally, there is a more aggressive version of this effect called `superglitch` that has 20% to change the letter each frame instead which is accessible via a parameter to the glitchy command.
* [fadeletter](../Individual%20commands/Fadeletter.md): The letter starts to be fully transparent and slowly fades into the original color the letter had.

Additionally, it supports the ability to `rotate`, randomly by 0, 90, 180 or 270 degrees every frames, but this has to be configured manually and it is not directly accessible via a command.

Since the FontEffects was intended to work per letter, they do not work in [Single Letter Rendering](../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md). The commands that uses the effects will not do any operation if they are processed in [Single Letter Rendering](../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md) mode with the exception of [rainbow](../Individual%20commands/Rainbow.md) which will render, but incorrectly.
