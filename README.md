# Bug-Fables-Internal-Docs
A repository to aggregate documentation about Bug Fables's inner working

This repos contains all the documentation I have accumulated throughout the years as well as new ones that will be pushed here as I learn more about the game. The purpose of this repos is to have a centralised place to consult my docs. It also prevent a situation where these files are lost  and it provides a source that doesn't involve the wiki or the Discord. I will seek plans to present the info better such as GitHub pages in the future, but for now, I just put the raw information in markdown which will be refined as I continue working on the game's tooling. 

The reason information shouldn't be from the wiki or the discord is because the former has proven to not be as reliable and it is difficult to consult or edit while the later is complex to lookup. Informations about the internals are going to be ESSENTIAL for anyone looking to mod anything and for myself in my current project so they MUST be provided in a central place that everyone has access to. This repos also has a [wiki](https://github.com/aldelaro5/Bug-Fables-Internal-Docs/wiki) which will be more focussed on important procedures such as extracting assets, decompiling and debugging the game.

This repos will get updates as more information is aggregated and figured out so it isn't lost in time. I may accept pull requests or issues (in case of typos or innacurate informations), but everything needs to provide a way for me to validate the infoormation.

For the Obsidian notebook version of the docs, check the [release](https://github.com/aldelaro5/Bug-Fables-Internal-Docs/releases) section of this repos.

## What is allowed in this repos and what isn't
This repos will ONLY contain what I like to call "metadata" informations. This is what is contained and allowed in this repos:

- Documentations on any data file (its format and what each lines or fields does)
- List of enum values
- High level documentation on a subsystem of the game or as a whole. This means sentences in mostly English describing how something works without providing any code from the game.

This means it will NEVER contain any of the following:

- Visual or audio assets
- Unity assets files generated from an extractor
- Data files or dialogue files (even formatted versions)
- Any code methods body (signatures, enum values and field names are acceptable)
- Any binary files from the game

This isn't just for obvious legal reasons (mainly the code part), but also because these aren't actually helpful here. There are many reasons:

- For code, I heard a couple of people do the above and not provide context on the game's engineering which leads to extremely negative (and sometimes hostile) comments. This is because there are parts of the code that suffers from tight coupling or have a monolithic approach which can lead to developmental issues which affects modding. What people fails to mention is there's actually very good reason for this considering how the development process went, there are benefits that came out of this engineering approach and most importantly, it's JUST developmental issues. It doesn't actually affect the functionality because the whole game was built with this engineering in mind aka, it works. Rather than spend a lot of time to explain the context on why the game turned out this way, I rather just not provide them and only document on the stuff that ACTUALLY matters which is how the game works.
- For data files, it's simply because there's no point: the formats are not made to be human readable because automated tools generated them. The goal of this repos is to allow them to be readable by external tools or by a human manually. As for why not providing formatted versions of them, it's because they arguably don't belong in a docs repos which should only contain the specification of the data, but not its implementation. I rather decouple the 2 since the data won't change, but the knowledge of it can.
- For Unity assets, it's just because the actual instances (mainly the maps prefabs) don't matter as much for this repos. What matters is how they work so one can pick an actual instance and understand what it is telling the game to do.
- For anything else, it's not needed + the less files I share from the game, the better.
