# Application Command Context Menus

Context menus are another type of Discord application commands. These types of
commands are not invoked by typing `/`, rather, they show up as context menus
when right clicking their corresponding object. This tutorial will walk through
remaking the previous `/text reverse` command into a context menu.

Currently, Discord has support for user commands and message commands. That
is, you can right click a user or message and under Apps see these commands.

## Defining context menus

Context menu commands cannot be nested as subcommands but are defined similar
to normal slash commands using the `@app.command()` decorator. To tell the
library to create a context menu command, you need to pass either
`CommandType.user` or `CommandType.message` to the decorator:

```python
@app.command(CommandType.message)
async def reverse(...) -> None:
    """Reverse a message's content."""
    ...
```

Now add the necessary parameters for the interaction and target (the user
or message, depending on context menu type):

```python
@app.command(CommandType.message)
async def reverse(
        interaction: CommandInteraction,
        message: Message,
) -> None:
    """Reverse a message's content."""
    ...
```

## Finalizing the command

Finally, just copy the text reversal from the previous `/text reverse` command,
but change `text` to `message.content`. It should look like this:

```python
@app.command(CommandType.message)
async def reverse(
        interaction: CommandInteraction,
        message: Message,
) -> None:
    """Reverse a message's content."""
    await interaction.respond(message.content[::-1])
```

Try implementing another context menu that reverses the name of the user it is
applied to! It should look like this:

```python
@app.command(CommandType.user)
async def reverse(
        interaction: CommandInteraction,
        message: InteractionUser,
) -> None:
    """Reverse a user's name."""
    await interaction.respond(user.name[::-1])
```

Once you understand defining commands in Wumpy, context menus don't get much
more complicated than this. Therefore not a lot of time is spent explaining or
walking through them. Continue to the next tutorial about advanced slash
command options!
