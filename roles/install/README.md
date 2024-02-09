# `callum027.palworld_dedicated_server.stop`

This playbook creates the files and directories for Palworld Dedicated Server,
starts the server, and configures it to run on startup.

Once the playbook has finished running, Palworld Dedicated Server will take a while to start
as it downloads required service files from Steam. To keep an eye on this process,
run the following command on the host (as a superuser, or user that is a member of the `docker` group)
to look at the service logs:

```bash
cd /opt/palworld-dedicated-server && docker compose logs -f palworld-dedicated-server
```

For more information on using this role, check the documentation for the `callum027.palworld_dedicated_server` collection.
