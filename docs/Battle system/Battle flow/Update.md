# Update
This Unity event is the main dispatcher of the battle flow. It only handles the synchronous high level flow that runs as long as control wasn't delegated to an action coroutine.

There are 3 possible flows to a BattleControl update: terminal, uncontrolled and controlled. The flows are documented in their own page.

The first flow whose conditions are fufilled will apply from the following table:

|Flow|Conditions|Description|
|---:|----------|-----------|
|[Terminal](Update%20flows/Terminal%20flow.md)|`cancelupdate` is true or instance.`pausemenu` exists|A flow where Update relegates almost complete control to a terminal coroutine which will end or retry this battle|
|[Uncontrolled](Update%20flows/Uncontrolled%20flow.md)|`action` or `inevent` is true|A flow where Update delegates most (but not all) control to an action coroutine which performs a specific battle action|
|[Controlled](Update%20flows/Controlled%20flow.md)|The battle isn't in a terminal or uncontrolled flow|A flow where Update has complete control of the battle and will act as a dispatcher according to the main turn procedure|
