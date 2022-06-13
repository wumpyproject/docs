# Application Command Option Choices

Option choices allow you to restrict what the user is allowed to input.
There can be up to 25 choices which can be set for integers, floats, or
strings.

If you want to mix between different types, such as having an integer choice
and a string choice, then it is best to use the strig type and "stringify"
the interger. It is not possible to mix types.

## Defining Option Choices

Wumpy has several ways to actually define which choices the user has. The
most basic way is to pass `choices=...` to the `Option()` parameter default
or `@option()` decorator:

```python
@app.command()
async def rps(
        interaction: CommandInteraction,
        play: str = Option(
            description='The play to make against the bot',
            choices=['rock', 'paper', 'scissors']
        ),
) -> None:
    """Play rock, paper, scissors against the bot."""
    pass
```

When your command callback is called, the parameter will be one of the passed
values.

The choices can also be defined as a dictionary with the key being the name of
the choice presented to the user and the value being the value passed to the
command callback:

```python
@app.command()
async def rps(
        interaction: CommandInteraction,
        play: str = Option(
            description='The play to make against the bot',
            choices={'rock': 'r', 'paper': 'p', 'scissors': 's']
        ),
) -> None:
    """Play rock, paper, scissors against the bot."""
    pass
```

In this example, when the user selects the `'paper'` choice while invoking the
command, the command callback's `play` parameter will be `p`.

## Choices in Annotations

Since the library reads the parameter annotations, it is also possible to use
the `Literal` annotation for this. This works similar to passing a list to
the `choices=` keyword argument:

```python
@app.command()
async def rps(
        interaction: CommandInteraction,
        play: Literal['rock', 'paper', 'scissors'] = Option(
            description='The play to make against the bot',
        ),
) -> None:
    """Play rock, paper, scissors against the bot."""
    pass
```

### Representing Choices with Enums

Wumpy can also read an Enum as a set of choices if you prefer working with that
as opposed to strings and literals. To utilize this feature, start by creating
an Enum subclass with the choices the user should have:

```python
from enum import Enum


class RockPaperScissors(Enum):
    rock = 'rock'
    paper = 'paper'
    scissors = 'scissors'
```

The name of the enum member is what will be shown to the user. You can place
much more complex types in the enum value than strings, integers and floats as
these are not sent to Discord.

You can now annotate the option with this Enum subclass and Wumpy will read
the members as choices for the user:

```python
from enum import Enum


class RockPaperScissors(Enum):
    Rock = 'rock'
    Paper = 'paper'
    Scissors = 'scissors'


@app.command()
async def rps(
        interaction: CommandInteraction,
        play: RockPaperScissors = Option(
            description='The play to make against the bot',
        ),
) -> None:
    """Play rock, paper, scissors against the bot."""
    pass
```

When the command is dispatched the command callback will receive the
corresponding Enum instance of the choice the user made.
