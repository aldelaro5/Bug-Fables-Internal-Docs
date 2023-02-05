# Recipes data

[Recipes entry](../Enums%20and%20IDs/librarystuff/Recipes%20entry.md) data are split in 3 TextAssets in the game:

* Loaded on boot by SetVariables: 
  * `Ressources/data/CookLibrary`
  * `Ressources/data/CookOrder`
* Loaded on boot by LoadEssentials:
  * `Ressources/data/RecipeData`

## `CookLibrary` data

The asset contains one line per [Recipes entry](../Enums%20and%20IDs/librarystuff/Recipes%20entry.md) whose id corresponds to the line index. Each line contains fields separated by `@`, but only one is defined:
Name | Type | Description
----- | ------ | -------
Recipe shown | [Items](../Enums%20and%20IDs/Items.md) ids list separated by `,` (or -1 for incompatible ingredients) | The [Items](../Enums%20and%20IDs/Items.md) that constitute the recipe to show as description and in the details page

The data will be loaded into `librarydata[2, id, 0]` where `id` is the [Discoveries entry](../Enums%20and%20IDs/librarystuff/Discoveries%20entry.md) id. There may be empty second fields, but since only the first one is defined, it should be considered that `librarydata[2, id, 1]` won't be defined either (it will be an empty string most of the time).

The game only supports a single recipe to be shown as the description and in the details page. It will be rendered using the corresponding [Items](../Enums%20and%20IDs/Items.md)'s names. The exception is if the value is -1 which means any incompatible recipes which will use menutext 150 for the details page and menutext 59 for the description underneath the icon.

This is not the same then the recipe in `RecipeData` which means it is possible to specify an invalid or incorrect recipes here.

## `CookOrder` data

The asset contains one line per [Recipes entry](../Enums%20and%20IDs/librarystuff/Recipes%20entry.md) whose id corresponds to the line index. Each line contains one field
Name | Type | Description
----- | ------ | -------
Result item shown | [Items](../Enums%20and%20IDs/Items.md) id | The resulting [Items](../Enums%20and%20IDs/Items.md) that will be shown in the library page

The data will be loaded into `libraryorder[2, id]`  where `id` is the [Recipes entry](../Enums%20and%20IDs/librarystuff/Recipes%20entry.md) id.

This field is used to generate the icon which is done by using the corresponding the [Items](../Enums%20and%20IDs/Items.md) sprites childed to an instance of sprite 10 of `Sprites/Items/EnemyPortraits` (which is a blank sprite). It is also used to generate the text of the details page by using the [Items](../Enums%20and%20IDs/Items.md)'s description. It's also used with the [Library List Type](../ItemList/List%20Types%20Group%20Details/Library%20List%20Type.md) to determine the name of the [Items](../Enums%20and%20IDs/Items.md). 

This is not the same than the resulting [Items](../Enums%20and%20IDs/Items.md) present in `RecipeData` which means it is possible to specify the incorrect [Items](../Enums%20and%20IDs/Items.md) for the corresponding recipe used in `CookLibrary`.

## `RecipesData` data

The asset contains one line per valid recipes. It is not the same than a [Recipes entry](../Enums%20and%20IDs/librarystuff/Recipes%20entry.md) as each of them can contain multiple valid recipes. Each line contains fields separated by `,`:
Name | Type | Description
----- | ------ | -------
First item | [Items](../Enums%20and%20IDs/Items.md) id | The first [Items](../Enums%20and%20IDs/Items.md) of the combination
Second item | [Items](../Enums%20and%20IDs/Items.md) id (-1 if not present) | The second [Items](../Enums%20and%20IDs/Items.md) of the combination if it exists
Result item | [Items](../Enums%20and%20IDs/Items.md) id | The resulting [Items](../Enums%20and%20IDs/Items.md) of the combination

The data will be loaded into `recipedata[i, x]` where i is the line index and x depends on the field and their value:

* First item: x is 0 if second item is > -1 AND the first item is \<= than the second item. Otherwise, x is 1.
* Second item: x is 1 if second item is > -1 AND the first item is \<= than the second item. Otherwise, x is 0.
* Result item: x is always 2.
  This means that the first 2 elements of the array for a given recipe are the [Items](../Enums%20and%20IDs/Items.md) ordered by their id and the third element is the result item.

Any combination of [Items](../Enums%20and%20IDs/Items.md) not listed in this file will produce [Items](../Enums%20and%20IDs/Items.md) 8 (Mistake).
