# Unused fields
This page only contains unused fields either because nobody references them or because the fields is technically used, but not in a semantically useful way in practice.

## Unreferenced fields
All these fields are never referenced and are therefore completely unknown besides their types and names.

|Name|type|Is static?|Is Private?|
|---:|-----|----------|----------|
|hudmovement|Coroutine|Yes|No|
|shadowcaster|Shader|Yes|No|
|reserveplayers|List<BattleData>|No|No|
|dynamicfont|Sprite[]|No|No|
|totalenemies|const int = 4|No|No|
|flaglimit|const int = 10|No|No|
|diaglimit|const int = 20|No|No|
|maxdata|const int = 10|No|No|
|max9box|const int = 5|No|No|
|maxitemtypes|const int = 1|No|No|
|totalitems|const int = 256|No|No|
|moneymax|const int = 999|No|No|
|hpmax|const int = 999|No|No|
|tpmax|const int = 99|No|No|
|regionalammount|const int = 100|No|No|
|maxdamage|const int = 99|No|No|
|chargelimit|const int = 3|No|No|
|maxfpsv|const int = 2|No|No|
|saveslots|const int = 3|No|No|
|maxletters|const int = 500|No|No|
|itemtime|const int = 600|No|No|
|levelcap|const int = 27|No|No|
|lowhp|const int = 4|No|No|
|defaultpartqueue|const int = 3000|No|No|
|beehs|const int = 4500|No|No|
|mkhs|const int = 9500|No|No|
|maxmedals|const int = 120|No|No|
|kerning|const float = -2.0|No|No|
|lineheight|const float = 0.7|No|No|
|spacing|const float = 0.3|No|No|
|punctuationdelay|const float = 0.15|No|No|
|defaultcamspeed|const float = 0.1|No|No|
|defaultcamoffsetspeed|const float = 0.1|No|No|
|defaultcamanglespeed|const float = 0.1|No|No|
|letterfixer|const float = 0.07|No|No|
|wordmulti|const float = 25.0|No|No|
|cookspeed|const float = 2.5|No|No|
|gamemonitor|Transform|Yes|No|
|rtex|RenderTexture|No|Yes|
|joytest|bool|Yes|No|
|basefont|int|Yes|No|
|existCheck|List<int>|Yes|Yes|
|distancemultiplier|const float = 1.0|No|No|
|holdkeytime|Coroutine|Yes|No|
|holdkeytime|const float = 7.0|No|No|
|colormultiplier|const float = 2.0|No|Yes|
|deadzone|const float = 0.5|No|Yes|
|defaultcolorspeed|const float = 5.9|No|No|
|defaultsounddistance|const float = 25.0|No|No|

## Semantically unused fields
All these fields are referenced in some ways, but aren't semantically used in practice. Check the details columns for how the field is references, but still not used.

|Name|type|Is static?|Is Private?|Details|
|---:|-----|----------|----------|-------|
|lasttextcenter|float|Yes|No|During the SetText char loop, when processing a regular character in regular letter rendering, this fiels is set to position.x - maxlength / 2.0. While the value is technically used to set the `centerx` field of a ButtonSprite on their Start, `centerx` is never read so it makes this both fields semantically unused|
|textwidth|float|Yes|No|During the SetText char loop, when processing a regular character in regular letter rendering, this fiels is set to maxlength. While the value is technically used to set the `textsize` field of a ButtonSprite on their Start, `textsize` is never read so it makes both fields semantically unused|
|letters|char[]|Yes|Yes||Set to the return of the `Data/Letters` resource on [LoadEssentials](Boot%20and%20reset%20process.md#loadessentials) with ToString().ToCharArray(). This asset and the data is never used, but it contains 319 different characters|
|musicchannel|int|Yes|No|Used in 2 UNUSED overload of ChangeMusicVolume as the clipid parameter. Since that parameter is always 0 under normal gameplay (all [music](../General%20systems/Music%20playback.md) are normally played on `music[0]`), that field isn't used|
|lastinside|int|Yes|No|Set to the last [inside](../MapControl/Insides.md) id entered, but the game doesn't use this information in any meaningful ways|
|battlenoexp|bool|Yes|No|Set to false on [StartBattle](../Battle%20system/StartBattle.md) and to true only if `expreward` is 0 on [ReturnToOverworld](../Battle%20system/Battle%20flow/Terminal%20coroutines/ReturnToOverworld.md), but the game never reads the value|
|debugenalbed|bool|Yes|No|The game never writes to this field, but if the value were to be true, it would allow to load any map entity whose name contains `debug` as those aren't normally loaded in [CreateEntities](../MapControl/Init%20methods/CreateEntities.md)|
|vectorflags|Vector3[]|No|No|Set to a new array of 10 elements on SetVariables, but the game never acceses the array|
|intransition|bool|No|No|Set to true at the start of [Transition](../General%20systems/Transition%20system.md) and to false at the end of it so it indicates if a transition is in progress, but the game never reads this field|
|hudstats|bool|No|No|BattleControl.LateUpdate reads this field twice (once guarding an empty code block), but the game never writes true to this field so it's not effectively used. If it was true, it would cause the method to set `hudcooldown` to -100.0 if the [message](../SetText/Notable%20states.md#message) lock is grabbed|
|speedup|bool|No|No|Set to true at the start of [ApplyBadges](../General%20systems/Player%20party.md#applybadges) and when applying a `SpeedUp` BadgeEffects (which is also UNUSED), but because the game never reads this field, nothing happens|
|camoffsetspeed|float|No|No|Set before and after an event, on ResetCamera and on PlayerControl.DefaultCamOffset (an UNUSED method), but the game never reads this field|
|timeddemo|bool|Yes|No|The game never writes to this field, but if the value were to be true, it would prevent any `save` [interaction](../Entities/NPCControl/Interaction.md) which prevents to save the game and the moment [DoClock](DoClock.md) notices that `demotimer` exceed 900 (it gets incremented every call so every in game seconds) and FreePlayer returns true while not in a `battle`, event 187 is started (an UNUSED event). This events ultimately ends with a game [Reset](Boot%20and%20reset%20process.md#reset) while presenting a SetText message thanking the player for playing "this demo". This suggests this field guarded a now UNUSED feature where the game could be played in a 15 minutes demo before reseting itself|
|demotimer|int|No|Yes|See the `timeddemo` field explanation above, but it would only have been used if `timeddemo` is true which isn't normally possible. This would act as a seconds counter up to 900 seconds or 15 minutes before event 187 is started|
