# `mahadzar81.nagiosql.nagios_plugins`

Builds and installs the official Nagios plugins from source into
`<nagiosql_prefix>/libexec`.

Like Nagios Core, the build is guarded by a version stamp file. `configure`
detects OpenSSL on its own; do not pass a bare `--with-openssl`, which autoconf
expands to the literal path `yes/lib`.

## Role variables

| Variable | Default | Description |
| --- | --- | --- |
| `nagiosql_plugins_version` | `2.4.12` | Plugin release to build. |
| `nagiosql_plugins_url` | nagios-plugins.org | Source tarball URL. |
| `nagiosql_plugins_checksum` | `""` | Optional bare sha256 digest. |
| `nagiosql_plugins_configure_extra_args` | `""` | Extra flags for `./configure`. |

`check_smb` additionally needs the `smbclient` binary at runtime, which is not
installed by default. Add `samba-client` (EL) or `smbclient` (Debian) through
`nagiosql_extra_packages` if you need it.

## Dependencies

`mahadzar81.nagiosql.common`

## Example

```yaml
- name: Install the Nagios plugins
  hosts: nagiosql_servers
  become: true
  roles:
    - role: mahadzar81.nagiosql.nagios_plugins
```

## Platforms

RHEL / Rocky Linux / AlmaLinux 8, 9 and 10; Debian 11, 12 and 13; Ubuntu 22.04
and 24.04.

> RHEL 8 ships Python 3.6 and must be managed with `ansible-core` 2.16. See the
> [collection README](../../README.md) for the full explanation.

## Licence

MIT. See the [collection README](../../README.md) for the complete variable
reference and architecture notes.
