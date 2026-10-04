# `mahadzar81.nagiosql.mk_livestatus`

Optional MK Livestatus broker module for Nagios Core.

**Disabled by default.** Version 1.5.0p25 is a 2019 C++ code base that does not
compile cleanly under GCC 12 and newer, which rules out Debian 12/13,
Ubuntu 24.04 and EL 9/10 in practice. Enable it only where you know it builds.

TCP access uses systemd socket activation rather than xinetd, which EL 9+ and
Debian 13 no longer ship.

## Role variables

| Variable | Default | Description |
| --- | --- | --- |
| `nagiosql_livestatus_enabled` | `false` | Master switch (defined in `common`). |
| `nagiosql_livestatus_version` | `1.5.0p25` | Release to build. |
| `nagiosql_livestatus_cxxflags` | `-std=gnu++14 -fpermissive -w` | First knob to try if the build fails. |
| `nagiosql_livestatus_tcp_enabled` | `false` | Expose Livestatus over TCP. |
| `nagiosql_livestatus_tcp_port` | `6557` | Listening port. |

Enabling this automatically appends the broker module to the generated
`nagios.cfg`.

## Dependencies

`mahadzar81.nagiosql.common`

## Example

```yaml
- name: Install MK Livestatus
  hosts: nagiosql_servers
  become: true
  roles:
    - role: mahadzar81.nagiosql.mk_livestatus
  vars:
    nagiosql_livestatus_enabled: true
    nagiosql_livestatus_tcp_enabled: true
```

## Platforms

RHEL / Rocky Linux / AlmaLinux 8, 9 and 10; Debian 11, 12 and 13; Ubuntu 22.04
and 24.04.

> RHEL 8 ships Python 3.6 and must be managed with `ansible-core` 2.16. See the
> [collection README](../../README.md) for the full explanation.

## Licence

MIT. See the [collection README](../../README.md) for the complete variable
reference and architecture notes.
