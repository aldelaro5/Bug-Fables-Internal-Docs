# LibraryShelf
This component is specific to the Lore Book functionality of a shelf in the `AntPalaceLibrary` map.

It exposes a method called Refresh that is called on Start and by the librarybook SetText command which will destroy every existing books and recreate them. The amount of books created is given by flagvar 15 (amount of Lore Books given). The GameObject representing the books is in the `book` field which is configured via the inspector.
