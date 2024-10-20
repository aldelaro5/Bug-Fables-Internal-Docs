# Miscellaneous features
Maps have other features that aren't easy to categorise while they involve [configuration fields](Fields/Configuration%20fields.md). They will be described here.

## `readdatafromothermap`
This feature involves the following configuration field:

|Name|Type|Init|Description|Default|
|---:|----|----|----------|-------|
|readdatafromothermap|MainManager.Maps|From prefab|When set to any value other than `TestRoom`, it will cause Start to read the `dialogues`, `commandlines` and `entities` TextAsset data from that map instead of the ones from `mapid`|`TestRoom`|

This mainly affects [CreateEntities](Init%20methods/CreateEntities.md) and the [SetText configuration](SetText%20configuration.md) where it will cause the data to be read from another map than the one being initialised.
