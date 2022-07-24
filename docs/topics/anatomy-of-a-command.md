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
`.group()` method. Try creating a group, like this:
