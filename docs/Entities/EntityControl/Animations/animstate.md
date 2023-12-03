# Animstate
The animstate of an [Entity](../../Entity.md) represents the current animation the entity should be on. It is an integer that allows most animations to be identified by. The system dictates a standard way to operate with animation clips while offering the ability to extend this standard via the [AnimSpecific](AnimSpecific.md) system. This allows to operate in Unity in such a way that the animstate system can track.

> NOTE: this field means something completely different for an [item entity](../Item%20entity.md).

## Standard animations
An animation is deemed to be within the standards of the animstate system if an animation clip of an entities's controller is named in a specific way. The name is either a predefined animation name or a number followed by optional letters. The name or number is referred to as the base name while the letters are optional arguments. Let's first explain the base name part.

### Standard animation clips base name
The base name can be defined in 2 ways: either it is a predefined animation name or an extra animation.

#### Predefined animations names
A predefined animation is any animstate below 100 (exclusive). However, while this means 100 animations can in theory predefined, this amount is overprovisioned: only the states ranging from 0 to 32 are valid states. This is because predefined animations are defined in an enum called MainManager.Animations which is defined like the following:

|Value|Name|
|-----:|----|
|0|Idle|
|1|Walk|
|2|Jump|
|3|Fall|
|4|ItemGet|
|5|Angry|
|6|Sad|
|7|Upset|
|8|Happy|
|9|Surprised|
|10|Flustered|
|11|Hurt|
|12|Death|
|13|BattleIdle|
|14|Sleep|
|15|Fallen|
|16|HurtFallen|
|17|WeakBattleIdle|
|18|KO|
|19|PickAction|
|20|WeakPickAction|
|21|Woobly|
|22|HurtWooble|
|23|Chase|
|24|Block|
|25|SleepFallen|
|26|AirTackle|
|27|ItemWalk|
|28|TossItem|
|29|Sit|
|30|FakeHurt|
|31|Dig|
|32|DigMove|

The enum name here is important: to define a predefined animations on a given controller, the clip MUST be named by the enum's name, NOT the value. The animstate will hold the value, but it won't load a clip named AS the value, it will load one named as the NAME of the enum value.

Because this is enforced by anything that attempts to set the animation, it can be safely assumed that those CAN exists, but they don't all have to. It's more a guarantee by the game that any animstate of 0 corresponds to "Idle" for example.

#### Extra animations
To define more animations, this must be done explicitly at the animation controller's level. This means the handling of those animations are not predefined at all and can be virtually anything. This corresponds to any animstate of 100 and above.

The base name of an animation clip in this range is the animstate value directly.

### Animation arguments
Any standard animations can contain optional arguments which is represented by letters appended at the end of a clip's base name. The arguments available are the following:

|Letter|Description|
|------:|-----------|
|b|back, refers to an entity's back sprite if applicable for the animation|
|t|talking, refers to an entity's equivalent animation when they are talking (This can be controlled by [SetText](../../../SetText/SetText.md))|
|f|flying, refers to an entity's equivalent animation when they are in the air|
|i|ice, refers to an entity's equivalent animation when they have been inflicted by an ice attack|
|u|UNUSED, this is ignored if the animation is set by force, but no animations is defined with this argument|

It is also possible to combine these arguments together by specifying them in a row, but the order much match the one used by the game from the order specified in Unity.

It should be noted that naming a clip with these arguments does NOT necessarily mean it is standard: it could simply not apply. In practice, the game suffixes the letter `b` to clips that only follow up on one, but the clip isn't applicable to be the back version of its matching clip.

## Non standard animations
Any animation clips of an [Entity](../../Entity.md)'s animation controller not named in a standard way cannot be played directly by the standard animation state system.

However, it is still possible to play clips that have a non standard name through various ways. One is by simply letting Unity do it via a transition from a standard named clip to a non standard one. Another is via the [AnimSpecific](AnimSpecific.md) systems which allows to extend the standard animation state system to do anything beyond its capabilities including non standard animation names.
