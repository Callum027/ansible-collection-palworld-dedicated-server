# Palworld Dedicated Server Ansible Collection Release Notes

**Topics**
- <a href="#v2-1-0">v2\.1\.0</a>
  - <a href="#release-summary">Release Summary</a>
  - <a href="#major-changes">Major Changes</a>
- <a href="#v2-0-0">v2\.0\.0</a>
  - <a href="#release-summary-1">Release Summary</a>
  - <a href="#major-changes-1">Major Changes</a>
  - <a href="#minor-changes">Minor Changes</a>
  - <a href="#breaking-changes--porting-guide">Breaking Changes / Porting Guide</a>
  - <a href="#bugfixes">Bugfixes</a>
- <a href="#v1-0-1">v1\.0\.1</a>
  - <a href="#release-summary-2">Release Summary</a>
- <a href="#v1-0-0">v1\.0\.0</a>
  - <a href="#release-summary-3">Release Summary</a>

<a id="v2-1-0"></a>
## v2\.1\.0

<a id="release-summary"></a>
### Release Summary

This release adds support for configuring Palworld Dedicated Server to run with service users and groups\.

This is more secure than the default configuration because it ensures that Palworld
is run using a dedicated system user \(with its own UID and GID\)\, configured
to only have access to resources it needs\, and a no\-login shell that forbids remote access\.
If any attacker breaks into the host system through a vulnerability in Palworld\,
they will be unable to do anything more than modify files owned by the service user\.

This is an opt\-in feature that can be enabled by setting <code>palworld\_dedicated\_server\_service\_user\_enable</code> to <code>true</code>\.

If full service users are not desired in your setup\, the collection also now supports configuring the container
to run using a custom UID/GID without creating a service user and group\.
Just set the <code>palworld\_dedicated\_server\_container\_default\_uid</code> <code>palworld\_dedicated\_server\_container\_default\_gid</code>
inventory variables to the desired values \(both set to <code>1000</code> by default for backwards compatibility\)\.

Existing installations can be converted to run with service users or a custom UID/GID by changing the collection settings\,
and running the <code>callum027\.palworld\_dedicated\_server\.update</code> playbook to apply them \(as a container update is required\)\.

For more information\, refer to the collection documentation\.

<a id="major-changes"></a>
### Major Changes

* Add support for creating and configuring Palworld to use service users and groups\, and changing permissions on the data files and directories if the configured UID/GID changes\.
* Add support for setting a custom UID/GID for the service containers without creating a service user\.

<a id="v2-0-0"></a>
## v2\.0\.0

<a id="release-summary-1"></a>
### Release Summary

This release adds support for orchestrating a Prometheus exporter alongside the Palworld Dedicated Server\.

Enable it by setting <code>palworld\_dedicated\_server\_exporter\_enable</code> to <code>true</code>\, and configure the bind IP address and port with <code>palworld\_dedicated\_server\_exporter\_bind\_address</code> \(defaults to <code>0\.0\.0\.0</code>\) and <code>palworld\_dedicated\_server\_exporter\_bind\_port</code> \(defaults to <code>9877</code>\)\.

This is a major version upgrade as unfortunately a number of breaking changes were required in this release\. In the future the necessity of major version releases like this should decrease\.

<a id="major-changes-1"></a>
### Major Changes

* Add support for orchestrating the [bostrt/palworld\-exporter](https\://hub\.docker\.com/r/bostrt/palworld\-exporter) Prometheus exporter\, [developed by palworld\.lol](https\://github\.com/palworldlol/palworld\-exporter)\, alongside Palworld Dedicated Server\.

<a id="minor-changes"></a>
### Minor Changes

* The Palworld Dedicated Server\'s bind address for external access is now configurable using <code>palworld\_dedicated\_server\_exporter\_bind\_address</code>\, which defaults to <code>0\.0\.0\.0</code>\.

<a id="breaking-changes--porting-guide"></a>
### Breaking Changes / Porting Guide

* The Palworld server admin password is now set in the <code>palworld\_dedicated\_server\_admin\_password</code> inventory variable\. It can no longer be defined by setting <code>ADMIN\_PASSWORD</code> in <code>palworld\_dedicated\_server\_settings</code> \(this value will now be ignored\)\.
* The <code>palworld\_dedicated\_server\_public\_port</code> inventory variable has been renamed to <code>palworld\_dedicated\_server\_bind\_port</code>\.
* The <code>palworld\_dedicated\_server\_rcon\_port</code> inventory variable has been renamed to <code>palworld\_dedicated\_server\_rcon\_bind\_port</code>\.

<a id="bugfixes"></a>
### Bugfixes

* Fix syntax errors in the compose file causing the service start to error out when RCON is enabled\.

<a id="v1-0-1"></a>
## v1\.0\.1

<a id="release-summary-2"></a>
### Release Summary

This release contains updates to the documentation\, giving more detail about the container
used by the collection\, and configuring it with Palworld game settings\.
The role README file titles have also been fixed to refer to the correct respective roles\.

No functional changes have been made\.

<a id="v1-0-0"></a>
## v1\.0\.0

<a id="release-summary-3"></a>
### Release Summary

Initial release of the Ansible collection for Palworld Dedicated Server\.
