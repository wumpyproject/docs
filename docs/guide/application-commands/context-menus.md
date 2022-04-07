# Context menus

Previously we have went through slash commands, also called chat input commands. There are more
commands than this, and on this page we will deep dive into user commands.

## Creating a user command

All you need to create a user command is to pass in a different application command type:

```python
from wumpy import interactions


app = interactions.InteractionApp(...)


@app.command(interactions.CommandType.user)
async def wave(
    interaction: interactions.CommandInteraction,
    user: interactions.InteractionUser
) -> None:
    """Wave hello to someone!"""
    pass
```

User commands cannot take options, all they receive are the user they were invoked on.

Let's respond to the interaction:

```python
from wumpy import interactions


app = interactions.InteractionApp(...)


@app.command(interactions.CommandType.user)
async def wave(
    interaction: interactions.CommandInteraction,
    user: interactions.InteractionUser
) -> None:
    """Wave hello to someone!"""
    await interaction.respond(f'{interaction.user} waves to {user} 👋')
```

## Creating message commands

Creating message commands is very similar to how user commands are made, just change the enum
value like this:

```python
from wumpy import interactions, models


app = interactions.InteractionApp(...)


@app.command(interactions.CommandType.message)
async def reverse(
    interaction: interactions.CommandInteraction,
    message: models.Message
) -> None:
    """Reverse a message's content."""
    pass
```

The command will always be given the interaction generated and message the command was invoked
with but nothing else.

Like with user commands you cannot use options with message commands.

Now that we understand message commands, let's finish the implementation:

```python
from wumpy import interactions, models


app = interactions.InteractionApp(...)


@app.command(interactions.CommandType.message)
async def reverse(
    interaction: interactions.CommandInteraction,
    message: models.Message
) -> None:
    """Reverse a message's content."""
    await interaction.respond(message.content[::-1])
```

!!! info
    The reversal implementation is a neat indexing trick, by passing a "step" with -1 we create
    a string with all characters reversed.

Like with user commands it's *that* easy so the documentation won't explore it more than this.
There is not much more to it.
