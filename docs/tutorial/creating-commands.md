# Creating Application Commands

Application Commands are first entrypoint to Discord's interactions. In
[Getting set up] you started with a very trivial `/hello` command but there
much more to it.

## Anatomy of a command

Before going into other aspects of commands it is important to go through the
basics thoroughly. Take another look at the `/hello` command from earlier:

```python
@app.command()
async def hello(interaction: CommandInteraction) -> None:
    """Your first command; hello world."""
    await interaction.respond('...world')
```

First, the decorator `@app.command()` placed on the callback causes Wumpy to
turn it into a command and register it. In doing so, it reads the name `hello`
and docstring `"""Your first command; hello world."""`. These can also be
specified as keyword arguments to the decorator (`@app.command()`).

Later, after starting the bot and sending `/hello` in Discord to run the
command, Wumpy will call the function which executes the code inside of it.
The line `await interaction.respond('...world')` causes Wumpy to send
`...world` back to Discord.

Try changing the name, docstring, or text inside of the quotes to see the
difference when you try running the command!

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

```python
greet = app.group('greet', 'Group of greeting commands')
```

After doing this, you can now create commands under this group as usual:

```python
greet = app.group('greet', 'Group of greeting commands')


# Note that it says 'greet' and not 'app', otherwise it'd
# just register a normal command
@greet.command()
async def gif(interaction: CommandInteraction) -> None:
    """Greet someone with a waving GIF."""
    await interaction.respond(
        'https://tenor.com/view/pokemon-poliwhirl-wave-gif-19295706'
    )
```

This command, once called with `/greet gif`, will send a waving GIF. Restart
your bot and wait for the commands to update. You should be able to run it and
see the GIF it responds with:

...

Try nesting a subcommand group under the group by creating two GIF subcommands.
For example, here are two subcommands with waving and hugging GIFs:

```python
greet = app.group('greet', 'Group of greeting commands')


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

Try playing around with this more and when you're done continue reading about
passing options to commands in the next page.

## Interaction responses

Commands cannot always produce a result within the 3 second timeframe that
Discord requires a response to be sent. In this case you can instead call
`await interaction.defer()` which will display a "is thinking..." message
to the user.

```python
@app.command()
async def think(interaction: CommandInteraction) -> None:
    await interaction.defer()
```
