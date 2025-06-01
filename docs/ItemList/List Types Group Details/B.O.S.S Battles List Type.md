# B.O.S.S Battles list type

Display the list of available mini bosses or bosses to battle using the B.O.S.S system.

## Options generation

`listvar` is the return of the GetBosses method:

```cs
public static int[] GetBosses()
```
It expects `multilist` to be filled beforehand with enemy ids (-1 for Zasp and Mothiva fight, -2 for Team Snakemouth fight). How this works is for everything that isn't -1 or -2, it is added to `listvar` if it the corresponding enemy has been seen at least once. If it is -1, it will only add it if [flags](../../Flags%20arrays/flags.md) 118 is true (beaten Zasp & Mothiva at Golden Hills). If it is -2, it will only add it if [flags](../../Flags%20arrays/flags.md) 555 is true (in postgame).

## Option's [SetText](../../SetText/SetText.md) input string

The text is the enemy name from `enemynames` of the option unless it is one of the following where it will pick a line from the current map dialogue:

* -2: line 87 (Team Snakemouth)
* -1: line 13 (Zasp and Mothiva)
* 51: line 74 (Kali and Kabbu)
* 85: line 75 (Cenn and Pisci)
* 92: line 76 (Team Maki)
* 74: line 77 (Cross and Poi)

After the text is determined, [flagvar](../../Flags%20arrays/flagvar.md) 6 is set to `option`.

The x position of the text is overridden to -2.65.

## Description box rendering

It uses the default rendering scheme described in [Description box rendering](../ShowItemList%20Life%20Cycle/Description%20box%20rendering.md) where the text is empty. It should be noted that under normal gameplay, this list type is not called with `showdescription` set to true.

## Confirmation handling

Confirmation is handled by MainManager's Update. First, the [flagvar](../../Flags%20arrays/flagvar.md) of the `storeid` is set to the selected option (bug?). Then, the [ItemList](../ItemList.md) gets destroyed which ends its processing and resets the [ItemList State Machine](../ItemList%20State%20Machine.md).
