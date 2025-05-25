# Obtaining values
Various SetText commands offers the ability to obtain specific interesting value from a string or integer. What's confusing about this ability is the logic is very similar between 2 methods, but they don't all support the same things. Additionally, several commands modifies these methods with their own thing giving different parsing logic per command.

Because of this, everything surrounding the parsing of string/integer to values is documented specifically for each commands. Nonetheless, this page exists to document the 2 methods that most commands uses to base their value parsing logic on: GetValueFromString and IntFromString.

## GetValueFromString

```cs
public static int GetValueFromString(string input)
```
A SetText command utility used to obtain a value from a string `input` that can be formatted in multiple ways. Here are the format it supports (checked and done in order):

- `money`: Returns instance.`money`
- `vX` or `varX`: Returns the value of the flagvar slot `X`
- Anything else: Returns `input` converted to int

## IntFromString

```cs
private static int IntFromString(string input)
```
A SetText command utility used to obtain a value from a string `input` that can be formatted in multiple ways. Here are the format it supports (checked and done in order):

- `vX` or `varX`: Returns the value of the flagvar slot `X`
- Anything else: Returns `input` converted to int

NOTE: It's essentially the same as GetValueFromString, but without support for the `money` value.
