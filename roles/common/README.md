# `mahadzar81.nagiosql.common`

Shared foundation for every other role in the collection. It resolves the
operating-system specific variables, asserts that the platform is supported,
configures the package repositories, and records whether SELinux is active.

Every other role declares this one as a `meta` dependency, so you rarely run it
directly. Its defaults are the collection's shared variable surface: paths, the
service account, Apache and MariaDB settings, and the NagiosQL database
connection details all live here, because several roles need them.

## Role variables

| Variable | Default | Description |
| --- | --- | --- |
| `nagiosql_prefix` | `/usr/local/nagios` | Nagios Core installation prefix. |
| `nagiosql_object_dir` | `/usr/local/nagiosql` | Where NagiosQL writes generated object configuration. |
| `nagiosql_build_dir` | `/usr/local/src/nagiosql` | Scratch space for source tarballs and build trees. |
| `nagiosql_user` / `nagiosql_group` | `nagios` | Service account. |
| `nagiosql_command_group` | `nagcmd` | Group shared with the Apache user for the command pipe. |
| `nagiosql_db_name` | `db_nagiosql_v350` | Derived from `nagiosql_version`. |
| `nagiosql_db_user` | `nagiosql_user` | NagiosQL database user. |
| `nagiosql_db_password` | `changeme` | **Override this.** Use Ansible Vault. |
| `nagiosql_db_encoding` | `utf8` | Must match the bundled NagiosQL schema. |
| `nagiosql_manage_epel` | `true` | Installs `epel-release` on the EL family. |
| `nagiosql_manage_crb` | `true` | Enables CodeReady Builder / PowerTools. |
| `nagiosql_crb_repo` | `crb`, `powertools` on EL 8 | Repository id to enable. |
| `nagiosql_selinux_manage` | `true` | Set `false` to leave SELinux alone. |
| `nagiosql_extra_packages` | `[]` | Anything extra you want installed. |

Package lists, Apache paths and MariaDB socket locations are resolved per
platform from `vars/` and exposed as `nagiosql_*_packages`,
`nagiosql_apache_*` and `nagiosql_mariadb_*`.

## Dependencies

None. This is the base role.

## Example

```yaml
- name: Resolve the platform facts only
  hosts: nagiosql_servers
  become: true
  roles:
    - role: mahadzar81.nagiosql.common
```

## Platforms

RHEL / Rocky Linux / AlmaLinux 8, 9 and 10; Debian 11, 12 and 13; Ubuntu 22.04
and 24.04.

> RHEL 8 ships Python 3.6 and must be managed with `ansible-core` 2.16. See the
> [collection README](../../README.md) for the full explanation.

## Licence

MIT. See the [collection README](../../README.md) for the complete variable
reference and architecture notes.
