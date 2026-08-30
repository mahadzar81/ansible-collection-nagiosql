# Changelog

All notable changes to this collection are documented in this file. The format
follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and the project
adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.0.0] - 2026-08-17

The `mahadzar81.ansible-role-nagiosql` standalone role has been restructured as
the `mahadzar81.nagiosql` collection. This is a breaking change: role names,
variable names and the supported platform list have all changed.

### Added

- Six composable roles - `common`, `mariadb`, `nagios_core`, `nagios_plugins`,
  `mk_livestatus` and `nagiosql` - plus a `server` role that wires them together.
- Support for RHEL/Rocky/AlmaLinux 8, 9 and 10, Debian 11, 12 and 13, and
  Ubuntu 22.04 and 24.04.
- SELinux support for the EL family: booleans, file contexts through
  `community.general.sefcontext`, and change-aware `restorecon` runs.
- Livestatus TCP exposure through systemd socket activation, replacing the
  xinetd configuration that several supported distributions no longer ship.
- A Molecule scenario under `extensions/molecule/default` covering all eight
  supported platforms, including an idempotence stage.
- GitHub Actions workflows for lint plus Molecule, and for tag-driven GitHub
  releases and Ansible Galaxy publishing.
- `ansible.builtin.assert` guard that fails early on unsupported platforms.

### Changed

- Every variable is now prefixed with `nagiosql_`.
- Every module is referenced by its fully qualified collection name.
- Nagios Core defaults to 4.5.14, the plugins to 2.4.12 and NagiosQL to 3.5.0,
  the first NagiosQL release that runs on PHP 8.
- MariaDB authentication defaults to local socket auth for `root` rather than
  rewriting the root password on every run.
- `nagios.cfg` is generated from a template driven by `nagiosql_cfg_files`,
  `nagiosql_cfg_dirs` and `nagiosql_broker_modules`, then checked with
  `nagios -v` and rolled back from a backup if the check fails.
- Source builds are guarded by version stamp files, so re-runs are no-ops and
  version bumps trigger a rebuild.
- `community.mysql` FQCNs were migrated to `ansible.mysql`.

### Removed

- CentOS 6/7, Debian 9/10 and Ubuntu 18.04/20.04 support.
- The `mod_gearman` and `ndoutils` broker modules that the old `nagios.cfg`
  hard-coded without ever installing them.
- The xinetd based Livestatus service definition.

### Fixed

- The `state: latest` package installs that made every run report changes.
- The backup-before-upgrade tasks that read from the controller rather than the
  managed host.
- The `replace` task that rewrote `nagios.cfg` immediately before a template
  overwrote it.
- Database imports that were driven by `changed` flags rather than by the state
  of the database itself.
