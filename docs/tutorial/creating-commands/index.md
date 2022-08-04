# Creating Application Commands

Application Commands is the first entrypoint to Discord's interactions. This
tutorial will walk through creating simple greeting commands and how to make
use of subcommands to categorize them.

The code in this tutorial should be placed where the `...` is in the code
copied from [Getting set up](./getting-set-up): between defining `app =` and
calling `uvicorn.run()`.

## Starting with commands

Start by defining an asynchronous `greet()` function decorated by
`@app.command()`. The function should take one argument being the interaction,
which contains contextual information about how it was called:

```python
@app.command()
async def greet(interaction: CommandInteraction) -> None:
    ...
```

Next, give the function a docstring. The library reads the docstring's first
line as the description for the command:

```python
@app.command()
async def greet(interaction: CommandInteraction) -> None:
    """Greet someone with a waving emoji."""
    ...
```

This is all boilerplate necessary to define a command which has no additional
options. To respond to the interaction with a message, call and await the
`interaction.respond()` method with the content:

```python
@app.command()
async def greet(interaction: CommandInteraction) -> None:
    """Greet someone with a waving emoji."""
    await interaction.respond('👋')
```

Save the file and restart the server. After a short while of waiting, Discord
will have updated and you should be able to invoke the command with `/greet`.

## Defining subcommands

This tutorial will introduce many more greeing commands, to organize this,
create a group to contain them all. Compared to commands which are created with
a decorator, groups are created by calling `.group()` with the name and
description of the roup.

Create a group called `greet` and change `@app.command()` to `@greet.command()`
so that the command gets registered *under the group*. Finally, rename the
command you created above to `wave` - this will make the full name of the
command `/greet wave`.

```python
greet = app.group('greet', 'Group of greeting commands')


# Note that it says 'greet' and not 'app', otherwise it'd
# just register a normal command
@greet.command()
async def wave(interaction: CommandInteraction) -> None:
    """Greet someone with a waving emoji."""
    await interaction.respond('👋')
```

This is all it takes to create a subcommand. Now restart the bot and go back
to your DMs with the bot to try out the command by typing `/greet wave`! You
should get a waving emoji in response:

![Bot responding to `/greet wave` command interaction](./used-greet-command.png)

## Nested subcommads

Discord also allows nesting one level deeper - making subcommand groups.
Add a `gif` group, nested underneath the original `greet` group, in
preparation for more subcommands:

```python
greet = app.group('greet', 'Group of greeting commands')


@greet.command()
async def wave(interaction: CommandInteraction) -> None:
    """Greet someone with a waving emoji."""
    await interaction.respond('👋')


gif = greet.group('gif', 'Greet someone with a GIF')
```

Now create the two subcommands *under the `gif` subcommand group* which respond
with different GIFs that embed:

```python
greet = app.group('greet', 'Group of greeting commands')


@greet.command()
async def wave(interaction: CommandInteraction) -> None:
    """Greet someone with a waving emoji."""
    await interaction.respond('👋')


gif = greet.group('gif', 'Greet someone with a GIF')


@gif.command()
async def waving(interaction: CommandInteraction) -> None:
    """Greet someone with a waving GIF."""
    await interaction.respond('https://tenor.com/bs7Re.gif')


@gif.command()
async def hugging(interaction: CommandInteraction) -> None:
    """Greet someone with a hugging GIF."""
    await interaction.respond('https://tenor.com/bbQCJ.gif')
```

You have now created 3 different subcommands at different levels of nesting:
`/greet wave`, `/greet gif waving`, and `/greet gif hugging`!

If you want to read an in-depth explanation of defining commands and nesting
subcommands, see [Anatomy of a command](../topics/anatomy-of-a-command),
otherwise continue with the tutorial to learn more about command options.
