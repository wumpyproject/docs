# Creating Application Commands

Application Commands is the first entrypoint to Discord's interactions. In
[Getting set up](./getting-set-up/) you started with a very simple `/hello`
command, this tutorial expands on that with new greeting subcommands.

The code in this tutorial should be placed where the `...` is in the code
copied from [Getting set up](./getting-set-up): between defining `app =` and
calling `uvicorn.run()`.

## Defining subcommands

Start by creating a group to house the subcommand, it will need a name
(`'greet'`) and a description of what it does (`'Group of greeting commands'`):

```python
greet = app.group('greet', 'Group of greeting commands')
```

Now that you have the group, you can create a subcommand named `wave` which
responds with a simple waving emoji:

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

...

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
