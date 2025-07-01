# Disk files handling
InputIO also handles interactions with the 2 kinds disk files it produces:

- The [save files](../MainManager/Methods/Save%20file.md)
- The [config file](../External%20data%20format/Config%20File.md)

Their format are documented in their respective page. This page explains more how InputIO deals with these files in general.

## General file utilities
The following methods handles any kinds of disk files:

```cs
public static bool CreateFile(string path, string content)
```
Creates or opens a file at `path`, write `content` into it, close it and returns true to indicate sucess.

The method returns false if any exception is thrown during this process.

```cs
public static string ReadFile(string path)
```
Returns File.ReadAllText(`path`) unless the file doesn't exist where null is returned instead.

```cs
public static bool DeleteFile(string path)
```
Deletes the file at `path` and returns true if it was sucessful.

The method returns false under any of the following conditions:

- The file doesn't exist
- Any exception was thrown when deleting the file

## Save files
There are several methods InputIO has to handle the save files.

The main one is Save:

```cs
public static bool Save(Vector3? savepos)
```
It writes the current game state to the current save file using `savepos` as the position of the party to save (if it's null, the current `player` position is used instead).

The logic to write to the save file is complex, here are the details:

- Obtain the [Save file](../MainManager/Methods/Save%20file.md) string data by calling MainManager.SaveFile(`savepos`)
- Creates or opens the `saveXt.dat` file where `X` is MainManager.`saveslot`. This is a temporary file that will be renamed later
- Call Encrypt on the SaveFile string obtained earlier to encode it (check below for details on that method)
- Writes the encoded version of the string to the opened file
- Closes the file
- If the `saveX.dat` file exists where `X` is MainManager.`saveslot` (meaning the current save slot has had a backing file):
    - If the `saveXbackup.dat` file exists where `X` is MainManager.`saveslot`, it is deleted
    - The existing `saveX.dat` file is renamed to `saveXbackup.dat` which moves the existing save file to its backup one for redundancy purposes
- The `saveXt.dat` file that was just created is renamed to remove the `t` which fully commits the save file to disk
- True is returned to indicate success

If any exception is thrown at any point in this process, false is returned alongside a Debug log saying the exception's Message prefixed with `save exception: `.

The idea with this complex logic is to have a redundancy mechanism in which the previous version of a save is always available whose filename is `saveXbackup.dat` where `X` is the matching save file slot. The file isn't used by the game, but it is persisted in case the main file gets corrupted which would allow the player to manually rename the file to restore its previous version.

This is also why there's a temprory file involved with the `t` suffix: it allows this switch operation to be atomic since the game won't check these files so if something goes wrong, the main file won't get corrupted.

```cs
public static string Encrypt(string input)
```
A part of Save. Returns a string where every character in `input` was XORed with the int contained in the `Data/key` TextAsset. This is used to encode or decode a [save file](../MainManager/Methods/Save%20file.md) before reading it or writting it.

Finally, there's a utility to know if a save slot is backed by a file:

```cs
public static bool SaveExists(int id)
```
Returns whether or not the `saveX.dat` [save file](../MainManager/Methods/Save%20file.md) exists where `X` is `id`.

## Config file
The only InputIO method related to the config file is LoadSettings:

```cs
public static bool LoadSettings(bool overwrite)
```
It either reads [config.dat](../External%20data%20format/Config%20File.md) if it exists and `overwrite` is false followed by applying the settings read or create a new config file if it doesn't exist or `overwrite` is true and saves the current field settings on it. In either cases, true is returned to indicate success.

Specifically, reading an existing file involves:

- MainManager.ReadSettings(File.ReadAllLines(`config.dat`))
- MainManager.ApplySettings()

Both method are documented in the [config file](../External%20data%20format/Config%20File.md) documentation. If any exception is thrown during this process, LoadSettings is recalled with an `overwrite` of true to force the recreation of the file so the game settings and the file are ensured to be in sync. If no exceptions occured, true is returned to indicate success.

In the case of creating a new `config.dat`, it involves the following:

- MainManager.SaveSettings is called to get the text to write to the config file
- File.CreateText(`config.dat`) called
- The string obtained earlier is written to the file
- The file is closed

If any exception is thrown during this process, a debug log of `config creation failed` happens and false is returned.

If no exception happened and `overwrite` was false (meaning an attempt to open the file resulted in its creation as it didn't exist), the resolution of the game is set to 1024 x 576 windowed.
