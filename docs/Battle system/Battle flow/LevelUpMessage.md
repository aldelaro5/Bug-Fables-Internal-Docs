# LevelUpMessage
This is a static coroutine that is part of [AddExperience](../Battle%20flow/Terminal%20coroutines/AddExperience.md) when a rank up occurs. Some ranks also have special level up bonuses that are signaled via this method which may end up calling [SetText](../../SetText/SetText.md) in [dialogue mode](../../SetText/Dialogue%20mode.md#dialogue-mode)

> NOTE: This coroutine expects [flagvar](../../Flags%20arrays/flagvar.md) 0 to be set to 0 so the caller can yield on it. It will be set to 1 at the very end to inform the caller of its completion.

## Message setup
The [rank data](../../TextAsset%20Data/Rank%20data.md#rank-data) matching the new instance.`partylevel` is found. If no one exists, this section is skipped and it only applies the last steps.

When one is found, 0.5 seconds are yielded.

The logic that happens here depends on the rank up bonus type:

### 0 (Skill grant)
There will be a message only if there is a player party member that exist with the [animid](../../Enums%20and%20IDs/AnimIDs.md) of whom should receive the skill. 

To setup the message:

- [flagstring](../../Flags%20arrays/flagstring.md) 0 is set to `menutext[46 + X]` where `X` is the animid (this is the name of the player party member)
- [flagstring](../../Flags%20arrays/flagstring.md) 1 is set to the skill's name from [skills data](../../TextAsset%20Data/Skills%20data.md)

### 1 (Stat bonus grant to a player party member)
There will be a message only if there is a player party member that exist with the [animid](../../Enums%20and%20IDs/AnimIDs.md) of whom should receive the stat bonus. This only supports `Attack`, `Defense` and `HP` bonuses.

Regardless if a message is sent:

- The corresponding stat bonus is applied to the player party member via AddStatBonus
- [flagstring](../../Flags%20arrays/flagstring.md) 0 is set to `menutext[46 + X]` where `X` is the animid (this is the name of the player party member)
- [flagstring](../../Flags%20arrays/flagstring.md) 1 is set to the amount of the bonus increase followed by ` ` followed by a `menutext` that depends on the bonus type which is its name (16 for `Attack`, 17 for `Defense` and 14 for `HP`)

### 2 (Stat bonus grant to the whole player party)
There will always be a message. This only supports `TP` and `MP` bonuses.

If this is a `TP` bonus:

- [flagstring](../../Flags%20arrays/flagstring.md) 0 is set to the amount of the bonus increase followed by ` ` followed by `menutext[15]` (TP)
- instance.`tp` is increased by the amount of the bonus
- instance.`maxtp` is increased by the amount of the bonus
- AddStatBonus is called to apply the `TP` bonus to the party, but it will either be called once if the amount is 3 for a single 3 `TP` bonus to the party, or it will be called multiple times if it's not 3 where each calls is a 1 `TP` bonus to the party adding up to the same amount

If this is an `MP` bonus:

- [flagstring](../../Flags%20arrays/flagstring.md) 0 is set to the amount of the bonus increase followed by ` ` followed by `menutext[19]` (MP)
- instance.`bp` is increased by the amount of the bonus
- instance.`maxbp` is increased by the amount of the bonus
- AddStatBonus is called to apply the `MP` bonus to the party with the amount of the bonus in a single call

### 3 (standard items capacity upgrade)
There will always be a message.

- [flagstring](../../Flags%20arrays/flagstring.md) 0 is set to the amount of the inventory upgrade
- instance.`maxitems` is increased by the amount of the inventory upgrade
- [flagstring](../../Flags%20arrays/flagstring.md) 0 is set to the new instance.`maxitems`

### Message display
This part only happens if we decided to show a message.

[SetText](../../SetText/SetText.md) is called in [dialogue mode](../../SetText/Dialogue%20mode.md#dialogue-mode) with the text being `|boxstyle,4||spd,0||halfline||center|` followed by `menutext[141 + X]` where `X` is the type of the rank up bonus. The call has the following properties:

- [fonttype](../../SetText/Notable%20states.md#fonttype) of 0 (`BubblegumSans`)
- linebreak of instance.`messagebreak`
- No tridimensional
- position of Vector3.zero
- No cameraoffset
- size of Vector3.one
- No parent
- No caller

From there, all frames are yielded while the [message](../../SetText/Notable%20states.md#message) lock is grabbed.

After, an additional frame is yielded

## Last steps
This section applies regardless if a rank data was found:

- [ApplyStatBonus](../ApplyStatBonus.md) is called
- A frame is yielded
- [flagvar](../../Flags%20arrays/flagvar.md) 0 is set to 1 which informs the caller that the coroutine completed
