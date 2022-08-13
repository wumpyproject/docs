# Advanced Application Command Options

Up until now only primitive option types such as integers and strings have
been mentioned in the tutorial, but Discord supports passing channels, users,
and roles as options as well. This page will walk through creating a command
to rAnDOm CaSE a user's name.

## Command boilerplate

As with previous pages, at this stage in the tutorial the basic boilerplate
of a command has already been explained so this will skip right to creating
the option. Create another subcommand called `/text randomcase` with an
`target` option:

```python
text = app.group('text', 'Group of text manipulation commands')


@text.command()
async def randomcase(
        interaction: CommandInteraction,
        target: ... = Option(description='Target to random case'),
) -> None:
    """Random case another user's username."""
    ...
```

## Annotating advanced options

Exactly the same way you can annotate primitive option types, the library
reads annotations for the option types. Annotate the `target` parameter with
`InteractionUser` to create a user option:

```python
text = app.group('text', 'Group of text manipulation commands')


@text.command()
async def randomcase(
        interaction: CommandInteraction,
        target: InteractionUser = Option(description='Target to random case'),
) -> None:
    """Random case another user's username."""
    ...
```

Finally, you can implement the command and try it out:

```python
text = app.group('text', 'Group of text manipulation commands')


@text.command()
async def randomcase(
        interaction: CommandInteraction,
        target: InteractionUser = Option(description='Target to random case'),
) -> None:
    """Random case another user's username."""
    await interaction.respond(''.join(
        c.upper() if round(random.random()) else c.lower()
        for c in target.name
    ))
```

## Special mentionable option type

On top of the user type, Discord has a special option type called "mentionable"
which - as the name suggests - allows the user to pick either users or roles.

Wumpy does not have a special type for this, instead you can annotate a union
of `InteractionUser` and `Role`. Extend the random-case to support roles as
well by updating the annotation:

```python
text = app.group('text', 'Group of text manipulation commands')


@text.command()
async def randomcase(
        interaction: CommandInteraction,
        target: Union[InteractionUser, Role] = Option(
            description='Target to random case',
        ),
) -> None:
    """Random case another user's username."""
    await interaction.respond(''.join(
        c.upper() if round(random.random()) else c.lower()
        for c in target.name
    ))
```
