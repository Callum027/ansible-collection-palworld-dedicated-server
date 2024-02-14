# Palworld Dedicated Server Collection for Ansible

[![GitHub License](https://img.shields.io/github/license/Callum027/ansible-collection-palworld-dedicated-server)](https://github.com/Callum027/ansible-collection-palworld-dedicated-server/blob/main/LICENSE) [![GitHub Release](https://img.shields.io/github/v/release/Callum027/ansible-collection-palworld-dedicated-server)](https://galaxy.ansible.com/ui/repo/published/callum027/palworld_dedicated_server) [![GitHub Actions Workflow Status](https://img.shields.io/github/actions/workflow/status/Callum027/ansible-collection-palworld-dedicated-server/test.yml?label=tests)](https://github.com/Callum027/ansible-collection-palworld-dedicated-server/actions/workflows/test.yml)

This is an Ansible collection for deploying and managing
[Palworld Dedicated Server](https://tech.palworldgame.com/dedicated-server-guide),
a dedicated server application for [Palworld](https://www.pocketpair.jp/palworld), a popular
multiplayer capable open-world survival game currently in early access.

This collection should make it easier to bring Palworld Dedicated Server
under configuration management on dedicated hardware or an IaaS cloud instance,
which is a recommended way of running Palworld as it is very resource intensive.

Palworld Dedicated Server is orchestrated as a Docker container on a standard machine,
managed by Docker Compose. The Docker container used is
[`jammsen/palworld-dedicated-server`](https://github.com/jammsen/docker-palworld-dedicated-server),
a regularly maintained container for Palworld Dedicated Server with strong community support,
that eliminates the usual manual setup steps required for configuring Palworld.

From version 2.0.0, this collection can optionally setup a
[Prometheus exporter for Palworld Dedicated Server (designed by palworld.lol)](https://github.com/palworldlol/palworld-exporter)
alongside your server. This enables localised metrics collection from your dedeicated server,
which can then be turned into dashboards using [Grafana](https://grafana.com).

## Requirements

This collection currently only supports Linux target hosts.
Currently, only Ubuntu 22.04 LTS is being tested and confirmed to work properly,
but it should work correctly on newer versions, and other distributions as well (e.g. Debian, Red Hat).

The following applications are required to be installed and running on target hosts:

* [Docker](https://docs.docker.com/desktop/install/linux-install)
* [Docker Compose V2](https://docs.docker.com/compose/install/linux) (as a Docker plugin)

## Architecture

The following file structure is created by the Ansible collection on the target host.

* `/opt/palworld-dedicated-server` - Palworld install directory (configurable)
    * `data` - Palworld data directory, mounted into the container (Steam binaries, game configuration, world save data)
    * `docker-compose.yml` - Compose file (service container configuration)

## Inventory

A number of inventory variables need to be set on the host to be able to deploy Palworld Dedicated Server to it.

### Required

The following inventory variables are **required** to be set on the hosts that Palworld will be installed.

#### `palworld_dedicated_server_admin_password`

*New in version 2.0.0.*

Administrator password for the server.

Used as the password when connecting to the RCON remote administration interface,
and used internally to allow additional supporting servers to connect to the dedicated server,
such as the Prometheus exporter.

```yaml
palworld_dedicated_server_admin_password: "{{ vault_palworld_dedicated_server_admin_password }}"  # Store the value in Ansible Vault.
```

#### `palworld_dedicated_server_settings`

*Changed in version 2.0.0*: The `ADMIN_PASSWORD` field is no longer used by the collection.
The administrator password for the dedicated server must now be set using the
[`palworld_dedicated_server_admin_password`](#palworld_dedicated_server_admin_password) inventory variable.

This variable defines the game settings for the Palworld Dedicated Server.
These will be set as environment variables in the container.

The following settings are **required** or highly recommended to be changed:

* `SERVER_NAME` - Name of the Palworld Dedicated Server as shown in Palworld. **Required**.
* `SERVER_PASSWORD` - Set the password required to access the server, or set to a blank string to not require a password. **Required**.
* `SERVER_DESCRIPTION` - Description of the server when viewed from Palworld.
* `PUBLIC_IP` - Public IP address of the server, as accessible from the Internet.
* `COMMUNITY_SERVER` - Set to `true` if this is a community server, and `false` if it is a private server. When set to `false`, the server will not be added to the Palworld server list. Default is `true`.

This is how these would be set in the Ansible inventory:

```yaml
palworld_dedicated_server_settings:
  SERVER_NAME: Palworld Dedicated Server Ansible Collection
  SERVER_DESCRIPTION: Palworld Dedicated Server automatically provisioned from an Ansible collection.
  SERVER_PASSWORD: "{{ vault_palworld_dedicated_server_server_password }}"  # Store the value in Ansible Vault.
  PUBLIC_IP: "192.0.2.1"
  COMMUNITY_SERVER: false
```

For more information on the available options, refer to the
[environment variable documentation](https://github.com/jammsen/docker-palworld-dedicated-server/blob/master/README_ENV.md).

When setting a value here, **make sure the type of the value is correct in Ansible**,
as the compose file template converts the values to environment variable definitions.

Take, for example, the following settings definition.

```yaml
palworld_dedicated_server_settings:
  SERVER_NAME: Palworld Dedicated Server Ansible Collection  # String type
  SERVER_PASSWORD: "{{ vault_palworld_dedicated_server_server_password }}"  # Should be a string type
  PUBLIC_IP: "192.0.2.1"  # String type
  COMMUNITY_SERVER: false  # Boolean type
  GUILD_PLAYER_MAX_NUM: 20  # Integer type
  PAL_CAPTURE_RATE: 1.0  # Float type
```

These will be rendered as:

```bash
SERVER_NAME=Palworld Dedicated Server Ansible Collection
SERVER_PASSWORD=...
PUBLIC_IP=192.0.2.1
COMMUNITY_SERVER=false
GUILD_PLAYER_MAX_NUM=20
PAL_CAPTURE_RATE=1.000000
```

### Optional

Here is the full list of available inventory variables. They all have sane defaults set, but may be changed so the collection can be configured to better suit your environment.

* `palworld_dedicated_server_container_name` - The name of the Palworld Dedicated Server container on the system.
    * When set to `null`, Docker Compose will automatically generate the container name.
    * Default is `null`.
* `palworld_dedicated_server_container_timezone` - Container timezone, used for timekeeping and logging.
    * Default is `Etc/UTC`.
* `palworld_dedicated_server_image_uri` - The URI for the image to install.
    * Default is [`jammsen/palworld-dedicated-server`](https://hub.docker.com/r/jammsen/palworld-dedicated-server).
    * **As this collection is designed around this image, this should not be changed.**
* `palworld_dedicated_server_image_tag` - The image tag to install.
    * Default is `latest`.
* `palworld_dedicated_server_image_pull_policy` - The standard image pull policy to set when starting the service using Docker Compose.
    * Set to `always` to always pull the latest version of the image before starting.
    * Default is `missing` (pull the latest version of the image only if it is missing).
* `palworld_dedicated_server_install_directory` - The target directory to install the compose file and service data folders to.
    * Default is `/opt/palworld-dedicated-server`.
* `palworld_dedicated_server_install_directory_owner` - Owning user of the install directory.
    * Default is `root`.
* `palworld_dedicated_server_install_directory_group` - Assigned group of the install directory.
    * You may want to set this to `docker` to give all users that can run Docker read access to the compose file.
    * Default is `root`.
* `palworld_dedicated_server_install_directory_mode` - Permissions for the install directory.
    * Default is `"0750"`.
* `palworld_dedicated_server_install_compose_file_owner` - Owning user of the compose file.
    * Default is to use the value set for the install directory.
* `palworld_dedicated_server_install_compose_file_group` - Assigned group of the compose file.
    * Default is to use the value set for the install directory.
* `palworld_dedicated_server_install_compose_file_mode` - Permissions for the compose file.
    * Default is to use the value set for the install directory.
* `palworld_dedicated_server_bind_address` - The local address to bind the Palworld Dedicated Server to.
    * *New in version 2.0.0.*
    * Default is `0.0.0.0` (bind to all interfaces).
* `palworld_dedicated_server_bind_port` - The port to expose for connecting to the Palworld Dedicated Server.
    * *Changed in version 2.0.0*: Renamed from `palworld_dedicated_server_public_port`.
    * Default is `8211`.
* `palworld_dedicated_server_rcon_enable` - When set to `true`, enable RCON port access to the Palworld Dedicated Server.
    * Default is `false`.
* `palworld_dedicated_server_rcon_bind_address` - Bind address for the RCON server.
    * Set to `0.0.0.0` to allow access from other hosts on the network.
    * Default is `127.0.0.1` (bind to `localhost`).
* `palworld_dedicated_server_rcon_bind_port` - RCON access port. Default is `25575`.
    * *Changed in version 2.0.0*: Renamed from `palworld_dedicated_server_rcon_port`.
* `palworld_dedicated_server_rcon_wait_timeout` - Timeout for waiting for the RCON port to come online after the container starts, in seconds.
    * This may need to be increased if the host's Internet connection is slow (as on the first run, the container takes a while to download Palworld after it starts).
    * Default is `600`.
* `palworld_dedicated_server_exporter_enable` - When set to `true`, orchestrate and start the Prometheus exporter for Palworld Dedicated Server.
    * *New in version 2.0.0.*
    * Default is `false`.
* `palworld_dedicated_server_exporter_image_uri` - The URI for the image to install.
    * *New in version 2.0.0.*
    * Default is [`bostrt/palworld-exporter`](https://hub.docker.com/r/bostrt/palworld-exporter).
    * **As this collection is designed around this image, this should not be changed.**
* `palworld_dedicated_server_exporter_image_tag` - The image tag to install.
    * *New in version 2.0.0.*
    * Default is `latest`.
* `palworld_dedicated_server_exporter_image_pull_policy` - The image pull policy to set for the Prometheus exporter.
    * *New in version 2.0.0.*
    * Default is to use the value of `palworld_dedicated_server_image_pull_policy`.
* `palworld_dedicated_server_exporter_container_name` - The name of the Palworld Prometheus exporter container on the system.
    * *New in version 2.0.0.*
    * When set to `null`, Docker Compose will automatically generate the container name.
    * Default is `null`.
* `palworld_dedicated_server_exporter_bind_address` - Bind address for the Prometheus exporter.
    * *New in version 2.0.0.*
    * You may want to override this to ensure it is only available on specific interfaces.
    * Default is `0.0.0.0` (bind to all interfaces).
* `palworld_dedicated_server_exporter_bind_port` - Prometheus exporter access port.
    * *New in version 2.0.0.*
    * Default is `9877`.
* `palworld_dedicated_server_exporter_wait_timeout` - Timeout for waiting for the Prometheus exporter port to come online after the container starts, in seconds.
    * *New in version 2.0.0.*
    * Default is `300`.
* `palworld_dedicated_server_service_user_enable` - Install a service user and group for Palworld Dedicated Server to use while running.
    * *New in version 2.1.0.*
    * When set to `false`, the containers are run as the UID/GID set in `palworld_dedicated_server_container_default_uid`/`palworld_dedicated_server_container_default_gid`, without an explicitly created user. This is the same behaviour as version 2.0.0 and earlier (which had the container user hard-coded to use `1000:1000`).
    * Default is `false`.
* `palworld_dedicated_server_service_user_name` - Name of the service user to create for Palworld Dedicated Server.
    * *New in version 2.1.0.*
    * Choose a unique name that is not yet used on the system.
    * Default is `palworld`.
* `palworld_dedicated_server_service_user_uid` - The unique UID to assign to the service user.
    * *New in version 2.1.0.*
    * No existing user on the system must be using the UID set in this value.
    * When set to `null`, the system will automatically assign a unique UID for the service user.
    * Default is `null`.
* `palworld_dedicated_server_service_user_additional_groups` - A list of additional group names or GIDs to assign to the service user.
    * *New in version 2.1.0.*
    * Default is to not assign any additional groups to the service user.
* `palworld_dedicated_server_service_group_name` - Name of the service group the service user is assigned to.
    * *New in version 2.1.0.*
    * Choose a unique name that is not yet used on the system.
    * Default is `palworld`.
* `palworld_dedicated_server_service_group_gid` - The unique GID to assign to the service group.
    * *New in version 2.1.0.*
    * No existing group on the system must be using the GID set in this value.
    * When set to `null`, the system will automatically assign a unique GID for the service group.
    * Default is `null`.

### How-tos

Here are a few short guides for how to configure the collection for some commonly used setups.

#### Explicitly setting the service container names

By default, Docker Compose will automatically generate the names for the service containers
(usually something along the lines of `palworld-dedicated-server_<service-name>_1`).

If you'd prefer to set the container names explicitly for each service,
you can set them using the following variables (or set them to `null` for auto-generation):

```yaml
palworld_dedicated_server_container_name: palworld-dedicated-server
palworld_dedicated_server_exporter_container_name: palworld-exporter
```

#### Enabling external access to RCON

RCON is always enabled on Palworld itself for a variety of purposes (monitoring, backup, etc), but external access to the RCON port is disabled by default.

To enable external access to the RCON port, set the following inventory variable:

```yaml
palworld_dedicated_server_rcon_enable: true
```

The external bind address and port can also be changed using the following variables (the below values are the defaults).
To allow access from other hosts on the network, set `palworld_dedicated_server_rcon_bind_address` to `0.0.0.0`.

```yaml
palworld_dedicated_server_rcon_bind_address: "127.0.0.1"
palworld_dedicated_server_rcon_bind_port: 25575
```

When external RCON access is enabled, the Ansible collection will make sure the RCON port is accessible after the containers have been started.
This operation has a default timeout of 10 minutes.
If your Palworld server has a slow Internet connection and cannot download from Steam quickly, you may need to increase this value.

```yaml
palworld_dedicated_server_rcon_wait_timeout: 600  # seconds
```

#### Enabling the Palworld Prometheus exporter

*New in version 2.0.0.*

A Prometheus exporter can optionally be orchestrated and run alongside your Palworld Dedicated Server,
allowing for metrics to be collected by Prometheus, and further processed using tools like [Grafana](https://grafana.com).

To enable the Prometheus exporter, set the following inventory variable:

```yaml
palworld_dedicated_server_exporter_enable: true
```

By default, the Prometheus exporter listens to all interfaces on port 9877. The bind address and port can be overridden using the following variables:

```yaml
palworld_dedicated_server_exporter_bind_address: "0.0.0.0"
palworld_dedicated_server_exporter_bind_port: 9877
```

#### Enabling the service user

*New in version 2.1.0.*

The Ansible collection can optionally create a service user and group for
Palworld Dedicated Server to use, configured to only have access to resources it needs,
and a no-login shell that forbids remote access.

This is more secure than the default configuration as it ensures that if any attacker
breaks into the host system through a vulnerability in Palworld,
they will be unable to do anything more than modify files owned by the service user.

Existing configurations (that do not currently use service users)
can be converted to use service users simply by enabling the option
and running the
[`callum027.palworld_dedicated_server.update`](#callum027palworld_dedicated_serverupdate)
playbook.

To enable service users, set the following inventory variable:

```yaml
palworld_dedicated_server_container_service_user_enable: true
```

By default, service users and groups have a unique UID/GID
automatically assigned to them by the system.

If you wish to explicitly set the UID/GID for the service user/group,
this may be done using the following inventory variables.

**Make sure the values you set are not used by any other users/groups on the system.**

```yaml
palworld_dedicated_server_container_service_user_uid: 300
palworld_dedicated_server_container_service_group_gid: 300
```

By default, the service user and group are creating used the name `palworld`.
These can also be changed using inventory variables.

**Make sure the values you set are not used by any other user/groups on the system.**

```yaml
palworld_dedicated_server_service_user_name: palworld
palworld_dedicated_server_service_group_name: palworld
```

#### Changing the service UID/GID without creating a service user

*New in version 2.1.0.*

To change the UID/GID that the service containers run as,
without creating a full service user and group, set the following inventory variables.

The default container UID and GID are both set to `1000` by default.

```yaml
palworld_dedicated_server_container_default_uid: 1000
palworld_dedicated_server_container_default_gid: 1000
```

## Playbooks

The collection comes with a number of convenience playbooks for managing Palworld Dedicated Server
from the command line.

To use them, make sure the hosts you wish to install Palworld Dedicated Server on
are part of the `palworld` host group in the Ansible inventory.

### `callum027.palworld_dedicated_server.install`

This playbook creates the files and directories for Palworld Dedicated Server,
starts the server, and configures it to run on startup.

```bash
ansible-playbook -i <path to inventory> callum027.palworld_dedicated_server.install
```

Once the playbook has finished running, Palworld Dedicated Server will take a while to start
as it downloads required service files from Steam. To keep an eye on this process,
run the following command on the host (as a superuser, or user that is a member of the `docker` group)
to look at the service logs:

```bash
cd /opt/palworld-dedicated-server && docker compose logs -f palworld-dedicated-server
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
cd /opt/palworld-dedicated-server && docker compose logs -f palworld-dedicated-server
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
and completely removes all containers, and files related to
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
* `callum027.palworld_dedicated_server.update`
* `callum027.palworld_dedicated_server.restart`
* `callum027.palworld_dedicated_server.stop`
* `callum027.palworld_dedicated_server.disable`
* `callum027.palworld_dedicated_server.uninstall`
