# Talk
Calls [SetText](../../../SetText/SetText.md) in [Dialogue mode](../../../SetText/Dialogue%20mode.md#dialogue-mode) using the text from the return of [GetDialogue](../Notable%20methods/GetDialogue.md). It also gives the NPC a chatting emoticon when the NPC is in the `npc` interact list of the player.

## args meaning
None.

## Interact
Calls [SetText](../../../SetText/SetText.md) in [dialogue mode](../../../SetText/Dialogue%20mode.md) with the input string being the current one obtained with [GetDialogue](../Notable%20methods/GetDialogue.md):

- The [fonttype](../../../SetText/Notable%20states.md#fonttype) is `BubblegumSans`
- The default `messagebreak` is used as the linebreak
- No tridimensional
- No render or camera offsets
- size is Vector3.one
- This transform as the parent and this NPCControl as the caller

Once the call is started right after the first yield, all `playerdata` entities have [FaceTowards](../../EntityControl/EntityControl%20Methods.md#facetowards) called on them with the position being this position and with forceback.

## MapControl.LateUpdate
With the exception of the `Kabbu` [animid](../../../Enums%20and%20IDs/AnimIDs.md) and the `keepobjectsalive` being true on the map, all NPC gets their entity.`emoticon` disabled whenever their gameObject enablement needs to be changed (which it does if the x/z distance between the NPC and the player is less than double the `radius` or not).

This behavior prevents this from happening meaning.

TODO: Need to revisit this when MapControl docs is done

## PlayerControl.LateUpdate
entity.`emoticonid` gets set to 0 (the chatting icon) and entity.`emoticoncooldown` gets set to 2.0 if all of the following conditions are met:

- We aren't in a `pause` or `minipause`
- The [message](../../../SetText/Notable%20states.md#message) lock is released
- The player isn't `digging`, `flying`, `startdig` or in `shield`
- This is the closest NPC in the `npc` list as of the last 3 frames