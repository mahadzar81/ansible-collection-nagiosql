# `mahadzar81.nagiosql.nagiosql`

Installs the NagiosQL web configuration front-end, generates `nagios.cfg`, and
bootstraps the database schema.

`nagios.cfg` is written in place and then checked with `nagios -v`; if the check
fails the previous file is restored from a backup and the run fails, so an
invalid configuration never reaches a running service.

The schema import is gated on the database itself rather than on an earlier
task's `changed` flag, so re-runs do not re-import and wipe your data.

## Role variables

| Variable | Default | Description |
| --- | --- | --- |
| `nagiosql_version` | `3.5.0` | NagiosQL release (defined in `common`). |
| `nagiosql_url` | SourceForge | Source tarball URL. |
| `nagiosql_web_dir` | `<docroot>/webadmin` | NagiosQL document root. |
| `nagiosql_admin_user` | `admin` | NagiosQL administrator. |
| `nagiosql_admin_password` | `changeme` | **Override this.** Use Ansible Vault. |
| `nagiosql_admin_password_hash` | MD5 of the above | Supply directly if your release hashes differently. |
| `nagiosql_manage_nagios_cfg` | `true` | Set `false` to leave `nagios.cfg` alone. |
| `nagiosql_validate_nagios_cfg` | `true` | Run the `nagios -v` pre-flight check and rollback. |
| `nagiosql_cfg_files` / `nagiosql_cfg_dirs` | see defaults | Object configuration included by `nagios.cfg`. |
| `nagiosql_broker_modules` | Livestatus when enabled | Event broker modules. |
| `nagiosql_process_performance_data` | `false` | Enables the perfdata directives. |
| `nagiosql_db_force_reinit` | `false` | Re-import the schema, **dropping** the settings, user and config-target tables. |

NagiosQL needs the PHP `mysqli`, `gettext`, `mbstring`, `xml`, `session` and
`ftp` extensions plus PEAR. All are handled by `nagiosql_php_packages`.

## Dependencies

`mahadzar81.nagiosql.common`, and in practice `nagios_core` and `mariadb`, which must have run first.

## Example

```yaml
- name: Install the NagiosQL web front-end
  hosts: nagiosql_servers
  become: true
  roles:
    - role: mahadzar81.nagiosql.nagiosql
  vars:
    nagiosql_admin_password: "{{ vault_nagiosql_admin_password }}"
```

## Platforms

RHEL / Rocky Linux / AlmaLinux 8, 9 and 10; Debian 11, 12 and 13; Ubuntu 22.04
and 24.04.

> RHEL 8 ships Python 3.6 and must be managed with `ansible-core` 2.16. See the
> [collection README](../../README.md) for the full explanation.

## Licence

MIT. See the [collection README](../../README.md) for the complete variable
reference and architecture notes.
