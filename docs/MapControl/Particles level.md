# Particles level
Some GameObject can have their enablement controlled by a game settings.

Such a GameObject has a `Particable`. All these GameObjects are processed on SetParticles which is only invoked by the first LateUpdate (`latestart`).

What happens is they all get disabled if the `particlelevel` setting is 0 (OFF). There is no differences in behavior if it's 1 (LOW) or 2 (HIGH).

However, the setting can impact other things that are meant to be in the map:

- Any LilypadEffects will start disabled if the value is 0 (OFF)
- Any PrefabParticles will have their behavior change depending on the setting's value:
    - 0 (OFF): The GameObject of the component gets disabled
    - 1 (LOW): `maxammount` is halved floored. This means that the particles will still spawn, but they are limited to only have half of what they can spawn
    - 2 (HIGH): No changes in behaviors
