# Application Command Options

Application Commands also have the ability to take input from the user. This
is done through options which are passed as arguments by the library. In this
tutorial, you will create another text command which reverses the content.

## Creating the command

To start off, add the command boilerplate for the subcommand under the `text`
group. This code fits into the code in the previous page:

```python
# The following line will already be in your code from the previous page, but
# it is included for the purpose of navigating where to place the subcommand.
text = app.group('text', 'Group of text manipulation commands')


@text.command()
async def reverse(interaction: CommandInteraction) -> None:
    """Reverse the passed text content."""
    ...
```

## Adding the text command option

Before you can respond to the interaction, you'll need to define an option the
user can pass with the text to reverse. This is done by adding another
parameter to the function.

Add a parameter named `text` with the annotation `str`. The default of this
parameter should be calling `Option()`:

```python
text = app.group('text', 'Group of text manipulation commands')


@text.command()
async def reverse(
        interaction: CommandInteraction,
        text: str = Option(),
) -> None:
    """Reverse the passed text content."""
    ...
```

Discord requires that each option has a name and description. The library is
able to read the parameter name, but you will need to pass the description you
wish to use to `Option()`:

```python
text = app.group('text', 'Group of text manipulation commands')


@text.command()
async def reverse(
        interaction: CommandInteraction,
        text: str = Option(description='Text to reverse'),
) -> None:
    """Reverse the passed text content."""
    ...
```

Now that you have the text to reverse, you can respond to the interaction:

```python
text = app.group('text', 'Group of text manipulation commands')


@text.command()
async def reverse(
        interaction: CommandInteraction,
        text: str = Option(description='Text to reverse'),
) -> None:
    """Reverse the passed text content."""
    # This is implemented as a nice indexing trick, by setting the
    # step to -1 it gets walked backwards
    await interaction.respond(text[::-1])
```

Save the file and restart the server. You can now invoke this command with
`/text reverse` and pass a msg to reverse. For example:

```text
/text reverse text: racecar
```

## Multiple command options and defaults

To demonstrate using multiple command options and setting option default
values, create another command for repeating some text multiple times.
Place the command under the same `text` group and name it `repeat`:

```python
# Similar to before, the following line will already be in your code from the
# previous page. You can place the command below the 'reverse' command
text = app.group('text', 'Group of text manipulation commands')


@text.command()
async def repeat(
        interaction: CommandInteraction,
) -> None:
    """Repeat the passed text a number of times."""
    ...
```

This command will need two options: one for the text to repeat and the second
for the number of times. Create two parameters for them and name them `text`
and `number`, with the annotations `str` and `int` for string- and integer
options:

```python
text = app.group('text', 'Group of text manipulation commands')


@text.command()
async def repeat(
        interaction: CommandInteraction,
        text: str = Option(description='The text to repeat'),
        number: int = Option(description='The number of times to repeat'),
) -> None:
    """Repeat the passed text a number of times."""
    ...
```

Now, it isn't very necessary to pass `repeat` every time if it is often the
same value. Give the option a default by passing a positional argument or
using `default=`:

```python
text = app.group('text', 'Group of text manipulation commands')


@text.command()
async def repeat(
        interaction: CommandInteraction,
        text: str = Option(description='The text to repeat'),
        number: int = Option(5, description='The number of times to repeat'),
) -> None:
    """Repeat the passed text a number of times."""
    ...
```

The library will now mark the option as non-required and automatically pass the
default when it isn't passed. Add the missing implementation and try running
the command with `/text repeat`:

```python
text = app.group('text', 'Group of text manipulation commands')


@text.command()
async def repeat(
        interaction: CommandInteraction,
        text: str = Option(description='The text to repeat'),
        number: int = Option(5, description='The number of times to repeat'),
) -> None:
    """Repeat the passed text a number of times."""
    await interaction.respond(text * number)
```

You have now created commands with several options with different option types.
To read more about the option API and commands, take a look at
[Anatomy of a command](../topics/anatomy-of-a-command.md). Otherwise, just
continue with the tutorial to learn about specifying command choices.
