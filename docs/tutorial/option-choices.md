# Application Command Option Choices

Option choices allow you to restrict what the user is allowed to input. In
this tutorial another subcommand will be added which styles text with
Discord's flavour of Markdown.

## Initial command boilerplate

At this point in the tutorial, options and commands have already been
explained so that won't be repeated here.

Define a subcommand named `markdown` which a suitable descriptions that has
two options - `style` and `text` - both annotated as `str`:

```python
text = app.group('text', 'Group of text manipulation commands')


@text.command()
async def markdown(
        interaction: CommandInteraction,
        style: str = Option(description='New style of the text'),
        text: str = Option(description='Text to style with Markdown'),
) -> None:
    """Apply Markdown styles to the passed text."""
    ...
```

## Specifying command option choices

With the current definition of the command, the user is allowed to input
anything for the `style` option. To change this, you need to specify the
choices they can decide between.

Specifying choices is done by passing `choices=` to `Option()` with a list
or dictionary of all the choices available. In this case that list will be
`['bold', 'italic', 'underline']`:

```python
text = app.group('text', 'Group of text manipulation commands')


@text.command()
async def markdown(
        interaction: CommandInteraction,
        style: str = Option(
            description='New style of the text',
            choices=['bold', 'italic', 'underline']
        ),
        text: str = Option(description='Text to style with Markdown'),
) -> None:
    """Apply Markdown styles to the passed text."""
    ...
```

Since Python has a native way to represent a string being of specific values
using `Literal`, the library can read this and make out the choices. Doing
this has the benefit of improving the editor experience while implementing
the command.

Remove the `choices=` parameter and instead annotate the option
with `Literal['bold', 'italic', 'underline']`:

```python
text = app.group('text', 'Group of text manipulation commands')


@text.command()
async def markdown(
        interaction: CommandInteraction,
        style: Literal['bold', 'italic', 'underline'] = Option(
            description='New style of the text',
        ),
        text: str = Option(description='Text to style with Markdown'),
) -> None:
    """Apply Markdown styles to the passed text."""
    ...
```

## Implementing the created command

Now that only `'bold'`, `'italic'`, and `'underline'` can be passed as the
`style` option, it is time to implement the command. Create a dictionary which
maps the style to the wrapping symbols:

```python
text = app.group('text', 'Group of text manipulation commands')


@text.command()
async def markdown(
        interaction: CommandInteraction,
        style: Literal['bold', 'italic', 'underline'] = Option(
            description='New style of the text',
        ),
        text: str = Option(description='Text to style with Markdown'),
) -> None:
    """Apply Markdown styles to the passed text."""
    mapping = {
        'bold': '**',
        'italic': '*',
        'underline': '__',
    }
```

Finally, respond to the interaction with the symbols inserted in front of- and
behind the text:

```python
text = app.group('text', 'Group of text manipulation commands')


@text.command()
async def markdown(
        interaction: CommandInteraction,
        style: Literal['bold', 'italic', 'underline'] = Option(
            description='New style of the text',
        ),
        text: str = Option(description='Text to style with Markdown'),
) -> None:
    """Apply Markdown styles to the passed text."""
    mapping = {
        'bold': '**',
        'italic': '*',
        'underline': '__',
    }

    wrapper = mapping[style]
    await interaction.respond(wrapper + text + wrapper)
```
