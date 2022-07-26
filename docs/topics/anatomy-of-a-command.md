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
