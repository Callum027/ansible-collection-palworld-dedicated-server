# `callum027.palworld_dedicated_server.restart`

This playbook restarts the Palworld Dedicated Service service container,
without recreating it or modifying it in any way.

Useful when you want to update Palworld itself without touching anything else,
as the container is configured to look for Palworld updates on startup by default.

Once the playbook has finished, keep an eye on Palworld update process
using the following command on the host:

```bash
cd /opt/palworld-dedicated-server && docker compose logs -f palworld-dedicated-server
```

For more information on using this role, check the documentation for the `callum027.palworld_dedicated_server` collection.
