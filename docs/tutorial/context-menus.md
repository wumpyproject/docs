# Application Command Context Menus

Context menus are another type of Discord application commands. These types of
commands are not invoked by typing `/`, rather, they show up as context menus
when right clicking their corresponding object.

Currently, Discord has support for user commands and message commands. That
is, you can right click a user or message and under Apps see these commands.

To define a context menu command, you should pass either `CommandType.user`
or `CommandType.message` to the `@app.command()` decorator. These types of
commands also take one additional argument depending on which type it is but
do not support more options. Take a look at the following examples:

=== "User context menu"

    ```python
    @app.command(CommandType.user)
    async def wave(
            interaction: CommandInteraction,
            user: Interactionuser
    ) -> None:
        """Wave hello to someone!"""
        await interaction.respond(f'{interaction.user} waves to {user} 👋')
    ```

=== "Message context menu"

    ```python
    @app.command(CommandType.message)
    async def reverse(
            interaction: CommandInteraction,
            message: Message,
    ) -> None:
        """Reverse a message's content."""
        # This is a neat indexing trick - by passing a step of -1 the
        # string gets reversed.
        await interaction.respond(message.content[::-1])
    ```

Context menus don't get more complicated than that, hence the documentation
won't go into it more.
