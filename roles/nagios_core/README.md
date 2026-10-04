# `mahadzar81.nagiosql.nagios_core`

Builds and installs Nagios Core from source, configures Apache and PHP in front
of it, and manages the CGI web users.

The build is guarded by a version stamp file, so re-runs are a no-op and
raising `nagiosql_core_version` triggers a genuine rebuild. SELinux booleans
and file contexts are applied on the EL family when SELinux is enforcing.

## Role variables

| Variable | Default | Description |
| --- | --- | --- |
| `nagiosql_core_version` | `4.5.14` | Nagios Core release to build. |
| `nagiosql_core_url` | assets.nagios.com | Source tarball URL. |
| `nagiosql_core_checksum` | `""` | Optional bare sha256 digest to verify the download. |
| `nagiosql_core_configure_extra_args` | `""` | Extra flags appended to `./configure`. |
| `nagiosql_core_service_state` | `started` | Service state. |
| `nagiosql_core_service_enabled` | `true` | Enable at boot. |
| `nagiosql_web_users` | one `nagiosadmin` entry | List of `{username, password}` for CGI basic auth. **Override and vault these.** |
| `nagiosql_htpasswd_file` | `<prefix>/etc/htpasswd.users` | Basic auth file. |
| `nagiosql_htpasswd_crypt_scheme` | `apr_md5_crypt` | Hashing scheme. |

## Dependencies

`mahadzar81.nagiosql.common`

## Example

```yaml
- name: Install Nagios Core
  hosts: nagiosql_servers
  become: true
  roles:
    - role: mahadzar81.nagiosql.nagios_core
  vars:
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
