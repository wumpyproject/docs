# Creating Application Commands

Application Commands is the first entrypoint to Discord's interactions. This
tutorial will walk through creating commands which make simple responses.

The code in this tutorial should be placed where the `...` is in the code
copied from [Getting set up](./getting-set-up): between defining `app =` and
calling `uvicorn.run()`.

## Starting with commands

To get a feel for how commands are defined, you can start by recreating
Discord's native `/shrug` command. This command takes no options and responds
with the text `'¯\_(ツ)_/¯'`.

Start by defining an asynchronous `shrug()` function decorated by
`@app.command()`. The function should take one argument being the interaction,
which contains contextual information about how it was called:

```python
@app.command()
async def shrug(interaction: CommandInteraction) -> None:
    ...
```

Next, give the function a docstring. The library reads the docstring's first
line as the description for the command:

```python
@app.command()
async def shrug(interaction: CommandInteraction) -> None:
    """Responds with shrugging ASCII art."""
    ...
```

This is all the boilerplate necessary to define the command. Now, respond to
the interaction by awaiting `interaction.respond()` with the shrugging
ASCII art as the first positional argument:

```python
@app.command()
async def shrug(interaction: CommandInteraction) -> None:
    """Responds with shrugging ASCII art."""
    await interaction.respond('¯\_(ツ)_/¯')
```

Save the file and restart the server. After a short while of waiting, Discord
will have updated and you should be able to invoke the command with `/shrug`.

## Defining subcommands

This tutorial will introduce many more commands. To organize all commands,
create a group to contain them all. Compared to commands which are created with
a decorator, groups are created by calling `.group()` with the name and
description of the group.

Create a group called `text` and change `@app.command()` to `@text.command()`
so that the command gets registered *under the group*. The new name for the
command is now `/text shrug`:

```python
text = app.group('text', 'Group of text manipulation commands')


# Note that it says 'text' and not 'app', otherwise it'd
# just register a normal command
@text.command()
async def shrug(interaction: CommandInteraction) -> None:
    """Responds with shrugging ASCII art."""
    await interaction.respond('¯\_(ツ)_/¯')
```

This is all it takes to create a subcommand! Try invoking the command again,
but this time with `/text shrug`.

## Nested subcommads

Discord also allows nesting one level deeper - making subcommand groups.
In preparation for another command which responds with ASCII art, create an
`art` subcommand group and move the `shrug` command below it:

```python
text = app.group('text', 'Group of text manipulation commands')


art = text.group('art', 'Group of commands for ASCII art')


@art.command()
async def shrug(interaction: CommandInteraction) -> None:
    """Responds with shrugging ASCII art."""
    await interaction.respond('¯\_(ツ)_/¯')
```

Now that you have the subcommand group ready, create another subcommand called
`tableflip` which replicates the equivalent native `/tableflip` command and
responds with `'(╯°□°）╯︵ ┻━┻'`:

```python
text = app.group('text', 'Group of text manipulation commands')


art = text.group('art', 'Group of commands for ASCII art')


@art.command()
async def shrug(interaction: CommandInteraction) -> None:
    """Responds with shrugging ASCII art."""
    await interaction.respond('¯\_(ツ)_/¯')


@art.command()
async def tableflip(interaction: CommandInteraction) -> None:
    """Responds with a tableflip ASGII art."""
    await interaction.respond('(╯°□°）╯︵ ┻━┻')
```

With the above code, you have now created two subcommands:
`/text art shrug` and `/text art tableflip`.

If you want to read an in-depth explanation of defining commands and nesting
subcommands, see [Anatomy of a command](../topics/anatomy-of-a-command),
otherwise continue with the tutorial to learn more about command options.
