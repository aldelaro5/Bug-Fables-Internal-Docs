# MainManager Fields
This page documents every fields on MainManager, except the ones that are unused. For those, consult the [unused MainManager fields](Unused%20fields.md) documenation.

## [Camera system](../General%20systems/Camera%20system.md)

|Name|type|Is static?|Is Private?|Description|
|---:|-----|----------|------------|------------|
|MainCamera|Camera|Yes|No|The main camera that renders the `Default` layer|
|GUICamera|Camera|Yes|No|The camera that renders the `UI` layer|
|globalcamdir|Transform|No|No|The `CamDir` GameObject|
|downsample|int|Yes|No|The index of `downsamples` to use for rendering the `MainCamera.targettexture` by SetRenderTexture. This is persisted to config.dat, but the setting isn't accessible under normal gameplay so it's normally always 0 which is no downsampling|
|downsamples|float[]|Yes|No|The available size scaler to use for rendering the `MainCamera.targettexture` by SetRenderTexture using the element in this array at index `downsample`. The default value is hardcoded to {1.0, 0.9, 0.8, 0.75, 0.6, 0.5, 0.4}|
|camtarget|Transform|No|No|If set to not null, the camera will try the look at this transform and follow them|
|defaultcamoffset|Vector3|Yes|No|A hardcoded value that represents the default and initial value of `camoffset`. The value is hardcoded to (0.0, 2.25, -8.25)|
|defaultcamangle|Vector3|Yes|No|A hardcoded value that represents the default and initial value of `camangleoffset`. The value is hardcoded to (10.0, 0.0, 0.0)|
|camoffset|Vector3|No|No|The current postion offset the camera is from its looking target. The default value is `defaultcamoffset`|
|camoffset2|Vector3|No|No|A secondary postion offset the camera is from its looking target that gets added to `camoffset`. It is used to smoothly offset the camera during player movement|
|camangleoffset|Vector3|No|No|The current angular offset the camera is from its looking target. The default value is `defaultcamangle`|
|camspeed|float|No|No|A lerp factor used to move the camera to the desired postion and if `camanglechange` is false, it is also the lerp factor when rotating the camere while `map.rotatecam` is false. The default value is hardcoded to 0.1|
|camtargetpos|Vector3?|No|No|The position the camera is set to look at or null if the camera isn't set to look at a particular point|
|camanglechange|bool|No|No|Indicate whether or not to use `camanglespeed` (if it's true) or `camspeed` (if it's false) as the angle lerp factor when rotating the camera while `map.rotatecam` is false|
|changecamspeed|bool|No|No|If this is true, it prevents the `camspeed` from being set to 0.1 during PlayerControl's Movement when going towards the camera. This is only set to true as part of a CameraChange with a custom speed configured. The value is reset to false on ResetCamera and on LoadMap|
|camanglespeed|float|No|No|The lerp factor used when rotating the camera when `camanglechange` is true while `map.rotatecam` is false|
|screenshake|Vector3|Yes|No|A vector that is used to generate a random position displacement of the camera every time RefreshCamera is called (happens every FixedUpdate after `basicload`). The range is generated from -`X` to `X` where `X` is the value of this vector. If the value is Vector3.zero, no displacement occurs|
|camposshake|Vector3|Yes|Yes|The `camoffset` value saved on the last ShakeScreen called which is meant to be restored when StopScreenShakeReturn is called which only happens if ShakeScreen was called with dontreset set to false|
|tempcamtarget|Transform|Yes|No|The value of `camtarget` saved when SaveCameraPosition(true) is called. This is meant to be restored to `camtarget` when SaveCameraPosition(false) is called|
|tempcamoffset|Vector3|Yes|No|The value of `camoffset` saved when SaveCameraPosition(true) is called. This is meant to be restored to `camoffset` when SaveCameraPosition(false) is called|
|tempcamangleoffset|Vector3|Yes|No|The value of `camangleoffset` saved when SaveCameraPosition(true) is called. This is meant to be restored to `camangleoffset` when SaveCameraPosition(false) is called|
|tempcamspeed|float|Yes|No|The value of `camspeed` saved when SaveCameraPosition(true) is called. This is meant to be restored to `camspeed` when SaveCameraPosition(false) is called|
|tempcampos|Vector3?|Yes|No|The value of `camtargetpos` saved when SaveCameraPosition(true) is called. This is meant to be restored to `camtargetpos` when SaveCameraPosition(false) is called|
|tempmaplp|Vector3|Yes|No|The value of `map.camlimitpos` saved when SaveCameraPosition(true) is called. This is meant to be restored to `map.camlimitpos` when SaveCameraPosition(false) is called|
|tempmapln|Vector3|Yes|No|The value of `map.camlimitneg` saved when SaveCameraPosition(true) is called. This is meant to be restored to `map.camlimitneg` when SaveCameraPosition(false) is called|

## [ItemList](../ItemList/ItemList.md)

|Name|type|Is static?|Is Private?|Description|
|---:|-----|----------|------------|------------|
|itemlist|Transform|No|No|The current ItemList being rendered or null if no ItemList is rendering|
|multiselect|List<int>|No|No|The list of selected items in the current ItemList|
|inlist|bool|No|No|Indicates whether or not there's an ItemList in progress. Unlike `itemlist` not being null, this value tracks the processing of the ItemList while `itemlist` tracks if one is rendered (there's a brief period of time after the ItemList disappears where it needs to be handled which is what this value is tracking because it means the ItemList processing isn't over even if it is no longer rendered). NOTE: This value isn't reset properly when the ItemList was started with a manual call to ShowItemList and it is cause of the inlist issue, check its [documentation](../ItemList/inlist%20issue.md) to leanr more|
|listtype|int|Yes|No|The [listtype](../ItemList/listtype.md) of the last ItemList OR the flagvar slot where the output of the current numberprompt will be stored OR the flagstring slot where the output of the current letterprompt will be stored. This value isn't automatically reset when the ItemList processing is over|
|listsell|bool|Yes|No|Reports the sell parameter of the last ShowItemList call which tells if this is a list used for selling to a merchant. It also serves as a field used to set its value and pass it as the sell parameter. This value isn't automatically reset when the ItemList processing is over|
|listvar|int[]|Yes|No|The generated options list of the current ItemList|
|listammount|int|Yes|No|The amount of elements to show in the visible part of an ItemList. This value isn't automatically reset when the ItemList processing is over|
|listdesc|bool|Yes|No|Reports the showdescription parameter of the last ShowItemList call which tells if a description of the hovered option should be rendered. It also serves as a field used to set its value and pass it as the showdescription parameter. This value isn't automatically reset when the ItemList processing is over|
|listlow|int|Yes|No|The lower index bound of the visible part of an ItemList (visually, it corresponds to the upper bound). This value isn't automatically reset when the ItemList processing is over, calling ResetList or another ShowItemList call will reset it|
|listcursor|int|Yes|No|The relative index of the item within the visible part of an ItemList. The index is relative to `listlow` meaning that a value of 0 means the cursor is at `listlow` and a value of `listammount` - 1 means the cursor is at `listmax`. This is mainly used to position the cursor on the screen. This value isn't automatically reset when the ItemList processing is over|
|listmax|int|Yes|No|The upper index bound of the visible part of an ItemList (visually, it corresponds to the lower bound). This value isn't automatically reset when the ItemList processing is over, calling ResetList or another ShowItemList call will reset it|
|listoption|int|Yes|No|The `option` value of the last ItemList that was destroyed via DestroyList|
|listY|int|Yes|No|The last value of `listlow` when ShowItemList was called which is used to detect if the visible part of the ItemList should be refreshed. If this value becomes different than the actual `listlow`, it means the ItemList needs to be refreshed on the next time ShowItemList is called. This value isn't automatically reset when the ItemList processing is over, calling ResetList or another ShowItemList call will reset it|
|listpos|Vector2|Yes|No|The position parameter of the last ShowItemList call (Vector2.zero if no calls was done yet)|
|storeid|int|Yes|No|The flagvar slot to store the listvar option upon confirmation of an ItemList. It should always be 0 unless specified otherwise by a [pickitem](../SetText/Individual%20commands/Pickitem.md) SetText command. This value isn't automatically reset when the ItemList processing is over|
|listdescbox|Transform|Yes|No|The description box part of the currently rendered `ItemList` or null if no ItemList is being rendered|
|defaultlistpos|Vector2|Yes|No|A hardcoded `GUICamera` local postion that indicates the default placement of an `ItemList` in x and y when calling ShowItemList. The value is hardcoded to (4.0, 0.5)|
|itemlistbg|readonly Color[]|Yes|Yes|An array of 2 colors where the first one represents the background color to render an ItemList's bar when the index isn't in `multiselect` and the second one represents the background color to render the bar when it is in `multiselect`. The value is hardcoded to {(1.0, 1.0, 1.0, 0.75), (0.0, 1.0, 0.0, 0.75)}|
|listcanceled|bool|Yes|No|This tells if the last ItemList, prompt or numberprompt was cancelled after the outcome was determined. This value isn't automatically reset when the ItemList, prompt or numberprompt processing is over, but a call to ShowItemList, prompt command or numberprompt command will reset the value to false|
|listcancel|int|Yes|No|The dialogue line id to redirect SetText when a cancellation is done on an ItemList that was created due to a pickitem command OR the option index during a prompt if a cancel is performed. It has a similar semantic for a numberprompt and for a letterprompt, it's the same value than `listredirect` since a letterprompt cannot be cancelled. This value isn't automatically reset when the ItemList processing is over|
|listredirect|int?|Yes|No|The dialogue line id to redirect SetText when a confirmation is done on an ItemList that was created due to a pickitem command (no redirection is done if the value is null). It has a similar semantic for a numberprompt and letterprompt. This value isn't automatically reset when the ItemList processing is over|
|overridedlist|int[]|Yes|No|The browsing info of the last saved ItemList by `savelastlist`. It is automatically set to null on restore to the next new ItemList call. It contains `option`, `listcursor`, `listlow`, `listmax` and `listoption` in that order. This is only used by CardGame|
|savelastlist|bool|Yes|No|If set to true, the next ItemList confirmation will cause `overridedlist` to be set to the values of the ItemList for restoration on the next ShowItemList call. This value is automatically set to false after setting `overridedlist`|
|multilist|int[]|No|No|An array of `listvar` options used in the following listtype as it is expected that this fields provides the `listvar` option before creating the ItemList: B.O.S.S battles, Chompy ribbons, equipped medals and spy cards. It is reset to a new array on CreateLetterPrompt due to a practically UNUSED purpose|
|caravanorder|int[]|Yes|No|The ordered list of medals id available at the Caravan medals shop, used in the matching listtype|
|settingsindex|int[]|Yes|No|A hardcoded array of all the MenuText indexes of all settings ordered by their matching `listvar` index for use in the settings listtype. The array is hardcoded to {33, 34, 160, 28, 29, 32, 116, 30, 31, 147, 140, 156, 80, 157, 35, 36, 183, 222, 231, 239, 245, 261, 270, 255, 256, 282}|

