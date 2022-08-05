# Anatomy of a command

This page explains how application commands work in Wumpy and how they have
been designed.

## Implicit information

The library takes advantage of runtime introspection to be able to implicitly
extract information from the command callback. Take the following `/hello`
command as an example:

```python
@app.command()
async def hello(interaction: CommandInteraction) -> None:
    """Your first command; hello world."""
    await interaction.respond('...world')
```

First, the decorator `@app.command()` placed on the callback causes Wumpy to
turn it into a command and register it. In doing so, it implicitly reads the
name `hello` and docstring `"""Your first command; hello world."""` which get
set as command's name and description. If the docstring spans multiple lines,
only the first line (called the summary) is used for the command's description.

Additionally, the library also reads the callback's parameter list for the
command options. Take a look at the following more complex example from the
command options tutorial:

```python
@app.command()
async def echo(
        interaction: CommandInteraction,
        text: str = Option(description='Text to repeat'),
        signature: str = Option('', description='Footer signature')
) -> None:
    """Have the bot echo back the specified text."""
    if signature:
        text += f'\n\n---\n{signature}'

    await interaction.respond(text)
```

The library reads the parameter list and understands that there are two options
named `text` and `signature`. The callback's defaults contain the option
instances used and the annotations of `str` means that the option type is set
to string.

## Subcommands and subcommand groups

Discord allows nesting of commands in subcommands and subcommand groups. In
combination you can nest subcommands up to 3 levels deep, see the following
examples of valid combinations:

```
command           command
|                 |
|__ subcommand    |_ subcommand-group
|                    |
|__ subcommand       |_ subcommand
```

You can of course also nest subcommand next to subcommand groups as follows:

```
command
|
|__ subcommand-group
    |
    |__ subcommand
|
|__ subcommand
```

Commands and subcommand groups that hold subcommands cannot be called on their
own, that's why these types of commands are created differently - using the
`.group()` method. Compared to `.command()`, `.group()` is not a decorator:

```python
greet = app.group('greet', 'Group of greeting commands')
```

The library fluently uses either a (sub)command or group for the top-level
command, and makes no distinction between the two. This means that a
double-nested subcommand is represented as
`subcommand-group -> subcommand-group -> subcommand` (compared to Discord's
official `command -> subcommand-group -> subcommand`.

### History of library representation

Previously the library had 3 classes matching Discord's: `Command`,
`SubcommandGroup`, `Subcommand`. However, the implementation became messy
because `Command` was forced to handle its callback being `None`. To reduce
duplication of code, the implementation of `Command` was actually a subclass
of both `SubcommandGroup` and `Subcommand` which meant that it was instead
`Subcommand` which had to handle the missing callback.

The library is designed to be statically typed, to aid with development and
reduce unexpected or complicated behaviour. Having a lot of dynamicness leads
to code which needs to be taught rather than simply read. This meant that
the dynamicness of having the callback be missing was unfavourable and
considered janky.

When redesigned to the current system, this was adjusted to remove the
forced top-level command. There is now only two representations:
`SubcommandGroup`, and `Command`. `SubcommandGroup` is mostly unchanged apart
from adding a `.group()` method, since it now used for both levels of nesting.
As a side-effect of this, there is nothing stopping the user from nesting
commands further than allowed by Discord - except for Discord itself which will
raise an error about invalid configuration if the user attempts this.

## Command option APIs

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

### Option types

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
for option choices.

### Optional and Required Options

Options can also be marked optional if they are not required. This is done by
specifying an option default. This default is only stored locally and will be
passed by the library when an optional option wasn't passed by the user.

Because the parameter default is already used by the library to specify option
metadata, the default is passed as the first positional argument. Here the
default is an empty string `''` for the `signature` parameter:

```python
@app.command()
async def echo(
        interaction: CommandInteraction,
        text: str = Option(description='Text to repeat'),
        signature: str = Option('', description='Footer signature')
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

### Number bounds

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
