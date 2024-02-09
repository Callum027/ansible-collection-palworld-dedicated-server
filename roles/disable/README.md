# `callum027.palworld_dedicated_server.disable`

This stops the Palworld Dedicated Server service container, and deletes it
so it cannot run on startup.

The container itself is immutable, and not modified while running.
The Palworld save game data is stored on disk, so no data is lost in this operation.

For more information on using this role, check the documentation for the `callum027.palworld_dedicated_server` collection.