## [SetText](../SetText/SetText.md)

|Name|type|Is static?|Is Private?|Description|
|---:|-----|----------|------------|------------|
|message|bool|No|No|The SetText message lock that enforces that only one dialogue mode call at a time may execute except in some controlled edgecases (such as the battle command). Essentially, this indicate whether or not a SetText call in dialogue mode is in progress|
|waitinput|bool|No|No|If true, it indicates that SetText is in dialogue mode and is awaiting an input from the player before resuming|
|skiptext|bool|No|No|Indicate whether or not SetText received an input telling it to stop the letter rendering delays in dialogue mode until the end of the textbox|
|isholdingskip|bool|No|No|Indicates whether or not the Cancel input was held for long enough for SetText needs to dispatch its ongoing call in dialogue mode as fast as possible by automatically processing every textbox with much smaller delays than normal. This is always reset to false when calling SetText in dialogue mode|
|noskip|bool|Yes|No|If set to true, it prevents `isholdingskip` to become true as a result of holding the Cancel input and it prevents a Time.timeScale adjustement to happen again after it already happened before during event 3. It is always initially set to false on any SetText call in dialogue mode (or when processing a next or break command), but it is possible to use the noskip command during the call to set it to true. It is always reset to false on EndEvent, but SetText by itself won't reset it to false at the end of the call|
|texttail|Transform|No|No|The tail part of the textbox during the current SetText call in dialogue mode or null if no textbox is being rendered by SetText|
|tailtarget|Transform|No|No|If set to not null, the `texttail` will be rotated to track this transform during the current SetText call in dialogue mode (it's meant to indicate who is talking)|
|maintextbox|GameObject|Yes|No|The textbox being used in the current SetText call in dialogue mode or null if no calls is in progress. It is an instance of `Prefabs/Textbox`|
|textbox|Transform|No|Yes|The textholder of the current SetText call in dialogue mode or null if no calls in dialogue mode is ongoing|
|messagebreak|float|Yes|No|The default linebreak parameter to send to SetText which sets the maximum width of the text that will be rendered. The initial value is hardcoded to 10.5, but if `languageid` is 0 (English) or -1 (unset), the value will be overriden to 9.75 on every LateUpdate cycle|
|itemdescbreak|float|Yes|No|The linebreak parameter to send to SetText when rendering an ItemList's description, CreateDescWindow is called or building a TempItemDesc (this is the maximum width of the text that will be rendered). The initial value is hardcoded to 10.5, but if `languageid` is 0 (English) or -1 (unset), the value will be overriden to 9.9 on every LateUpdate cycle|
|blinker|SpriteRenderer|No|No|A reference to the blinking leaf sprite used in SetText in dialogue mode|
|letterpool|TextMesh[]|Yes|Yes|A pool of 500 TextMesh used to render letters in SetText's regular letter rendering or lines in single letter rendering|
|letteroffset|Vector2|Yes|No|A hardcoded offset position to add after calculating the local position to render any letter during a SetText call (both regular or single letter rendering). The value is hardcoded to (0.0, -0.1)|
|textcolors|Color[]|No|No|An array configured from the Main scene using the Unity inspector. It is expected to have at least 10 elements to reference colors that are meant to be easily accessible by the game for rendering. More information on their normal values in the color SetText command documentation|
|menutext|string[]|Yes|No|The contents of `Data/DialoguesX/MenuText` where `X` is the `languageid` split by LF|
|commondialogue|string[]|Yes|No|The contents of `Data/DialoguesX/CommonDialogue` where `X` is the `languageid` split by LF|
|bleeps|AudioSource|Yes|Yes|A special AudioSource meant only to play a bleep from SetText in dialogue mode or when using the bleep command|
|dsounds|AudioClip[]|Yes|Yes|The contents of `Audio/Sounds/Dialogue` which are kept referenced in this field for caching purposes|
|bleepvolume|float|Yes|No|Indicates the volume of all bleeps sounds playback in the settings between 0.0 (muted) and 1.0 (full volume). This is persisted to config.dat, but the default value before config.dat is loaded is hardcoded to 0.5f|
|define|List<string[]>|Yes|Yes|The SetText defined list of user defined entity names to their entity id association. This is only used in the current SetText call in dialogue mode and it is always reset to a new list on every call in dialogue mode. Each element represents one entity association and each string[] has 2 elements with the first one being the user defined name and the second one being the entity id the name refers to converted to string|
|bubblepos|Vector3|Yes|No|A hardcoded position to send to SetText when presenting the GameOver menu. The value is hardcoded to (0.0, 4.15, 10.0)|
|backtracking|bool|Yes|Yes|Indicate whether or not the current SetText call in dialogue mode is being backtracked. This is always reset to false on any SetText call in dialogue mode or when processing a next command. Check the [backtracking](../SetText/Related%20Systems/Backtracking.md) system documentation to learn more|
|notextbacktrack|bool|Yes|No|If true, it prevents to use the backtracking feature of SetText. It is always initially set to false when SetText is called, but the lockbacktrack command can set it to true for the remainder of the call|
|diagstring|List<string>|Yes|Yes|The SetText dialogue lines cumulated in order for the backtracking system|
|currentdialogue|int|Yes|Yes|The `diagstring` index of the SetText dialogue line that is being seen for the backtracking system|
|tempdiag|string|Yes|Yes|The cumulated text of the current textbox of the current SetText call in dialogue mode. This is always reset to `\|size,X,Y\|` on every SetText call in dialogue mode where `X` and `Y` are the components of the size parameter|
|asiansize|readonly string|Yes|No|A hardcoded string inserted at the start of the text in every SetText call in dialogue mode if `languageid` is 3 (Japanese). The value is hardcoded to `\|size,0.8,0.9\|`|
|linebr|float|Yes|Yes|The last observed value of linebreak during a SetText call in dialogue mode updated at the start of the char loop. This is used for the backtracking system to use as the maxoffset parameter to OrganiseLines when adding the current textbox to `diagstring`|
|fontdsize|float|Yes|Yes|The last observed value of size during a SetText call in dialogue mode updated at the start of the char loop. This is used for the backtracking system to use as the size parameter to OrganiseLines when adding the current textbox to `diagstring`|
|fontdtype|int|Yes|Yes|The last observed value of fonttype during a SetText call in dialogue mode updated at the start of the char loop. This is used for the backtracking system to use as the fontid parameter to OrganiseLines when adding the current textbox to `diagstring`|

## [Prompt](../SetText/Individual%20commands/Prompt.md), [numberprompt](../SetText/Individual%20commands/NumberPrompt.md) and [letterprompt](../SetText/Individual%20commands/LetterPrompt.md)

|Name|type|Is static?|Is Private?|Description|
|---:|-----|----------|------------|------------|
|prompt|bool|No|No|Indicate whether or not there's a prompt, numberprompt or letterprompt in progress|
|option|int|No|No|The hovered option index of an ItemList, prompt, numberprompt or letterprompt|
|maxoptions|int|No|No|The total amount of options of an ItemList, prompt, numberprompt or letterprompt|
|promptpick|int|No|No|The option index at the moment that a prompt or numberprompt was confirmed or cancelled. The default value is hardcoded to -1 which means no prompt or numberprompt needs to be processed. The value is automatically reset to -1 on any SetText call in dialogue mode and it is also reset during SetText when processing a prompt or numberprompt|
|promptpointers|int[]|No|No|The dialogue line ids associated with the current prompt's options so that SetText can redirect the call to `promptpointers[promptpick]`. For a numberprompt, this is set to an array with 1 element being either `listcancel` or `listredirect.value` depening on if Confirm or Cancel (or the entire numberprompt was cancelled) was chosen with `promptpick` being set to 0 to simulate a 1 option prompt that was confirmed so SetText can proceed. For a letterprompt, it is set to an array of 1 element being `listredirect.Value` when the Confirm option is selected with `promptpitck` at 0 which acomplishes a similar purpose. The default value is an empty array configured by Unity|
|lastPrompt|int|No|No|The option index of the last prompt, numberprompt or ItemList that was confirmed. NOTE: This value is never reset and the default value is configured to be 0 from the from the Main scene using the Unity inspector which is incorrect because it's possible that no prompt, numberprompt or ItemList were processed yet|
|promptbox|Transform|No|No|The 9Box where the current prompt, numberprompt or letterprompt is rendered or null if no prompts is currently rendering|
|npromptholder|Transform|No|No|The part of the `promptbox` that renders the numbers for a numberprompt or the letters for a letterprompt (it is childed to `promptbox`) or null if there's no numberprompt or letterprompt currently rendering|
|numberprompt|bool|No|No|Indicate whether or not there's a numberprompt or letterprompt in progress. The value of `flagvar[0]` can be used to differentiate between the 2: if it's -555, it's a letterprompt and if it's not, it's a numberprompt NOTE: This only reset when a prompt command happens, but it only takes effect when `prompt` is true|
|letterprompt|int|No|No|The identifier of the current letterprompt or -1 if no letterprompt is being processed. The default value is configured to be -1 from the from the Main scene using the Unity inspector and it is automatically reset to -1 on a SetText call in dialogue mode or while processing a numberprompt command|
|letterPromptHelp|readonly string[]|Yes|Yes|The help text to display at the bottom part of the current letter prompt ordered in such a way that they are shifted by 1 compared to the `letterprompt` numbers. This is done because it's meant to show the help text of the NEXT letterprompt, not the current one. The value is hardcoded to {"ひらがな", "カタカナ", "Русский", "한국어", "✰ ♡ ♩ $", "ABC/123/?!ß"}|
|koreanHL|int[]|Yes|Yes|The value of `option + 3` that was saved for the first 2 sections of the current Korean letterprompt. See the Korean letterprompt documentation to learn more. The default value is hardcoded to {-1, -1} to mean no Korean letterprompt character inputs is ongoing. It is also reset to this value when confirming the character for the last section, when deleting the last character or when creating a new Korean letterprompt. The value saved is `option` + 3 because all boundaries in `koreanlimi` are off by + 3 and the lower bounds is used to calculate the id of the different character's sections so saving the value this way cancels out this offset|
|koreanLimit|readonly Vector2Int[]|Yes|Yes|The boundaries of the `option` indexes within `letterprompt4` (Korean letterprompt) that forms the 3 different sections (each x component represents the starting index and each y component represents the last index). NOTE: All numbers in this array are off by +3. See the Korean letterprompt documentation to learn more. The value is hardcoded to {(3, 21), (24, 44), (45, 72)}|

## [Medals](../General%20systems/Medals.md) and [items](../Enums%20and%20IDs/Items.md)

|Name|type|Is static?|Is Private?|Description|
|---:|-----|----------|------------|------------|
|items|List<int>[]|No|No|The 3 items inventories of the player:<ul><li>0: Standard items</li><li>1: Key items</li><li>2: Stored items</li></ul> This is persisted to the save file|
|badges|List<int[]>|No|No|The list of medals on hand where each element contains 2 integers where the first is the medal id and the second is the equip target (-2 means unequipped, -1 means equipped to the party and any other number is the animid of whom the medal is equipped to). This is persisted to the save file|
|badgeshops|List<int>[]|No|No|All the medals that each medals shop have in stock:<ul><li>0: Merab</li><li>1: Shades</li></ul> This is meant to indicate all medals that could be purchased, but not necessarily the order they appear or which ones will be displayed. This is persisted to the save file|
|avaliablebadgepool|List<int>[]|No|No|Essentially the same as `badgeshop`, but each list is ordered in the order they will appear in the shop (the first ones are on display and the ones after will start to appear once medals gets purchased). This is more meant to indicate the order of `badgeshops` that are actually shown, but it's still based on `badgeshops` so it doesn't by itself indicate the medals available. This is persisted to the save file|
|badgedata|string[,]|Yes|No|The contents of `Data/BadgeData` split by LF merged with the contents of `Data/DialoguesX/BadgeName` where `X` is the `languageid` split by LF. The first dimension corresponds to a line index in either TextAsset (which is a medal id) and the second dimension is formated in a specific way which is detailed in the medals data TextAsset documentation|
|badgeorder|int[]|Yes|No|The contents of `Data/BadgeOrder` where each line is an element in this array in the same order|
|prizeflags|int[]|No|No|The flags flagvar slots bound to each of the 23 [prize medals](../General%20systems/Medals.md#prize-medals)|
|prizeids|int[]|No|No|The medals id bound to each of the 23 [prize medals](../General%20systems/Medals.md#prize-medals)|
|prizeenemyids|int[]|No|No|The enemies id bound to each of the 23 [prize medals](../General%20systems/Medals.md#prize-medals)|
|itemdata|string[,,]|Yes|No|The contents of `Data/ItemData` split by LF merged with the contents of `Data/DialoguesX/Items` where `X` is the `languageid` split by LF. The first dimension corresponds to a line index in either TextAsset (which is an item id) and the second dimension is formated in a specific way which is detailed in the Items data TextAsset documentation|
|termacadeprize|int[,]|Yes|No|The contents of `Data/Termacade` where each line corresponds to a termacade prize on the first dimension and the second dimensions corresponds to the field index of each line. Essentially, the data is loaded as is in this jagged array without formatting other than converting the strings to integers|
|itemsprites|Sprite[,]|Yes|No|The sprites in `Sprites/Items/items0` and `Sprites/Items/items1` merged together in a specific way. The first dimension groups the sprites per type where 0 are items and 1 are medals. The second dimension corresponds to the matching item or medal id. As for which array to pull the sprite from, it depends on the type:<ul><li>Items: pulls from items0 unless the itemid is 176 or higher where it pulls from items1 at index 176 - the item id</li><li>Medals: pulls from items0 at index 176 + the medal id unless if the matching `badgedata`'s field 8 is not negative where it will pull from items1 at the index specified in that field 8</li></ul> This loading logic is done by LoadItemSprites|

## [HUD](../General%20systems/HUD.md) and UI

|Name|type|Is static?|Is Private?|Description|
|---:|-----|----------|------------|------------|
|pausemenu|PauseMenu|Yes|No|The current PauseMenu when one is created and started (usually happens when pausing, accessing the settings menu from the StartMenu or selecting the change loadout and retry option on GameOver). This is null if no PauseMenu exists|
|hud|Transform[]|No|No|An array of 5 elements containing the rendered HUD elements where the first 3 are each of the party member's HP, the 4th element is the party's TP and the 5th element is the berry count. This is null between the time RebuildHUD is called (only happens on loading a save file or ChangeParty) and the time CreateHUD is called (happens on RefreshHUD when `hud` is null which happens as part of LateUpdate)|
|hudvisible|bool|Yes|No|Indicates whether or not the HUD is visible according to ShowHUD. If the value is false when ShowHUD(true) is called, ForceHUD is called and the value is set to true|
|hudvalue|float[]|Yes|Yes|Set to a hardcoded array on CreateHUD which is {6.5, 7.5, 8.5, 9.5, -6.5, -7.0}|
|hudcooldown|float|No|No|The amount of frames to leave the HP and TP `hud` shown before hiding it. When this reaches 0.0 or below during RefreshHUD, the value is reset to -100.0 which will hide the HUD since it's below 0.0|
|showmoney|float|No|No|The amount of frames to leave the berry count `hud` shown before hiding it. Any values below 0.0 will hide the HUD. The countdown is frozen when the SetText's `message` lock is active|
|hudsprites|SpriteRenderer[]|Yes|Yes|An array of 4 elements containing the rendered `hud`'s sprites where the first 3 are each of the party member's HP and the 4th element is the party's TP|
|hudfont|DynamicFont[]|No|Yes|The DynamicFont used to render the matching number for each `hud` elements (same order as `hud`)|
|discoveryhud|float|No|No|The amount of frames to leave the discovery notification HUD element shown before hiding it. Any values below 0.0 will hide the discovery notification|
|discoverymessage|Transform|No|No|Contain the `Discovery` UI GameObject childed to `Prefabs/Objects/logbookicon` that is created on the first [RefreshDiscovery](../General%20systems/HUD.md#discoverymessage) calls, but shows up onscreen when `discoveryhud` is above 0.0 (left hidden below the screen otherwise)|
|tpt|int|No|No|The displayed value of `tp` in the HUD which is tracked separately to account for the "count up/down" effect|
|moneyt|int|No|No|The displayed value of `money` in the HUD which is tracked separately to account for the "count up/down" effect|
|tmtp|int[]|Yes|Yes|An array of 2 elements where the first one represents the last displayed value of `tpt` and the second the last displayed value of `maxtp`. This is used to detect that the `hud` element's DynamicFont needs to be updated whenever `tpt` or `maxtp` changes. Both elements are updated whenever either of the displayed values is updated and both elements are reset to -1 on ForceHUD. The initial value is hardcoded to an empty array with a length of 2|
|tphp|int[]|Yes|Yes|The last displayed value of all the player party members's `hpt` which is the displayed `hp` value in the HUD. This is used to detect that the `hud` element's DynamicFont needs to be updated whenever a `hpt` changes. This value is updated whenever the displayed value is and reset to -1 on ForceHUD|
|ptmhp|int[]|Yes|Yes|The last displayed value of all the player party members's `maxhp`. This is used to detect that the `hud` element's DynamicFont needs to be updated whenever a `maxhp` changes. This value is updated whenever the displayed value is and reset to -1 on ForceHUD|
|tempmoneh|int|Yes|Yes|The last displayed value of `moneyt`. This is used to detect that the `hud` element's DynamicFont needs to be updated whenever `moneyt` changes. This value is updated whenever the displayed value is and reset to -1 on ForceHUD|
|cursor|SpriteRenderer|No|No|A standard cursor leaf sprite created when calling CreateCursor meant for the caller to manipulate it|
|cursorsprite|Sprite[]|Yes|No|A single element array hardcoded to `guisprites[145]` which is the leaf cursor sprite|
|switchicon|SpriteRenderer|No|No|A UI GameObject childed to the `GUICamera` with a SpriteRenderer used to render the new party leader's icon in the overworld when switching the party|
|transition|Coroutine|Yes|No|The current [Transition](../General%20systems/Transition%20system.md) coroutine call or null if no calls is in progress|
|transitionobj|Transform[]|No|No|The transition objects used when applicable as part of a Transition coroutine|
|letterbox|SpriteRenderer[]|Yes|Yes|An array of 2 SpriteRenderers representing 2 GameObject named `letterbox0` and `letterbox1` which are rectangles of solid colors rendered at the top and bottom of the screen (they are childed to the `GUICamera`). Their color automatically are lerped to pure black when `inevent` and not `inbattle` or to clear otherwise|
|charcolor|Color[]|No|No|An array configured from the Main scene using the Unity inspector. It corresponds to the accent color used for each player party member ordered by their animid which is used throughout the game's UI to associate the party members visually. It is normally configured to be:<ul><li>0: FFC000 (mostly yellow, used for `Bee`)</li><li>1: 00D100 (mostly green, used for `Beetle`)</li><li>2: 00AFE6 (mostly blue, used for `Moth`)</li></ul>|
|menucolors|Color[]|No|No|An array configured from the Main scene using the Unity inspector. It corresponds to some colors used in the game's UI to render various informations. It is normally configured to be:<ul><li>0: FF4B4B (mostly red, used to present HP and berries count)</li><li>1: 18CA00 (mostly green, used to present Attack)</li><li>2: 0080FF (mostly blue, used to present Defense)</li><li>3: FFAE00 (mostly orange, used to present TP)</li><li>4: FFDA00 (mostly yellow, used to present EXP)</li></ul>|
|leafpos|Vector2[]|Yes|No|The contents of `Data/LeafPos` where each line is an element in the array and the components are split by `,` converted to float|
|leafsprites|Sprite[]|Yes|No|The sprites in `Sprites/GUI/battleleaves`|
|guisprites|Sprite[]|Yes|No|The sprites in `Sprites/GUI/gui` appended with the sprites in `Sprites/GUI/gui2`|
|battlemessage|Sprite[]|Yes|No|The sprites in `Sprites/GUI/BattleMessage/battlemX` where `X` is the `languageid`. If there's no sprites in the array or if the array doesn't exist, it falls back to the sprites in `Sprites/GUI/BattleMessage/battlem0`|

## [Sounds](../General%20systems/Sounds%20playback.md)

|Name|type|Is static?|Is Private?|Description|
|---:|-----|----------|------------|------------|
|sounds|AudioSource[]|Yes|No|An array of 15 AudioSources used to play sounds. Simultaneous playback of sounds is supported so long as there's a free AudioSource from this array to use|
|lastsoundid|int|Yes|No|The last `sounds` index used to play a sound via PlaySound if the sound volume isn't muted|
|soundvolume|float|Yes|No|Indicates the volume of all sounds playback in the settings between 0.0 (muted) and 1.0 (full volume). This is persisted to config.dat, but the default value before config.dat is loaded is hardcoded to 0.5f|
|asounds|AudioClip[]|Yes|Yes|The contents of `Audio/Sounds` which are kept referenced in this field for caching purposes|

## [Music](../General%20systems/Music%20playback.md)

|Name|type|Is static?|Is Private?|Description|
|---:|-----|----------|------------|------------|
|music|AudioSource[]|Yes|No|An array of a single AudioSource used to play a background music. In practice, only `music[0]` is used and allocated. This is more meant for longer playback than sounds as it doesn't support simultaneous playback, but is treated specially by the music playback system|
|musicids|int[]|Yes|No|The array of Musics id currently playing. In practice, there should always be one element in the array as only `musicids[0]` is used. If `musicids[0]` is -1, no music is currently playing|
|musiccoroutine|Coroutine|Yes|No|The current SwitchMusic coroutine call or null if no calls is in progress. It is also assigned to a ChangeMVolume coroutine call without being set to null once it's over, but that coroutine is not practically used under normal gameplay|
|musicvolume|float|Yes|No|Indicates the volume of all music playback in the settings between 0.0 (muted) and 1.0 (full volume). This is persisted to config.dat, but the default value before config.dat is loaded is hardcoded to 0.4|
|lastmusic|int|Yes|Yes|The last Musics id played using ChangeMusic which is used to determine the loop points to use for the playback|
|keepmusicafterbattle|bool|Yes|No|A boolean that indicates if the game is configured to resume the overworld music after a battle in the settings. false means the music after battle will RESTART, true means the music after battle will RESUME. This is persisted to config.dat, but the default value before config.dat is loaded is hardcoded to true|
|musicresume|float|Yes|No|If `keepmusicafterbattle` is true, this value represents the timestamp in seconds to set the next music playback when SwitchMusic is called. This is set by BattleControl's ReturnToOverworld and on the next time SwitchMusic is called, `music[0].time` will be set to this value followed by this value being reset to -1.0 (meaning no timestamp is saved). The default value on boot is hardcoded to -1.0. NOTE: This value can be incorrectly set by ReturnToOverworld in some circumstances, check this music playback issue's documentation for more details|
|musicnames|string[]|Yes|No|The contents of `Data/DialoguesX/MusicList` where `X` is the `languageid` split by LF|
|musicloop|float[][]|Yes|No|The contents of `Data/LoopPoints` where the first dimension is each music ordered by Musics id and the second dimensions are the loop points for the music. This is populated on the first LoopMusic call when this value is empty or null. Check the music loop points data documentation to learn more|
|msounds|AudioClip[]|Yes|Yes|This is only populated on consoles platform to the contents of `Audio/Music/X` where `X` is any name from the Musics enum. They are kept referenced in this field for caching purposes. On PC platforms, the value is left to null|
|samiramusics|List<int[]>|No|No|The list of Musics available when talking to Samira where each element contains 2 integers where the first is the Musics id and the second is the unlock status (-1 means not bought and 1 means bought). This is persisted to the save file|

## [PlayerControl](../PlayerControl/PlayerControl.md) and [player party](../General%20systems/Player%20party.md)

|Name|type|Is static?|Is Private?|Description|
|---:|-----|----------|------------|------------|
|player|PlayerControl|Yes|No|The only instance of PlayerControl that exists which represents the current player or null if a map is being loaded and the player is being recreated|
|playerdata|BattleData[]|No|No|The party's data in the overworld or the player party's data during a battle ordered by trueid (determined by the last ChangeParty call)|
|partyorder|int[]|No|No|The animid order of the overworld party after taking into account the switching. The default value is configured to be {0, 1, 2} from the from the Main scene using the Unity inspector|
|statbonus|List<int[]>|No|No|The list of stats bonuses applied on top of the base stats where each element has the following integers in each arrays: <ul><li>0: The bonus type (0 is HP, 1 is Attack, 2 is Defense, 3 is TP and 4 is MP)</li><li>1: The bonus amount</li><li>2: The bonus target (-1 is the party and any other number is the animid of whom this bonus applies to)</li></ul> This is persisted to the save file|
|tp|int|No|No|The current TP amount of the party. The default value is hardcoded to 10. This is persisted to the save file|
|maxtp|int|No|No|The current maximum TP amount of the party. The default value is hardcoded to 10. This value is calculated from its default value (10) added with the stats bonuses and medals effects|
|basetp|int|No|No|This is essentially the value of `maxtp`, but without taking into account the medals effects. The default value is hardcoded to 10. This is persisted to the save file|
|bp|int|No|No|The remaining amount of MP of the party. The default value is hardcoded to 5, but it should be noted that event 8 (creating a new file) will override this to 4 if flag 613 is true (RUIGEE is active). This is persisted to the save file|
|maxbp|int|No|No|The maximum amount of MP of the party. The default value is hardcoded to 5, but it should be noted that event 8 (creating a new file) will override this to 4 if flag 613 is true (RUIGEE is active). This is persisted to the save file|
|partylevel|int|No|No|The rank of the party. The default value is hardcoded to 1. This is persisted to the save file|
|partyexp|int|No|No|The amount of EXP accumulated for the current rank (this isn't a total amount cumulated in general as it restarts to count from 0 when ranking up) This is persisted to the save file|
|neededexp|int|No|No|The amount of EXP that `partyexp` needs to reach for the party to rank up. The default value is configured to be 100 from the from the Main scene using the Unity inspector. This is persisted to the save file|
|extrafollowers|List<int>|No|No|The animids of the followers that the player is requesting to have on each map load, see the follower system documentation for more details. Initially an empty list, but it is persisted to the save file|
|maxitems|int|No|No|The current maximum quantity of standard items allowed to be in the player's inventory. The default value is hardcoded to 10. This is persisted to the save file|
|maxstorage|int|No|No|The current maximum quantity of standard items allowed to be in the player's storage. The default value is hardcoded to 35. This is persisted to the save file|
|money|int|No|No|The berry count of the party on hand. This is persisted to the save file|
|skilldata|string[,]|Yes|No|The contents of `Data/SkillData` split by LF merged with the contents of `Data/DialoguesX/Skills` where `X` is the `languageid` split by LF. The first dimension corresponds to a line index in either TextAsset (which is a Skills id) and the second dimension is formated in a specific way which is detailed in the skills data TextAsset documentation|

## Inputs
TODO: document these after InputIO docs

|Name|type|Is static?|Is Private?|Description|
|---:|-----|----------|------------|------------|
|keyhold|float[]|Yes|Yes|A float array of 2 elements used in KeyHold and ResetKeyHold which implements the delayed autoshift (DAS) used in ItemList and prompts menu. The first element corresponds to a counting frametimer where the DAS kicks in once it reaches 15.0 frames and the second elements is a throttle where KeyHold won't return true for the next 7.0 frames after it has already returned true|
|joybinds|int[]|Yes|No|The array of bound joystick inputs ordered by Directions values. The format of each element is in the return of InputIO.GetJoystickRaw and it defaults to an array of 10 elements each being -55 which means no inputs is bound to this Directions. This is persisted to config.dat|
|joybuttons|Sprite[]|No|No|An array configured from the Main scene using the Unity inspector. It is expected to have at least 20 elements where the first 10 corresponds to the xbox joy glyph sprites ordered by Directions and the next 10 elements corresponds to the switch joy glyph sprites ordered by Directions|
|joybuttonsps|Sprite[]|No|No|An array configured from the Main scene using the Unity inspector. It is expected to have at least 4 elements where the elements corresponds to the playstation joy glyph sprites for the Directions Confirm, Cancel, Switch and Select respectively. NOTE: If `languageid` is set to 3 (`Japanese`), Confirm and Cancel are swapped so O means Confirm and X means Cancel while they would otherwise be reversed|
|joyid|int|Yes|No|The controller id configured in the settings TODO: document the values and detail stuff, the semantic is very weird and conflicting. This is persisted to config.dat|
|usejoystick|int|Yes|No|A number that indicates the configuration of the advanced particles in the settings. Here are the different values and their meaning:<ul><li>0: DISABLED</li><li>1: AUTO DETECT</li><li>2: FORCE XBOX</li><li>3: CHOOSE MANUALLY</li><li>4: PRE-CONFIGURED</li><li>5: CUSTOM BINDINGS</li></ul>This is persisted to config.dat, but the default value before loading config.dat is hardcoded to 1 (AUTO DETECT)|
|forcejoystick|int|Yes|No|TODO: what is happening with this value ??? persisted to config.dat, but it has a hardcoded default value of -1 before config.dat is loaded which means ???|
|analog|int|Yes|No|A number that indicates the configuration of the analog stick input sensitivity in the settings. 0 means OFF, 1 means LOW and 2 means FULL. This is persisted to config.dat, but the default value before loading config.dat is hardcoded to 2 (FULL)|
|joystick|bool|Yes|No|TODO: ???|
|stickholdx|bool|Yes|No|TODO: ???|
|stickholdy|bool|Yes|No|TODO: ???|
|forcecontrollerupdate|bool|Yes|No|TODO: ???|
|snapTo8|bool|Yes|No|A boolean that indicates if the game is configured to snap the travel direction of the Beemerang to the nearest of the 8 cardinal directions in the settings. false means OFF, true means ON. This is persisted to config.dat true, but the default value before config.dat is loaded is hardcoded to true (ON)|
|inputcooldown|float|No|No|The amount of frames that must ellapse before any ItemList, PauseMenu and SetText related inputs (including prompt, numberprompt or letterprompt) inputs are processed again. Maintained by LateUpdate|
|preconfigjoy|int[]|Yes|No|TODO: only the length is used in PauseMenu and related to PauseMenu's joystickid ??? The value is hardcoded to {0, 1, 2, 3, 4, 5, 6, 7, 8}|
|precjoystring|int[]|Yes|No|The `menutext` indexes of all the pre configured controller's names in the settings ordered by TODO: ??? The value is hardcoded to {225, 226, 227, 232, 233, 228, 229, 242, 269}|


## Library

|Name|type|Is static?|Is Private?|Description|
|---:|-----|----------|------------|------------|
|librarystuff|bool[,]|No|No|A jagged array that tracks all library's collection flag grouped by their type on the first dimension:<ul><li>0: [Discovery](../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md)</li><li>1: [Bestiary](../Enums%20and%20IDs/librarystuff/Bestiary%20entry.md)</li><li>2: [Recipe](../Enums%20and%20IDs/librarystuff/Recipes%20entry.md)</li><li>3: [Record](../Enums%20and%20IDs/librarystuff/Records%20entry.md)</li><li>4: [Area](../Enums%20and%20IDs/librarystuff/Areas.md)</li></ul>The lengths of the array are hardcoded to 5 (the library types) and 256 (overprovisions each type, the actual amount for each type is in `librarylimit`). The entire jagged array is persisted to the save file|
|librarylimit|int[]|Yes|No|A hardcoded list of the amount of each elements of `librarystuff` grouped by type:<ul><li>0: Discovery</li><li>1: Bestiary</li><li>2: Recipe</li><li>3: Record</li><li>4: Area</li></ul> The list is hardcoded to {50, 92, 70, 30, 100}|
|librarydata|string[,,]|Yes|No|All the `librarystuff` ids grouped by their type on the first dimension, the second dimension corresponds to a `librarystuff` id and the third dimension depends on the type:<ul><li>0: Discovery (from `Data/DialoguesX/Discoveries` where `X` is the `languageid`)</li><li>1: Bestiary (from `Data/DialoguesX/EnemyTattle` where `X` is the `languageid`)</li><li>2: Recipe (from `Data/DialoguesX/CookLibrary` where `X` is the `languageid`)</li><li>3: Record (from `Data/DialoguesX/Synopsis` where `X` is the `languageid`)</li><li>4: Area (from `Data/DialoguesX/Synopsis` where `X` is the `languageid`. NOTE: This is incorrect, but it doesn't matter because the game never reads this data for areas under normal gameplay and it instead uses `areanames` which contains the areas's names)</li></ul>|
|recipedata|int[,]|Yes|No|The contents of `Data/RecipeData` where each line corresponds to a recipe on the first dimension. The second dimension features a reformated version of the data whose arrangement depends on if it's a single or double items recipe:<ul><li>Single item: index 0 is the input item id, index 1 is -1 and index 2 is the result item id</li><li>Double item: index 0 is the numerically smaller input item id, index 1 is the numerically larger input item id and index 2 is the result item id</li></ul>|
|libraryorder|int[,]|Yes|No|All the `librarystuff` ids grouped by their type on the first dimension and ordered by their respective rendering order:<ul><li>0: Discovery (from `Data/DiscoveryOrder`)</li><li>1: Bestiary (from `Data/TattleList`)</li><li>2: Recipe (from `Data/CookOrder`)</li><li>3: Record (from `Data/SynopsisOrder`)</li><li>4: Area (always empty since there's no order on this one)</li></ul>|
|achiveicons|int[]|Yes|No|All the sprite indexes of each record located in `Sprites/Items/EnemyPortraits`. The order and data comes from `Data/SynopsisOrder` (same order and the data is only the sprite index field of each lines)|
|discoveryicons|int[]|Yes|No|All the sprite indexes of each discoveries located in `Sprites/Items/EnemyPortraits`. The order and data comes from `Data/DiscoveryOrder` (same order and the data is only the sprite index field of each lines)|
|librarysprites|Sprite[]|Yes|No|The sprites in `Sprites/Items/EnemyPortraits`|
|enemyencounter|int[,]|No|No|Represents the amount of times each enemies was seen and defeated in the bestiary grouped by enemies id in the first dimension, but overprovisioned to 256 elements that each have 2 elements where the first represents the amount seen and the second represents the amount defeated. The entire jagged array is persisted to the save file|

## [Quest board](../General%20systems/Quest%20boards.md)

|Name|type|Is static?|Is Private?|Description|
|---:|-----|----------|------------|------------|
|boardquests|List<int>[]|No|No|All the BoardQuests grouped by their state:<ul><li>0: Open</li><li>1: Taken</li><li>2: Done</li></ul> This is persisted to the save file|
|questboardobj|Transform|No|No|The current `QuestBoard` UI GameObject rendered by OpenQuestBoard or null if no quest board UI is rendering|
|boardcaller|NPCControl|No|No|The last caller NPCControl used during OpenQuestBoard which happens during the QuestBoard Interaction where the caller is the NPCControl that was interacted with|
|boardquestdata|string[,]|Yes|No|The contents of `Data/BoardData` split by LF merged with the contents of `Data/DialoguesX/BoardQuests` where `X` is the `languageid` split by LF. The first dimension corresponds to a line index in either TextAsset (which is a BoardQuests id) and the second dimension is formated in a specific way which is detailed in the BoardQuests data TextAsset documentation|
|questchecks|int[][]|Yes|Yes|The contents of `Data/QuestChecks` where each line corresponds to a BoardQuests on the first dimension and the second dimensions corresponds to the field index of each line. Essentially, the data is loaded as is in this multidimensional array without formatting other than converting the strings to integers|
|bountyquests|int[]|Yes|Yes|The BoardQuests ids of the bounty quests. This is used in LateUpdate to ensure that any that were present in `boardquests[2]` (done quests) are removed from the `boardquests[0]` (open) and `boardquests[1]` (taken) arrays. The value is hardcoded to {9, 23, 8, 21, 10}|
|questcolors|Color[]|No|No|An array configured from the Main scene using the Unity inspector. It corresponds to the colors used in the BoardQuests's UI to visually differentiate between the tabs (each tab corresponds to one of the 3 BoardQuests state). It is normally configured to be:<ul><li>0: CA8E22 (mostly brown, used for open quests)</li><li>1: FBF511 (mostly yellow, used for taken quests)</li><li>2: 98E505 (mostly green, used for done quests)</li></ul>|

## [BattleControl](../Battle%20system/BattleControl.md) and CardGame

|Name|type|Is static?|Is Private?|Description|
|---:|-----|----------|------------|------------|
|battle|BattleControl|Yes|No|The only instance of BattleControl that exists which represents the current battle if one exists, null if no battle is in progress|
|inbattle|bool|No|No|Indicate whether or not a battle is in progress. This is similar to `battle` not being null, but this value is set to true much later during BattleControl's StartBattle (right before enemy first strike logic if applicable). The value is reset to false between retries, when reloading the save or when ReturnToOverworld is called|
|battleresult|bool|Yes|No|Tells if the last battle, CardGame, FlappyBee or MazeGame ended in a win (the value is true) or a loss (the value is false). A battle is assumed to be won by StartBattle which sets this value to true until this assumption is proven otherwise when DeadParty is called while `battlelossevent` was true or TryFlee ended in a sucessfull flee. For FlappyBee, the value is always set to true when it ends|
|lastdefeated|List<int>|No|No|The list of enemy ids that were defeated during the course of the last battle. The list is cleared on StartBattle and on the Death of an Enemy NPCControl|
|lastitemuser|int|Yes|No|The `trueid` of the player party member that last used an item during UseItem. This is used by DoItemEffect to determine the user of the item to see if the `HealPlus` medal applies|
|battlefled|bool|Yes|No|Indicate whether or not the last battle was exited as a result of a successful TryFlee. This is set by BattleControl's ReturnToOverworld|
|battleenemyfled|bool|Yes|No|Indicate whether or not the last battle resulted in at least one enemy fled while no EXP was rewarded which typically happens when all enemies fled. This is always reset to false on StartBattle. This is only used in event 40 (theater event)|
|battlelossevent|bool|Yes|No|If set to true and BattleControl's DeadParty is called, GameOver won't be called and instead, ReturnToOverworld will be called after setting `battleresult` to false. It is always reset to false on StartEvent and EndEvent so it is meant to be set to true during events to not game over upon loosing a battle|
|haltbattleload|bool|Yes|No|If set to true, it will stall StartBattle in the middle to let the event further configure the battle as it is starting. This value isn't automatically reset which means that the event using it must reset it to false after usage|
|battlecampos|Vector3|Yes|No|A hardcoded value of `camoffset` to set during a battle when the camera is set to its default battle state. The value is hardcoded to (0.0, 3.0, -9.25)|
|battlecamangle|Vector3|Yes|No|A hardcoded value of `camangleoffset` to set during a battle when the camera is set to its default battle state. The value is hardcoded to (5.0, 0.0, 0.0)|
|enemydata|string[,]|Yes|No|The contents of `Data/EnemyData` where each line index corresponds to an enemy id on the first dimension and the second dimensions corresponds to the field index of each line. Essentially, the data is loaded as is in this jagged array without formatting. For details on the TextAsset, consult the EnemyData documentation|
|enemynames|string[]|Yes|No|The contents of `Data/DialoguesX/EnemyTattle` where `X` is the `languageid` split by LF|
|commandhelptext|string[]|Yes|No|The contents of `Data/DialoguesX/ActionCommands` where `X` is the `languageid` split by LF|
|conditionsprites|Sprite[]|No|No|An array configured from the Main scene using the Unity inspector. It is expected to have at least 22 elements where the first element corresponds to the sprite used to render a number on it about the amount of actor turns (or `moreturnnextturn`) available in battle and the other 21 elements corresponds to a BattleCondition sprite icon to render when the condition is inflicted (oredered by BattleCondtion id). This array is used by EntityControl's UpdateConditionBubbles. NOTE: The `Shield`, `Sturdy`, `Eaten` and `EventStop` conditions are expected to have a sprite, but will never be rendered by the game under normal gameplay|
|entitytouchevent|int|No|No|If set to a valid event id, any overworld battle encounter that would have happened as a result of NPCControl's StartBattle won't happen and instead, an event will be started with the id being this value. In practice, this is only used in event 102 (rescuing the Overseer at the Honey Factory). The default value is configured to be -1 from the from the Main scene using the Unity inspector. This value is not automatically reset and a manual set to -1 must be done after usage|
|firstbattleaction|bool|No|No|Indicates whether or not the dummy DoAction call has happened throughout the game's session. Check the documentation on DoAction's performance considerations to learn more|
|cardgame|CardGame|No|No|The only instance of CardGame that exists which represents the current CardGame in progress or null if no CardGame is in progress|

## Language

|Name|type|Is static?|Is Private?|Description|
|---:|-----|----------|------------|------------|
|languageid|int|Yes|No|The current language id or -1 if no language is set yet. This is persisted to config.dat. The default value is -1 before the booting process determines its value from config.dat if it exists|
|languagenames|readonly string[]|Yes|No|A hardcoded list of the names of the supported languages in the game ordered by languageid for use in the languages list listtype. Value is harcoded to { "English (US)", "Español (LA)", "Português (BR)", "日本語 (Japanese)", "Deutsch (German)", "한국어 (Korean)", "Русский (Russian)" }|
|languagehelp|string[]|Yes|No|The contents of `Data/LanguageHelp` TextAsset split by LF|

## Flags arrays

|Name|type|Is static?|Is Private?|Description|
|---:|-----|----------|------------|------------|
|flags|bool[]|No|No|The [flags](../Flags%20arrays/flags.md) array which are global booleans used for various purposes. The array always has a length of 750 and the array is persisted to the save file|
|flagvar|int[]|No|No|The [flagvar](../Flags%20arrays/flagvar.md) array which are global integers used for various purposes. The array always has a length of 70 and the array is persisted to the save file|
|flagstring|string[]|No|No|The [flagstring](../Flags%20arrays/flagstring.md) array which are global strings used for various purposes. The array always has a length of 15 and the array is persisted to the save file|
|regionalflags|bool[]|No|No|The [regional flags](../Flags%20arrays/Regionalflags.md) array which are booleans scoped to the current area used for various purposes. The array is always reset on a UpdateArea call which only happens on MapControl's Start when the `areaid` changed. The array always has a length of 100 and the array is persisted to the save file|
|crystalbflags|bool[]|No|No|A flags array which are global booleans used specifically to track each [crystal berries](../Enums%20and%20IDs/crystalbfflags.md)'s collection (each booleans tells if the crystal berry was collected or not). The array always has a length of 50 and the array is persisted to the save file|

## Runtime state

|Name|type|Is static?|Is Private?|Description|
|---:|-----|----------|------------|------------|
|instance|MainManager|Yes|No|The MainManager singleton's instance|
|saveslot|int|Yes|No|The save slot index that was loaded or created which is used to determine the filename for the current save file|
|framestep|float|Yes|No|The value of TieFramerate(1.0) which is itself Time.smoothDeltaTime * 60.0. This is the main way the game scales time which can be thought of as the fraction of time that has passed on the last frame relative to the expected time assuming 60 FPS (meaning a value of 1.0 is equivalent to 1/60th of a second that ellapsed on the last frames on average, anything lower means the frame took less time than expected and anything higher means the frame took more time than expected). This is updated on every LateUpdate cycle which means the value lags behind during earlier Unity events in the frame (so an Update cycle will have an outdated value)|
|pause|bool|No|No|Indicate whether or not the game is in a complete pause which typically happens when the pause menu is present. There is also a period of time during BattleControl's StartBattle where a pause is entered temporarilly|
|minipause|bool|No|No|This value is similar to `pause`, but it set progammatically in a wide variety of situations. It doesn't have all the effects of `pause`, but it features most of them so it allows the game to be in a paused state without needing a complete pause menu|
|inevent|bool|No|No|Indicate whether or not an event is in progress. It also indicates if a CardGame is in progress|
|basicload|bool|Yes|No|Indicates whether or not LoadEverything completed|
|roomtransition|bool|Yes|No|A value that indicate whether or not a [TransferMap](../MapControl/Map%20loading.md#transfermap) call or a save SetText command processing is in progress. This is used by [DoClock](DoClock.md) to see if it is save to call Ressources.UnloadUnusedAssets where if the value is true, the call won't happen|
|started|bool|No|No|This indicates whether or not DetectIgnoreSphere was called on the `player.entity` which happens on the first LateUpdate cycle where the player isn't null. This value ensures it happens at most once which is necessary after a save file load because this normally happens on TransferMap, but loading a save file won't load the map with that method|
|globalcooldown|float|No|No|The amount of frames that must ellapses before PlaySound or any SoundControl are allowed to process again. This is only used by TransferMap and BattleControl's ReturnToOverworld where the value is set to 30.0 frames|
|clocksec|int|No|No|The seconds component of the in game timer representing the playtime. The timer is maintained in DoClock. This is persisted to the save file|
|clockmin|int|No|No|The minutes component of the in game timer representing the playtime. The timer is maintained in DoClock. This is persisted to the save file|
|clockhour|int|No|No|The hours component of the in game timer representing the playtime. The timer is maintained in DoClock. This is persisted to the save file|
|halt|bool|Yes|No|This is specifically used in event 1 to yield all frames until StopHalt is called by an Animator the event uses|

## [Events](../Enums%20and%20IDs/Events.md)

|Name|type|Is static?|Is Private?|Description|
|---:|-----|----------|------------|------------|
|events|EventControl|Yes|No|The only instance of EventControl that exists used to start events|
|eventtoss|int|Yes|No|Used to store the `data[1]` value of the caller of a SetText call that processes an [additemtoss](../SetText/Individual%20commands/Additemtoss.md) command which corresponds to an event id of an event to start at the end of the SetText call. The default value is hardcoded to -1 which means no event id is assigned. The value is reset to -1 once the event is started|
|lastevent|int|Yes|No|The event id of the current event or -1 if no event is playing. It is set on StartEvent and reset to -1 on EndEvent, but the default value on boot is 0|


## [MapControl](../MapControl/MapControl.md)

|Name|type|Is static?|Is Private?|Description|
|---:|-----|----------|------------|------------|
|map|MapControl|Yes|No|The only instance of MapControl that represents the current map or null if it was destroyed during the loading of a map|
|areaid|int|No|No|The current Areas which is determined by the current `map`. This is persisted to the save file|
|insideid|int|No|No|The current inside's id which are defined by the current `map` or -1 if the player isn't in any inside. The default value is hardcoded to be -1|
|battlestage|int|No|No|The `map`'s `battlemap` value (it's assigned on the MapControl's Start). This is used to determine the default stage to use on StartBattle when stageid is -1|
|inmusicrange|int|No|No|The map entity id of the MusicRange that currently has control over the music playback or -1 if no MusicRange is taking control of the music playback. The default value is configured to be -1 from the from the Main scene using the Unity inspector and it is automatically reset to -1 on a TransferMap call|
|areanames|string[]|Yes|No|The contents of `Data/DialoguesX/AreaNames` where `X` is the `languageid` split by LF|

## Reusable particles

|Name|type|Is static?|Is Private?|Description|
|---:|-----|----------|------------|------------|
|hitpart|ParticleSystem|Yes|No|An instance of `Prefabs/Particles/HitPart` initially offscreen, but will show up and play when HitPart is called with the specified position|
|deathpart|ParticleSystem|Yes|No|An instance of `Prefabs/Particles/deathsmoke` initially offscreen, but will show up and play when DeathSmoke is called with the specified position, size and render|

## Coroutine tracking

|Name|type|Is static?|Is Private?|Description|
|---:|-----|----------|------------|------------|
|itempicked|bool|No|No|Indicate whether or not there's a [CheckItem](../Entities/NPCControl/ObjectTypes/Item.md#checkitem) call in progress which controls some logic related to [JumpSpring](../Entities/NPCControl/ObjectTypes/JumpSpring.md) objects and [Shop](../Entities/NPCControl/Interaction/Shop.md) / [CaravanBadge](../Entities/NPCControl/Interaction/CaravanBadge.md) interactions|
|templetter|Coroutine|Yes|No|The TempColor or TempLetters coroutine call in progress or null if neither have a call in progress. TempColor is only used by CardGame and TempLetters is UNUSED|
|chaptername|Coroutine|Yes|No|The ChapterName, LoadEssentials or InnSleep coroutine call in progress or null if none of them have a call in progress|

## Entities

|Name|type|Is static?|Is Private?|Description|
|---:|-----|----------|------------|------------|
|entityactive|Vector3|Yes|No|A hardcoded value that determines the bounds an entity or Transform's WorldToViewportPoint of the `MainCamera` can be in so that InCameraRange returns true (for an entity, this is their `campos`). The allowed range is defined as being between -`X` and `X` where `X` is the value of this vector. The value is hardcoded to (0.6, 1.25, -1.0)|
|overridefollower|bool|No|No|If this value is true, it will prevent the entity follow logic from happening meaning they won't have their standard follow movement which gives more control for the game to move entities manually|
|endata|Entity_Data[]|Yes|No|The contents of `Data/EntityValues` ordered by animid (enum format). See the [endata](../TextAsset%20Data/Entity%20data.md) documentation to learn more|

## Other settings

|Name|type|Is static?|Is Private?|Description|
|---:|-----|----------|------------|------------|
|secretunlocks|bool[]|Yes|No|An array that tells if each secret menu codes was entered at least once which is used in event 8 (creating a new file) to determine what to show in the secret menu. This is persisted to config.dat, but it defaults to be an empty array of 5 elements|
|resolution|Vector2[]|Yes|No|A hardcoded array that represents the different display resolutions available in the settings. The x component is the horizontal pixel count and the y component is the vertical pixel count. Each component is in float, but they are meant to be converted to int to get the actual resolution it represents. Here is the hardcoded array: { (1024.0, 576.0), (1152.0, 648.0), (1280.0, 720.0), (1366.0, 768.0), (1600.0, 900.0), (1920.0, 1080.0), (2560.0, 1440.0), (3840.0, 2160.0) }|
|resolutionindex|int|Yes|No|The index of `resolution` that represents the current display resolution configured. This is persisted to config.dat|
|fps|int|Yes|No|A number that indicates the Application.targetFrameRate configured in the settings. Here are the meanings of the values:<ul><li>0: Application.targetFrameRate of 30</li><li>1: Application.targetFrameRate of 60</li><li>2: Application.targetFrameRate of -1 (unlimited, but it is not possible to configure this value under normal gameplay)</li></ul>This is persisted to config.dat. The default value before loading config.dat is hardcoded to 1 (60 FPS)|
|vsync|int|Yes|No|A number that indicates if vsync is enabled or not in the settings. 0 means OFF, 1 means ON. This is persisted to config.dat|
|enableoutline|int|Yes|No|A number that indicates the configuration of the 3D outlines in the settings. 0 means OFF, 1 means LOW and 2 means FULL. This is persisted to config.dat, but the default value before loading config.dat is hardcoded to 2 (FULL)|
|particlelevel|int|Yes|No|A number that indicates the configuration of the advanced particles in the settings. 0 means OFF, 1 means LOW and 2 means FULL. This is persisted to config.dat, but the default value before loading config.dat is hardcoded to 2 (FULL)|
|fullscreen|bool|Yes|No|A boolean that indicates if the game is configured to render fullscreen in the settings. false means OFF, true means ON. This is persisted to config.dat|
|lowshadows|bool|Yes|No|A boolean that indicates if the shadows are rendered with low quality in the settings. false means a shadow quality of FULL (QualitySettings.shadowResolution is set to High), true means a shadow quality of LOW (QualitySettings.shadowResolution is set to Low). This is persisted to config.dat|
|lowtexture|bool|Yes|No|A boolean that indicates if the textures are rendered with low quality in the settings. false means a texture quality of FULL (QualitySettings.masterTextureLimit is set to 0 which renders the texture as is), true means a texture quality of LOW (QualitySettings.masterTextureLimit is set to 1 which decreases by half the sizes of all textures). This is persisted to config.dat|
|nowindeffect|bool|Yes|No|A boolean that indicates if the wind effects aren't rendered in the settings (meaning the Wind component won't do anything and the material of BeetleGrass wont be set to `windshader`). false means the effects are ON, true means the effects are OFF. This is persisted to config.dat|
|mashcommandalt|bool|Yes|No|A boolean that indicates if the game is configured to replace RandomTappingBar and TappingKey actions commands by SequentialKeys in the settings. false means FILL BAR, true means SEQUENTIAL KEYS. This is persisted to config.dat|
|monoaudio|bool|Yes|No|A boolean that indicates if the game is configured to have mono audio in the settings. false means STEREO (AudioSettings.speakerMode is set to Stereo), true means MONO (AudioSettings.speakerMode is set to Mono). This is persisted to config.dat|
|pauseonfocus|bool|Yes|No|A boolean that indicates if the game is configured to pause when the game window isn't in focus in the settings. false means OFF (Application.runInBackground is set to true), true means ON (Application.runInBackground is set to false). This is persisted to config.dat|

## Other cached assets

|Name|type|Is static?|Is Private?|Description|
|---:|-----|----------|------------|------------|
|projectilepsrites|Sprite[]|No|No|The sprites in `Sprites/Misc/projectiles`|
|shadowsprite|Sprite|Yes|No|The sprites in `Sprites/Misc/shadow`|
|fakelight|Shader|Yes|No|The `Materials/FakeLight` material's shader|
|fonts|Font[]|Yes|No|All the Font assets loaded from `Fonts/X` where `X` is the name of the font where the name and the order in the array matches the fonttype|
|fontmat|Material[]|Yes|No|All the Font materials assets loaded from `Fonts/X` where `X` is the name of the font where the name and the order in the array matches the fonttype|
|spritemat|Material|Yes|No|The `Materials/SpriteMat` material|
|holosprite|Material|Yes|No|The `Materials/SpriteHologram` material|
|spritematlit|Material|Yes|No|The `Materials/SpriteLit` material|
|spritedefaultunity|Material|Yes|No|The `Materials/SpriteDefault` material|
|grayscale|Material|Yes|No|The `Materials/Grayscale` material|
|outlinemain|Material|Yes|No|The `Materials/OutlineMain` material|
|Main3D|Material|Yes|No|The `Materials/3DMain` material|
|Fade3D|Material|Yes|No|The `Materials/3DFade` material|
|mainPlane|Material|Yes|No|The `Materials/MainPlane` material|
|fadePlane|Material|Yes|No|The `Materials/FadePlane` material|
|windShader|Material|Yes|No|The `Materials/WindShader` material|
|emptymat|Material|Yes|No|The `Materials/Empty` material|
|defaultpmat|PhysicMaterial|Yes|No|The `Materials/DefaultPhysis` physics material|
|grasssprite|Sprite[]|Yes|No|The sprites in `Sprites/Objects/grass`|
|parttex|Texture[]|Yes|Yes|The contents of `Sprites/Particles` (the textures) which are kept referenced in this field for caching purposes|
|spritepart|Sprite[]|Yes|Yes|The contents of `Sprites/Particles` (the sprites) which are kept referenced in this field for caching purposes|
|partfab|GameObject[]|Yes|Yes|The contents of `Prefabs/Particles` which are kept referenced in this field for caching purposes|
