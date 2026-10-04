# `mahadzar81.nagiosql.mariadb`

Installs MariaDB and provisions the NagiosQL database and its user.

By default the `root` account keeps its distribution default (local socket
authentication) and Ansible connects over the socket, rather than rewriting the
root password on every run.

## Role variables

| Variable | Default | Description |
| --- | --- | --- |
| `nagiosql_mariadb_state` | `started` | Service state. |
| `nagiosql_mariadb_enabled` | `true` | Enable the service at boot. |
| `nagiosql_mariadb_manage_root_password` | `false` | Set `true` to manage the root password and write `~/.my.cnf`. |
| `nagiosql_mariadb_root_password` | `""` | Only used when the above is `true`. |
| `nagiosql_db_name`, `nagiosql_db_user`, `nagiosql_db_password` | see `common` | Database provisioned by this role. |

## Dependencies

`mahadzar81.nagiosql.common`

## Example

```yaml
- name: Provision the NagiosQL database
  hosts: nagiosql_servers
  become: true
  roles:
    - role: mahadzar81.nagiosql.mariadb
  vars:
    nagiosql_db_password: "{{ vault_nagiosql_db_password }}"
```

## Platforms

RHEL / Rocky Linux / AlmaLinux 8, 9 and 10; Debian 11, 12 and 13; Ubuntu 22.04
and 24.04.

> RHEL 8 ships Python 3.6 and must be managed with `ansible-core` 2.16. See the
> [collection README](../../README.md) for the full explanation.

## Licence

MIT. See the [collection README](../../README.md) for the complete variable
reference and architecture notes.
