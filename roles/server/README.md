# `mahadzar81.nagiosql.server`

Installs the complete NagiosQL server: MariaDB, Nagios Core, the Nagios
plugins, optionally MK Livestatus, and the NagiosQL web front-end, in the right
order.

This is the role to start with. Turn individual layers off when they are
managed elsewhere in your inventory, for example an external database.

## Role variables

| Variable | Default | Description |
| --- | --- | --- |
| `nagiosql_server_install_mariadb` | `true` | Install and provision MariaDB. |
| `nagiosql_server_install_nagios_core` | `true` | Build and install Nagios Core. |
| `nagiosql_server_install_nagios_plugins` | `true` | Build and install the plugins. |
| `nagiosql_server_install_nagiosql` | `true` | Install the NagiosQL web front-end. |
| `nagiosql_livestatus_enabled` | `false` | Also install MK Livestatus. |

Every variable from the roles it wraps applies here too.

## Dependencies

`mahadzar81.nagiosql.common`. It includes `mariadb`, `nagios_core`, `nagios_plugins`, `mk_livestatus` and `nagiosql` at run time.

## Example

```yaml
- name: Provision a Nagios Core and NagiosQL server
  hosts: nagiosql_servers
  become: true
  roles:
    - role: mahadzar81.nagiosql.server
  vars:
    nagiosql_db_password: "{{ vault_nagiosql_db_password }}"
    nagiosql_admin_password: "{{ vault_nagiosql_admin_password }}"
    nagiosql_web_users:
      - username: nagiosadmin
        password: "{{ vault_nagiosql_web_password }}"
```

## Platforms

RHEL / Rocky Linux / AlmaLinux 8, 9 and 10; Debian 11, 12 and 13; Ubuntu 22.04
and 24.04.

> RHEL 8 ships Python 3.6 and must be managed with `ansible-core` 2.16. See the
> [collection README](../../README.md) for the full explanation.

## Licence

MIT. See the [collection README](../../README.md) for the complete variable
reference and architecture notes.
