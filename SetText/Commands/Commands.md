# [SetText](../SetText.md) Commands

A `Commands` is a enum used by [SetText](../SetText.md) that indicates a command to processes something with or without parameters. When specifying the input string, a command and its parameters must be enclosed in `|`. The command name and each of its parameters must be separated by `,`. The command text is refered to anything between both `|` which the command has the ability to replace the text and resume processing at the same position.

Commands also has the ability to perform what's called a redirection which overwrites the input string with a new one and then resumes processing at the start of this new string.

## About case sensitivity

While the actual parsing of the command's name is case insensitive, it is strongly recommended to specify it in all lowercase. This is because there are some logic which are only compatible with the name being in all lowercase notably [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) and some checks such as the commands allowed in [Testdiag](Individual%20commands/Testdiag.md) mode. Not putting it in all lowercase can result in commands being misidentified in some special cases.

Unless specified otherwise, everything is case sensitive.

## About the notes column

The notes contain useful information about each commands, here are what they mean:

* NOT REFERENCED: The enum value exists, but no implementation or references exists, the command name can be parsed without errors, but it will do nothing and so, it does not have a documentation page.
* UNUSED: The command remains functional and has an implementation, but there are no known usage in the game. Whether the command is useful or not can be found in its details page.
* DIALOGUE MODE: The command requires [Dialogue mode](../Dialogue%20mode.md) or works incorrectly in non [Dialogue mode](../Dialogue%20mode.md).
* REGULAR LETTER RENDERING: The command requires to be using [Regular Letter Rendering](../Letter%20Rendering%20Methods/Regular%20Letter%20Rendering.md) or works incorrectly in [Single Letter Rendering](../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md).
* SINGLE LETTER RENDERING: The command requires to be using [Single Letter Rendering](../Letter%20Rendering%20Methods/Single%20Letter%20Rendering.md) or works incorrectly in [Regular Letter Rendering](../Letter%20Rendering%20Methods/Regular%20Letter%20Rendering.md).
* TESTDIAG: The command is supported by [Testdiag](Individual%20commands/Testdiag.md).
* BACKTRACK: The command is supported by the [Backtracking](../Related%20Systems/Backtracking.md) system and its command text may be accumulated or it may directly interact with the system.
* REDIRECT: The command may cause a redirect and resume processing at the start of the new input string.
* REPLACE: The command may replace the command text and resume processing at the same position.
* PROCESSED IN ORGANISELINES: The command can be processed by [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) instead of [SetText](../SetText.md).
* INFLUENCES ORGANISELINES: The command may influence the logic of [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md) without necessarily being processed by it.

# Commands table

There are 217 commands as of 1.1.2.

