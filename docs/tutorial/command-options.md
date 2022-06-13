# Application Command Options

Application Commands also have the ability to take input from the user. This
is done through options which are passed as arguments by the library.

## Specifying Command Options

Wumpy has support for specifying options in a variety of ways and aims to be
as flexible as possible. This means there are several ways to specify the
information that Discord needs.

To get started, take a look at this example of an `/echo` command:

```python
@app.command()
async def echo(
        interaction: CommandInteraction,
        text: str = Option(description='Text to repeat')
) -> None:
    """Have the bot echo back the specified text."""
    await interaction.respond(text)
```

Here the library will read the option name from the parameter - which in this
case is `text` - then continue analysing the annotion and default. From this,
it understands that the `/echo` command should have one option named `text`
with the description `'Text to repeat'` which takes in a string from the user.

If you do not want to use type annotations you can instead pass `type=`
to the `Option()` default.

### Alternate API

The second API available to specify options uses decorators instead of a
special parameter default:

```python
@option('text', description='Text to repeat')
@app.command()
async def echo(interaction: CommandInteraction, text: str) -> None:
    """Have the bot echo back the specified text."""
    await interaction.respond(text)
```

The name of the parameter is passed to the decorator, followed by the fields to
specify. You cannot create new options this way, each option needs a parameter
in the callback function.

This API is not recommended because the metadata from each option is moved away
from its definition and is not as extensible. You are free to use a combination
of these APIs as you wish, but remember to be consistent within your codebase!

## Option types

Wumpy supports a variety of annotations representing different option types:

- `str`
- `int`
- `bool`
- `float`
- `InteractionChannel`
- `User`
- `InteractionMember`

Special handling is done for `User` and `InteractionMember`. If a command
annotated with `InteractionMember` does not receive member information it will
not be called. This means that is is recommended to use
`Union[User, InteractionMember]` which causes the library to attempt to lookup
member information and fallback to a User - the command will always be called.

On top of this, the library handles more advanced typhints including
`Annotated`, certain `Union`s (such as `Optional` and
`Union[User, InteractionMember]`), as well as `Literal` and `Enum` subclasses
for [option choices](./option-choices.md).

## Optional and Required Options

Options can also be marked optional if they are not required. This is done by
specifying an option default. This default is only stored locally and will be
passed by the library when an optional option wasn't passed by the user.

Because the parameter default is already used by the library to specify option
metadata, the default can be passed as the first positional argument. Here the
default is an empty string `''` for the `signature` parameter:

```python
@app.command()
async def echo(
        interaction: CommandInteraction,
        text: str = Option(description='Text to repeat'),
        signature: text = Option('', description='Footer signature')
) -> None:
    """Have the bot echo back the specified text."""
    if signature:
        text += f'\n\n---\n{signature}'

    await interaction.respond(text)
```

If you are using the alternate decorator-based API, the actual parameter
is free to use. Therefore the default can just be specified by using the actual
parameter default. Alternatively, you can pass `default=` to the `@option()`
decorator.

```python
@option('text', description='Text to repeat')
@option('signature', description='Footer signature')
@app.command()
async def echo(
        interaction: CommandInteraction,
        text: str,
        signature: str = ''
) -> None:
    """Have the bot echo back the specified text."""
    if signature:
        text += f'\n\n---\n{signature}'

    await interaction.respond(text)
```

You can also have defaults of a different type than the option. For example,
the library special-cases `Optional` from `typing` and understands that the
default must be `None`:

```python
@app.command()
async def echo(
        interaction: CommandInteraction,
        text: str = Option(description='Text to repeat'),
        signature: Optional[str] = Option(description='Footer signature')
) -> None:
    """Have the bot echo back the specified text."""
    if signature is not None:
        text += f'\n\n---\n{signature}'

    await interaction.respond(text)
```

## Number bounds

The Discord API allows specifying a minimum and maximum value for integers
and floats. This is done by specifying a `min=` and `max=` to the `Option()`
parameter default or `@option()` default:

=== "Parameter default"

    ```python
    # The name of the command needs to be specified because if the function
    # name was hex() it would override the built-in and cause recursion.
    @app.command(name='hex')
    async def hex_cmd(
            interaction: CommandInteraction,
            num: int = Option(min=0, max=255, description='Number to convert')
    ) -> None:
        """Convert a number to its hex representation."""
        await interaction.respond(hex(num))
    ```

=== "Decorator API"

    ```python
    # The name of the command needs to be specified because if the function
    # name was hex() it would override the built-in and cause recursion.
    @option('num', min=0, max=255, description='Number to convert')
    @app.command(name='hex')
    async def hex_cmd(interaction: CommandInteraction, num: int) -> None:
        """Convert a number to its hex representation."""
        await interaction.respond(hex(num))
    ```
