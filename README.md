# Palworld Dedicated Server Collection for Ansible

This is an Ansible collection for deploying and managing
[Palworld Dedicated Server](https://tech.palworldgame.com/dedicated-server-guide),
a dedicated server application for [Palworld](https://www.pocketpair.jp/palworld), a popular
multiplayer capable open-world survival game currently in early access.

This collection should make it easier to bring Palworld Dedicated Server
under configuration management on dedicated hardware or an IaaS cloud instance,
which is a recommended way of running Palworld as it is very resource intensive.

Palworld Dedicated Server will be run as a Docker container on a standard machine,
managed by Docker Compose.

## Requirements

This collection currently only supports Linux target hosts.
Currently, only Ubuntu 22.04 LTS is being tested and confirmed to work properly,
but it should work correctly on newer versions, and other distributions as well (e.g. Debian, Red Hat).

The following applications are required to be installed and running on target hosts:

* [Docker](https://docs.docker.com/desktop/install/linux-install)
* [Docker Compose V2](https://docs.docker.com/compose/install/linux) (as a Docker plugin)

## Architecture

The following file structure is created by the Ansible collection on the target host.

* `/opt/palworld` - Palworld install directory (configurable)
    * `data` - Palworld data directory, mounted into the container (Steam binaries, game configuration, world save data)
    * `docker-compose.yml` - Compose file (service container configuration)

## Inventory

A number of inventory variables need to be set on the host to be able to deploy Palworld Dedicated Server to it.

### Required

The following inventory variables are **required** to be set on the hosts that Palworld will be installed.

#### `palworld_dedicated_server_settings`

This variable defines the game settings for the Palworld Dedicated Server.
These will be set as environment variables in the container.

The following settings are **required** or highly recommended to be changed:

* `SERVER_NAME` - Name of the Palworld Dedicated Server as shown in Palworld. **Required**.
* `ADMIN_PASSWORD` - Administrator password for the server. **Required**.
* `SERVER_PASSWORD` - Set the password required to access the server, or set to a blank string to not require a password. **Required**.
* `SERVER_DESCRIPTION` - Description of the server when viewed from Palworld.
* `PUBLIC_IP` - Public IP address of the server, as accessible from the Internet.
* `COMMUNITY_SERVER` - Set to `true` if this is a community server, and `false` if it is a private server. When set to `false`, the server will not be added to the Palworld server list. Default is `true`.

This is how these would be set in the Ansible inventory:

```yaml
palworld_dedicated_server_settings:
  SERVER_NAME: Palworld Dedicated Server Ansible Collection
  SERVER_DESCRIPTION: Palworld Dedicated Server automatically provisioned from an Ansible collection.
  ADMIN_PASSWORD: "{{ vault_palworld_dedicated_server_admin_password }}"  # Store the value in Ansible Vault.
  SERVER_PASSWORD: "{{ vault_palworld_dedicated_server_server_password }}"  # Store the value in Ansible Vault.
  PUBLIC_IP: "192.0.2.1"
  COMMUNITY_SERVER: false
```

For more information on the available options, refer to the
[environment variable documentation](https://github.com/jammsen/docker-palworld-dedicated-server/blob/master/default.env).

When setting a value here, **make sure the type of the value is correct in Ansible**,
as the compose file template converts the values to environment variable definitions.

Take, for example, the following settings definition.

```yaml
palworld_dedicated_server_settings:
  SERVER_NAME: Palworld Dedicated Server Ansible Collection  # String type
  ADMIN_PASSWORD: "{{ vault_palworld_dedicated_server_admin_password }}"  # Should be a string type
  SERVER_PASSWORD: "{{ vault_palworld_dedicated_server_server_password }}"  # Should be a string type
  PUBLIC_IP: "192.0.2.1"  # String type
  COMMUNITY_SERVER: false  # Boolean type
  GUILD_PLAYER_MAX_NUM: 20  # Integer type
  PAL_CAPTURE_RATE: 1.0  # Float type
```

These will be rendered as:

```bash
SERVER_NAME=Palworld Dedicated Server Ansible Collection
ADMIN_PASSWORD=...
SERVER_PASSWORD=...
PUBLIC_IP=192.0.2.1
COMMUNITY_SERVER=false
GUILD_PLAYER_MAX_NUM=20
PAL_CAPTURE_RATE=1.000000
```

### Optional

The following variables are also available, with sane defaults already set. They can be changed, so the collection can be configured to better suit your environment:

* `palworld_dedicated_server_container_name` - The name of the Palworld Dedicated Server container on the system. When set to `null`, Docker Compose will automatically generate the container name. Default is `null`.
* `palworld_dedicated_server_container_timezone` - Container timezone, used for timekeeping and logging. Default is `Etc/UTC`.
* `palworld_dedicated_server_image_uri` - The URI for the image to install. Default is [`jammsen/palworld-dedicated-server`](https://hub.docker.com/r/jammsen/palworld-dedicated-server). **As this collection is designed around this image, this should not be changed.**
* `palworld_dedicated_server_image_tag` - The image tag to install. Default is `latest`.
* `palworld_dedicated_server_image_pull_policy`` Default is `missing` (pull the container only if it is missing).
* `palworld_dedicated_server_install_directory` - The target directory to install the compose file and service data folders to. Default is `/opt/palworld`.
* `palworld_dedicated_server_public_port` - The port to expose for connecting to the Palworld Dedicated Server. Default is `8211`.
* `palworld_dedicated_server_rcon_enable` - When set to `true`, enable RCON port access to the Palworld Dedicated Server. Default is `false`.
* `palworld_dedicated_server_rcon_bind_address` - Bind address for the RCON port. Set to `0.0.0.0` to allow access from other hosts on the network. Default is `127.0.0.1`.
* `palworld_dedicated_server_rcon_port` - RCON access port. Default is `25575`.
* `palworld_dedicated_server_rcon_wait_timeout` - Timeout for waiting for the RCON port to come online after the container starts, in seconds. This may need to be increased if the host's Internet connection is slow (as on the first run, the container takes a while to download Palworld after it starts). Default is `600`.
* `palworld_dedicated_server_user_name` - Name of the service user to create. Default is `palworld`.
* `palworld_dedicated_server_user_group` - Group name for the service user (created if it does not exist). Default is `palworld`.
* `palworld_dedicated_server_user_additional_groups` - List of additional groups to assign the service user to. Default is `[]` (do not assign any additional groups).
* `palworld_dedicated_server_user_uid` - Set a specific UID to use for the service user. Default is to let the operating system auto-generate it.
* `palworld_dedicated_server_user_gid` - Set a specific GID to use for the service group. Default is to let the operating system auto-generate it.

## Playbooks

The collection comes with a number of convenience playbooks for managing Palworld Dedicated Server
from the command line.

To use them, make sure the hosts you wish to install Palworld Dedicated Server on
are part of the `palworld` host group in the Ansible inventory.

### `callum027.palworld_dedicated_server.install`

This playbook creates the service users, directories and files for Palworld Dedicated Server,
starts the server, and configures it to run on startup.

```bash
ansible-playbook -i <path to inventory> callum027.palworld_dedicated_server.install
```

Once the playbook has finished running, Palworld Dedicated Server will take a while to start
as it downloads required service files from Steam. To keep an eye on this process,
run the following command on the host (as a superuser, or user that is a member of the `docker` group)
to look at the service logs:

```bash
cd /opt/palworld && docker compose logs -f palworld-dedicated-server
```

### `callum027.palworld_dedicated_server.update`

This playbook is similar to [`callum027.palworld_dedicated_server.install`](#callum027palworld_dedicated_serverinstall),
except it will always update and recreate the Palworld Dedicated Server container image if there is an update available.

```bash
ansible-playbook -i <path to inventory> callum027.palworld_dedicated_server.update
```

### `callum027.palworld_dedicated_server.restart`

This playbook restarts the Palworld Dedicated Service service container,
without recreating it or modifying it in any way.

Useful when you want to update Palworld itself without touching anything else,
as the container is configured to look for Palworld updates on startup by default.

```bash
ansible-playbook -i <path to inventory> callum027.palworld_dedicated_server.restart
```

Once the playbook has finished, keep an eye on Palworld update process
using the following command on the host:

```bash
cd /opt/palworld && docker compose logs -f palworld-dedicated-server
```

### `callum027.palworld_dedicated_server.stop`

This temporarily stops the Palworld Dedicated Server service container,
without destroying it or modifying it in any way.

Note that the service is still configured to launch on startup, so if the host reboots,
the service will restart.

```bash
ansible-playbook -i <path to inventory> callum027.palworld_dedicated_server.stop
```

### `callum027.palworld_dedicated_server.disable`

This stops the Palworld Dedicated Server service container, and deletes it
so it cannot run on startup.

The container itself is immutable, and not modified while running.
The Palworld save game data is stored on disk, so no data is lost in this operation.

```bash
ansible-playbook -i <path to inventory> callum027.palworld_dedicated_server.disable
```

### `callum027.palworld_dedicated_server.uninstall`

This stops the Palworld Dedicated Server instance (if running),
and completely removes all containers, service users, and files related to
Palworld Dedicated Server from the target hosts.

**This playbook will irrecoverably remove Palworld Dedicated Server and the save data
for the running game world. Make sure to backup any required data before running.**

```bash
ansible-playbook -i <path to inventory> callum027.palworld_dedicated_server.uninstall
```

## Roles

The following Ansible roles are provided. These roles perform the tasks run by the playbooks of the same name.

Using the roles directly allows you to run them on any host, or include them as part of another play.

* `callum027.palworld_dedicated_server.install`
* `callum027.palworld_dedicated_server.restart`
* `callum027.palworld_dedicated_server.stop`
* `callum027.palworld_dedicated_server.disable`
* `callum027.palworld_dedicated_server.uninstall`
