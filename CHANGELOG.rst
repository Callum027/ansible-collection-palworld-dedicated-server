==========================================================
Palworld Dedicated Server Ansible Collection Release Notes
==========================================================

.. contents:: Topics


v2.0.0
======

Release Summary
---------------

This release adds support for orchestrating a Prometheus exporter alongside the Palworld Dedicated Server.

Enable it by setting ``palworld_dedicated_server_exporter_enable`` to ``true``, and configure the bind IP address and port with ``palworld_dedicated_server_exporter_bind_address`` (defaults to ``0.0.0.0``) and ``palworld_dedicated_server_exporter_bind_port`` (defaults to ``9877``).

This is a major version upgrade as unfortunately a number of breaking changes were required in this release. In the future the necessity of major version releases like this should decrease.


Major Changes
-------------

- Add support for orchestrating the `bostrt/palworld-exporter <https://hub.docker.com/r/bostrt/palworld-exporter>`_ Prometheus exporter, `developed by palworld.lol <https://github.com/palworldlol/palworld-exporter>`_, alongside Palworld Dedicated Server.

Minor Changes
-------------

- The Palworld Dedicated Server's bind address for external access is now configurable using ``palworld_dedicated_server_exporter_bind_address``, which defaults to ``0.0.0.0``.

Breaking Changes / Porting Guide
--------------------------------

- The Palworld server admin password is now set in the ``palworld_dedicated_server_admin_password`` inventory variable. It can no longer be defined by setting ``ADMIN_PASSWORD`` in ``palworld_dedicated_server_settings`` (this value will now be ignored).
- The ``palworld_dedicated_server_public_port`` inventory variable has been renamed to ``palworld_dedicated_server_bind_port``.
- The ``palworld_dedicated_server_rcon_port`` inventory variable has been renamed to ``palworld_dedicated_server_rcon_bind_port``.

Bugfixes
--------

- Fix syntax errors in the compose file causing the service start to error out when RCON is enabled.

v1.0.1
======

Release Summary
---------------

This release contains updates to the documentation, giving more detail about the container
used by the collection, and configuring it with Palworld game settings.
The role README file titles have also been fixed to refer to the correct respective roles.

No functional changes have been made.


v1.0.0
======

Release Summary
---------------

Initial release of the Ansible collection for Palworld Dedicated Server.
