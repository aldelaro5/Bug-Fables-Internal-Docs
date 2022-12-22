# Font

Change the current [fonttype](../../fonttype.md) used by SetText from now on.

## Syntax

(1)

````
|font,fonttype|
````

(2)

````
|font,fonttype,lockcontrol|
````

## Parameters

### `fonttype`: int

The [fonttype](../../fonttype.md) to set the current one to. This must be a valid [fonttype](../../fonttype.md) or an exception will be thrown.

### `lockcontrol`: `lock` | var

Set whether or not to honor [fonttype](../../fonttype.md) overrides rules concerning the current [languageid](../../languageid.md) during [life cycle > LF processing](../../life%20cycle.md#lf-processing). A value of `lock` indicates to not respect them while any other value indicates to honor them. This can be toggled back and forth during the SetText call, but the default at the start of the SetText call is to honor these rules.

The default value is to not change this setting and keep using the current state of it.

## Remarks

Normally, during the [life cycle > Setup](../../life%20cycle.md#setup) phase, the font gets overridden to `Uzura` in `Japanese` and to `ONEMobilePOP` in `Korean`, but this command allows to override this behavior. It can even disregard it completely by placing this command at the start of the input string.

However, doing so would still allow [life cycle > LF processing](../../life%20cycle.md#lf-processing) to override the [fonttype](../../fonttype.md) to the same values under the same languages if in [Dialogue mode](../../Dialogue%20mode.md) while not processing a [NumberPrompt](NumberPrompt.md). The `lockcontrol` allows to also override this which effectively mean that this command allows complete control over which [fonttype](../../fonttype.md) to use regardless of the [languageid](../../languageid.md).