|ID|Name|Notes|
|--|----|-----|
|0|[String](Individual%20commands/String.md)|REPLACE|
|1|[Var](Individual%20commands/Var.md)|REPLACE|
|2|[Anstring](Individual%20commands/Anstring.md)|REPLACE|
|3|`Checkitem`|NOT REFERENCED|
|4|[Prompt](Individual%20commands/Prompt.md)|DIALOGUE MODE, REDIRECT|
|5|`Getitem`|NOT REFERENCED|
|6|[Line](Individual%20commands/Line.md)|INFLUENCES ORGANISELINES, BACKTRACK, TESTDIAG|
|7|[Next](Individual%20commands/Next.md)|BACKTRACK, TESTDIAG, INFLUENCES ORGANISELINES|
|8|[End](Individual%20commands/End.md)|DIALOGUE MODE|
|9|[Break](Individual%20commands/Break.md)||
|10|[Blank](Individual%20commands/Blank.md)|BACKTRACK, INFLUENCES ORGANISELINES|
|11|[Lock](Individual%20commands/Lock.md)|UNUSED, DIALOGUE MODE|
|12|[Cancelaction](Individual%20commands/Cancelaction.md)|UNUSED|
|13|[Center](Individual%20commands/Center.md)||
|14|[Halfline](Individual%20commands/Halfline.md)|INFLUENCES ORGANISELINES, BACKTRACK, TESTDIAG|
|15|[Stopskip](Individual%20commands/Stopskip.md)|DIALOGUE MODE|
|16|[Noskip](Individual%20commands/Noskip.md)|DIALOGUE MODE|
|17|[Hide](Individual%20commands/Hide.md)|DIALOGUE MODE|
|18|[Rainbow](Individual%20commands/Rainbow.md)|REGULAR LETTER RENDERING, BACKTRACK, TESTDIAG|
|19|[Shaky](Individual%20commands/Shaky.md)|REGULAR LETTER RENDERING, BACKTRACK, TESTDIAG|
|20|[Wavy](Individual%20commands/Wavy.md)|REGULAR LETTER RENDERING, BACKTRACK, TESTDIAG|
|21|[Glitchy](Individual%20commands/Glitchy.md)|REGULAR LETTER RENDERING, BACKTRACK, TESTDIAG|
|22|`Buffer`|NOT REFERENCED|
|23|[Overfollower](Individual%20commands/Overfollower.md)||
|24|[Choicewave](Individual%20commands/Choicewave.md)||
|25|[Spd](Individual%20commands/Spd.md)||
|26|[Speed](Individual%20commands/Speed.md)||
|27|[Color](Individual%20commands/Color.md)|REGULAR LETTER RENDERING, TESTDIAG|
|28|[Anim](Individual%20commands/Anim.md)||
|29|[Sort](Individual%20commands/Sort.md)|REGULAR LETTER RENDERING|
|30|[Save](Individual%20commands/Save.md)|REPLACE|
|31|[Parent](Individual%20commands/Parent.md)||
|32|[Tail](Individual%20commands/Tail.md)|DIALOGUE MODE, BACKTRACK, TESTDIAG|
|33|[Flag](Individual%20commands/Flag.md)||
|34|[Checkmoney](Individual%20commands/Checkmoney.md)|REDIRECT|
|35|[Checkinvqtd](Individual%20commands/Checkinvqtd.md)|REDIRECT|
|36|[Additemtoss](Individual%20commands/Additemtoss.md)|REDIRECT|
|37|[Additem](Individual%20commands/Additem.md)||
|38|[Money](Individual%20commands/Money.md)||
|39|[Goto](Individual%20commands/Goto.md)|REDIRECT, BACKTRACK, TESTDIAG|
|40|[Currency](Individual%20commands/Currency.md)|REPLACE|
|41|[Kill](Individual%20commands/Kill.md)||
|42|[Boxstyle](Individual%20commands/Boxstyle.md)|DIALOGUE MODE|
|43|[Pickitem](Individual%20commands/Pickitem.md)|DIALOGUE MODE, REDIRECT|
|44|[Checktrue](Individual%20commands/Checktrue.md)|REDIRECT|
|45|[Removeitem](Individual%20commands/Removeitem.md)||
|46|[Getstorage](Individual%20commands/Getstorage.md)|REPLACE|
|47|[Button](Individual%20commands/Button.md)|REGULAR RENDERING, INFLUENCE ORGANISELINES, BACKTRACK|
|48|[Movewait](Individual%20commands/Movewait.md)||
|49|[Move](Individual%20commands/Move.md)||
|50|[Forcewait](Individual%20commands/Forcewait.md)||
|51|[Wait](Individual%20commands/Wait.md)||
|52|[Face](Individual%20commands/Face.md)||
|53|[Camtarget](Individual%20commands/Camtarget.md)||
|54|[Flip](Individual%20commands/Flip.md)||
|55|[Warp](Individual%20commands/Warp.md)||
|56|[Transfer](Individual%20commands/Transfer.md)||
|57|[NumberPrompt](Individual%20commands/NumberPrompt.md)|DIALOGUE MODE, REDIRECT|
|58|[Common](Individual%20commands/Common.md)|REPLACE|
|59|[Checkvar](Individual%20commands/Checkvar.md)|REDIRECT|
|60|[Fwait](Individual%20commands/Fwait.md)||
|61|[FadeIn](Individual%20commands/FadeIn.md)||
|62|[FadeOut](Individual%20commands/FadeOut.md)||
|63|`LeafIn`|NOT REFERENCED|
|64|`LeafOut`|NOT REFERENCED|
|65|[Align](Individual%20commands/Align.md)||
|66|[Discovery](Individual%20commands/Discovery.md)||
|67|[size](Individual%20commands/size.md)|INFLUENCE ORGANISELINES, BACKTRACK|
|68|[Minibubble](Individual%20commands/Minibubble.md)||
|69|[Destroyminibubble](Individual%20commands/Destroyminibubble.md)||
|70|[Halt](Individual%20commands/Halt.md)||
|71|[Waitminibubble](Individual%20commands/Waitminibubble.md)||
|72|[Showmoney](Individual%20commands/Showmoney.md)||
|73|[Hidemoney](Individual%20commands/Hidemoney.md)||
|74|[Innsleep](Individual%20commands/Innsleep.md)||
|75|[Event](Individual%20commands/Event.md)||
|76|[Checkregional](Individual%20commands/Checkregional.md)|REDIRECT|
|77|[Checkflag](Individual%20commands/Checkflag.md)|REDIRECT|
|78|[Regionalflag](Individual%20commands/Regionalflag.md)||
|79|[Createitem](Individual%20commands/Createitem.md)||
|80|[Addboard](Individual%20commands/Addboard.md)|UNUSED|
|81|`Openboard`|NOT REFERENCED|
|82|[Camangle](Individual%20commands/Camangle.md)||
|83|[Camoffset](Individual%20commands/Camoffset.md)||
|84|[Completequest](Individual%20commands/Completequest.md)||
|85|[Activateselectedquest](Individual%20commands/Activateselectedquest.md)||
|86|[Resetcamera](Individual%20commands/Resetcamera.md)||
|87|[Quarterline](Individual%20commands/Quarterline.md)|INFLUENCES ORGANISELINES, BACKTRACK, TESTDIAG|
|88|[Questprompt](Individual%20commands/Questprompt.md)||
|89|[Heal](Individual%20commands/Heal.md)||
|90|[Lockmovement](Individual%20commands/Lockmovement.md)||
|91|[Teleportparty](Individual%20commands/Teleportparty.md)||
|92|[Flagvalue](Individual%20commands/Flagvalue.md)|REPLACE|
|93|[Removebadgeshop](Individual%20commands/Removebadgeshop.md)||
|94|[Savecamera](Individual%20commands/Savecamera.md)||
|95|[Loadcamera](Individual%20commands/Loadcamera.md)||
|96|[Camspeed](Individual%20commands/Camspeed.md)||
|97|[Stars](Individual%20commands/Stars.md)|REGULAR RENDERING|
|98|[Giveitem](Individual%20commands/Giveitem.md)|REDIRECT|
|99|[Tailextra](Individual%20commands/Tailextra.md)|DIALOGUE MODE, BACKTRACK, TESTDIAG|
|100|[Setvar](Individual%20commands/Setvar.md)||
|101|[Gettail](Individual%20commands/Gettail.md)|DIALOGUE MODE, BACKTRACK|
|102|[Jump](Individual%20commands/Jump.md)||
|103|[position](Individual%20commands/position.md)|UNUSED|
|104|[Hidespeed](Individual%20commands/Hidespeed.md)|DIALOGUE MODE|
|105|[Breakflag](Individual%20commands/Breakflag.md)|UNUSED|
|106|[Bleep](Individual%20commands/Bleep.md)|DIALOGUE MODE|
|107|[Exitgame](Individual%20commands/Exitgame.md)|UNUSED|
|108|[Openpause](Individual%20commands/Openpause.md)||
|109|[Font](Individual%20commands/Font.md)||
|110|[Exp](Individual%20commands/Exp.md)||
|111|[Icon](Individual%20commands/Icon.md)|REGULAR RENDERING, BACKTRACK, TESTDIAG|
|112|[Dropshadow](Individual%20commands/Dropshadow.md)||
|113|[Level](Individual%20commands/Level.md)||
|114|[Clonestring](Individual%20commands/Clonestring.md)||
|115|[Pauseline](Individual%20commands/Pauseline.md)|INFLUENCES ORGANISELINES, BACKTRACK|
|116|[Addfollower](Individual%20commands/Addfollower.md)||
|117|[Fadeletter](Individual%20commands/Fadeletter.md)|REGULAR RENDERING|
|118|[Setprize](Individual%20commands/Setprize.md)||
|119|[Libraryline](Individual%20commands/Libraryline.md)|INFLUENCES ORGANISELINES, BACKTRACK|
|120|[Shopline](Individual%20commands/Shopline.md)|INFLUENCES ORGANISELINES, BACKTRACK|
|121|[Shakecamera](Individual%20commands/Shakecamera.md)||
|122|[Removemaplimits](Individual%20commands/Removemaplimits.md)||
|123|[Resetmaplimits](Individual%20commands/Resetmaplimits.md)||
|124|[Music](Individual%20commands/Music.md)||
|125|[Sound](Individual%20commands/Sound.md)||
|126|[Destroydescbox](Individual%20commands/Destroydescbox.md)||
|127|[Resetregion](Individual%20commands/Resetregion.md)||
|128|[Mapflag](Individual%20commands/Mapflag.md)||
|129|[Checkmapflag](Individual%20commands/Checkmapflag.md)|REDIRECT|
|130|[Kinematicplayer](Individual%20commands/Kinematicplayer.md)||
|131|[Removefollower](Individual%20commands/Removefollower.md)||
|132|[Shoppool](Individual%20commands/Shoppool.md)||
|133|[Optiontovar](Individual%20commands/Optiontovar.md)||
|134|[Librarybreak](Librarybreak.md)|NOT REFERENCED (This is processed by PauseMenu)|
|135|[Setbreak](Individual%20commands/Setbreak.md)||
|136|[Librarybook](Individual%20commands/Librarybook.md)||
|137|[Area](Individual%20commands/Area.md)|REPLACE|
|138|[Sstring](Individual%20commands/Sstring.md)|PROCESSED IN [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md)|
|139|[Menu](Individual%20commands/Menu.md)|PROCESSED IN [OrganiseLines](../Related%20Systems/Automatic%20Line%20Breaks/OrganiseLines.md)|
|140|[Battle](Individual%20commands/Battle.md)||
|141|[Removestat](Individual%20commands/Removestat.md)||
|142|[Addstat](Individual%20commands/Addstat.md)||
|143|[Checkpos](Individual%20commands/Checkpos.md)|REDIRECT|
|144|[Triui](Individual%20commands/Triui.md)|REGULAR RENDERING|
|145|[Backline](Individual%20commands/Backline.md)|REGULAR RENDERING|
|146|[Cardbattle](Individual%20commands/Cardbattle.md)||
|147|[Boxspeed](Individual%20commands/Boxspeed.md)|DIALOGUE MODE|
|148|[Battlewon](Individual%20commands/Battlewon.md)|REDIRECT|
|149|[Switch](Individual%20commands/Switch.md)||
|150|[Checkminibubble](Individual%20commands/Checkminibubble.md)|REDIRECT|
|151|[Breakend](Individual%20commands/Breakend.md)|TESTDIAG|
|152|[DungeonGame](Individual%20commands/DungeonGame.md)||
|153|[BeeGame](Individual%20commands/BeeGame.md)||
|154|[Copyvar](Individual%20commands/Copyvar.md)|UNUSED|
|155|[Addvar](Individual%20commands/Addvar.md)||
|156|[Showtokens](Individual%20commands/Showtokens.md)||
|157|[Call](Individual%20commands/Call.md)|REPLACE, TESTDIAG|
|158|[Igcolmove](Individual%20commands/Igcolmove.md)|DIALOGUE MODE|
|159|[PartyGame](Individual%20commands/PartyGame.md)|UNUSED|
|160|[LetterPrompt](Individual%20commands/LetterPrompt.md)|DIALOGUE MODE, REDIRECT|
|161|[Define](Individual%20commands/Define.md)|DIALOGUE MODE|
|162|[Loadmap](Individual%20commands/Loadmap.md)||
|163|[Checkanim](Individual%20commands/Checkanim.md)|REDIRECT|
|164|[Camlimit](Individual%20commands/Camlimit.md)|UNUSED|
|165|[Waitcn](Individual%20commands/Waitcn.md)|UNUSED|
|166|[Faketail](Individual%20commands/Faketail.md)|TESTDIAG|
|167|[Unpauseline](Individual%20commands/Unpauseline.md)|INFLUENCES ORGANISELINES, BACKTRACK|
|168|[Cberrytotal](Individual%20commands/Cberrytotal.md)|REPLACE|
|169|[Medaltotal](Individual%20commands/Medaltotal.md)|REPLACE|
|170|[Single](Individual%20commands/Single.md)||
|171|[Tab](Individual%20commands/Tab.md)|SINGLE LETTER RENDERING|
|172|[Singlebreak](Individual%20commands/Singlebreak.md)|INFLUENCES ORGANISELINES, SINGLE LETTER RENDERING|
|173|[Librarysize](Individual%20commands/Librarysize.md)|INFLUENCE ORGANISELINES, BACKTRACK|
|174|[Mothfly](Individual%20commands/Mothfly.md)|REPLACE|
|175|[Chapterintro](Individual%20commands/Chapterintro.md)||
|176|[Backbox](Individual%20commands/Backbox.md)||
|177|[Testdiag](Individual%20commands/Testdiag.md)|UNUSED|
|178|[Lore](Individual%20commands/Lore.md)|REPLACE|
|179|[Follow](Individual%20commands/Follow.md)||
|180|[Transitionsort](Individual%20commands/Transitionsort.md)||
|181|[Moveahead](Individual%20commands/Moveahead.md)||
|182|[Addquest](Individual%20commands/Addquest.md)||
|183|[Textangle](Individual%20commands/Textangle.md)||
|184|[Particle](Individual%20commands/Particle.md)||
|185|[Itemname](Individual%20commands/Itemname.md)||
|186|[Addprize](Individual%20commands/Addprize.md)||
|187|[Entityalive](Individual%20commands/Entityalive.md)|REDIRECT|
|188|[Scorecheck](Individual%20commands/Scorecheck.md)||
|189|[Fademusic](Individual%20commands/Fademusic.md)||
|190|[Limit](Individual%20commands/Limit.md)|UNUSED, INFLUENCES ORGANISELINES|
|191|[Termacadecheck](Individual%20commands/Termacadecheck.md)|REDIRECT, BACKTRACK|
|192|[Removeitemat](Individual%20commands/Removeitemat.md)||
|193|[Unpausesize](Individual%20commands/Unpausesize.md)|INFLUENCE ORGANISELINES, BACKTRACK|
|194|[Alwaysactive](Individual%20commands/Alwaysactive.md)||
|195|[Lockbacktrack](Individual%20commands/Lockbacktrack.md)|BACKTRACK|
|196|[Fixchompy](Individual%20commands/Fixchompy.md)||
|197|[Updateanim](Individual%20commands/Updateanim.md)||
|198|[Checkallquests](Individual%20commands/Checkallquests.md)|REDIRECT|
|199|[Takeopenquests](Individual%20commands/Takeopenquests.md)||
|200|[Deathsmoke](Individual%20commands/Deathsmoke.md)||
|201|[Emoticon](Individual%20commands/Emoticon.md)||
|202|[Checksum](Individual%20commands/Checksum.md)|REDIRECT, BACKTRACK|
|203|[Questsize](Individual%20commands/Questsize.md)|INFLUENCE ORGANISELINES, BACKTRACK|
|204|[Questbreak](Individual%20commands/Questbreak.md)|UNUSED|
|205|[Mapsize](Individual%20commands/Mapsize.md)|INFLUENCE ORGANISELINES, BACKTRACK|
|206|[Caravanmedal](Individual%20commands/Caravanmedal.md)|REDIRECT, BACKTRACK|
|207|`Name`|NOT REFERENCED|
|208|[Itemvalue](Individual%20commands/Itemvalue.md)||
|209|[Sizemulti](Individual%20commands/Sizemulti.md)|INFLUENCE ORGANISELINES, BACKTRACK|
|210|[Ignorenext](Individual%20commands/Ignorenext.md)|UNUSED|
|211|[Rerollshops](Individual%20commands/Rerollshops.md)||
|212|[Maxmedals](Individual%20commands/Maxmedals.md)|REPLACE|
|213|[Battlesize](Individual%20commands/Battlesize.md)|BACKTRACK|
|214|[Pausesize](Individual%20commands/Pausesize.md)|INFLUENCE ORGANISELINES, BACKTRACK|
|215|[GetFromMap](Individual%20commands/GetFromMap.md)|REPLACE|
|216|[Listsize](Individual%20commands/Listsize.md)|INFLUENCE ORGANISELINES, BACKTRACK|
