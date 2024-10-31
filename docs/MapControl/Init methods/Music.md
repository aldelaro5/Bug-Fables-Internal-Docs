# Music
This is a parameterless method that ends up calling [ChangeMusic](../../General%20systems/Music%20playback.md#changemusic) according to `music[musicid]` (`musicid` is updated if its value wasn't set). It's what sets up the [map music system](../Map%20music.md)

While it's mainly used by Start, it is also a public method which allows anyone to change `musicid` and to cause a music change accordingly.

Here is what the method does:

- If there's no `music`, ChangeMusic is called with null (silence) at 0.1 fadespeed
- Otherwise:
    - If there's `musicflags`, `musicid` is set to -1 followed by the processing of each music flag in reverse order:
        - The first one with an x of -1 or an x of a flags slot that is true will be selected. Being selected means `musicid` will be set to the y component and the processing is over
    - If `musicid` isn't negative, ChangeMusic is called with `music[musicid]` with 0.1 fadespeed followed by a CheckSamira call on `music[musicid]`
    - Otherwise, ChangeMusic is called with null (silence)
