# Ansible Collection: `mahadzar81.nagiosql`

[![CI](https://github.com/mahadzar81/ansible-collection-nagiosql/actions/workflows/ci.yml/badge.svg)](https://github.com/mahadzar81/ansible-collection-nagiosql/actions/workflows/ci.yml)

Deploys **Nagios Core** together with **NagiosQL**, the web based configuration
front-end, on RHEL-family and Debian-family systems. Nagios Core and the Nagios
plugins are built from source; MariaDB, Apache and PHP come from distribution
packages.

This collection replaces the standalone `mahadzar81.nagiosql` role. See
[CHANGELOG.md](CHANGELOG.md) for what changed and what was removed.

---

## Supported platforms

| Family | Versions | SELinux |
| --- | --- | --- |
| RHEL / Rocky Linux / AlmaLinux | 8, 9, 10 | Supported and configured |
| Debian | 11 (bullseye), 12 (bookworm), 13 (trixie) | Not applicable |
| Ubuntu | 22.04 (jammy), 24.04 (noble) | Not applicable |

Anything outside this list fails fast with an explicit message from the
`common` role rather than part-way through a source build.

## Requirements

- `ansible-core` 2.16 or newer on the controller.
- `become: true` — the collection installs packages, builds software into
  `/usr/local` and manages systemd units.
- Collection dependencies, installed with `ansible-galaxy collection install -r requirements.yml`:
  - `ansible.posix` >= 1.5.0
  - `ansible.mysql` >= 1.0.0 (`community.mysql` was renamed; the old FQCNs still
    redirect but are deprecated)
  - `community.general` >= 8.0.0
- Outbound HTTPS from the managed host to `assets.nagios.com`,
  `nagios-plugins.org` and `sourceforge.net`.

NagiosQL needs the PHP `mysqli`, `gettext`, `mbstring`, `xml`, `session` and
`ftp` extensions, plus PEAR. The first five come from `php-common` on both
families; PEAR is installed explicitly as `php-pear`, because NagiosQL bundles
`HTML/Template/IT.php` but not the `PEAR.php` base class it requires. All of
this is handled by `nagiosql_php_packages`.

## Installation

```bash
ansible-galaxy collection install mahadzar81.nagiosql
```

Or from a `requirements.yml`:

```yaml
---
collections:
  - name: mahadzar81.nagiosql
    version: ">=2.0.0"
```

## Quick start

```yaml
---
- name: Provision a Nagios Core and NagiosQL server
  hosts: nagiosql_servers
  become: true
  gather_facts: true
  vars:
    nagiosql_db_password: "{{ vault_nagiosql_db_password }}"
    nagiosql_admin_password: "{{ vault_nagiosql_admin_password }}"
    nagiosql_web_users:
      - username: nagiosadmin
        password: "{{ vault_nagiosql_web_password }}"
  roles:
    - role: mahadzar81.nagiosql.server
```

A ready-made playbook ships with the collection:

```bash
ansible-playbook -i inventory mahadzar81.nagiosql.nagiosql
```

Once it finishes:

- Nagios Core — `http://<host>/nagios/` (basic auth, `nagiosql_web_users`)
- NagiosQL — `http://<host>/webadmin/` (form login, `nagiosql_admin_user`)

> **Change the passwords.** Every credential default in this collection is
> `changeme`. Put the real values in Ansible Vault.

## Roles

| Role | Purpose |
| --- | --- |
| `mahadzar81.nagiosql.server` | Meta role that installs the whole stack. Start here. |
| `mahadzar81.nagiosql.common` | OS variable resolution, platform assertion, EPEL/CRB, SELinux detection. Every other role depends on it. |
| `mahadzar81.nagiosql.mariadb` | MariaDB server, NagiosQL database and database user. |
| `mahadzar81.nagiosql.nagios_core` | Builds Nagios Core, configures Apache and PHP, manages CGI web users. |
| `mahadzar81.nagiosql.nagios_plugins` | Builds the official Nagios plugins. |
| `mahadzar81.nagiosql.mk_livestatus` | Optional MK Livestatus broker module. Off by default — see below. |
| `mahadzar81.nagiosql.nagiosql` | NagiosQL web front-end, `nagios.cfg` generation, database bootstrap. |

Use them individually when part of the stack is managed elsewhere:

```yaml
roles:
  - role: mahadzar81.nagiosql.nagios_core   # external MariaDB, no web UI
```

Or keep the `server` role and switch layers off:

```yaml
vars:
  nagiosql_server_install_mariadb: false
```

## Variables

Every variable is prefixed `nagiosql_`. Only the ones you are likely to touch
are listed here; the full set is in each role's `defaults/main.yml`.

### Credentials

| Variable | Default | Description |
| --- | --- | --- |
| `nagiosql_db_password` | `changeme` | Password for the NagiosQL database user. |
| `nagiosql_admin_password` | `changeme` | NagiosQL web UI administrator password. |
| `nagiosql_admin_password_hash` | MD5 of the above | Supply directly if your NagiosQL release hashes differently. |
| `nagiosql_web_users` | one `nagiosadmin` entry | List of `{username, password}` for the Nagios CGI basic auth. |
| `nagiosql_mariadb_manage_root_password` | `false` | When `false`, `root` keeps distribution default socket auth and Ansible connects over the local socket. |
| `nagiosql_mariadb_root_password` | `""` | Only used when the above is `true`. |

### Versions

| Variable | Default |
| --- | --- |
| `nagiosql_core_version` | `4.5.14` |
| `nagiosql_plugins_version` | `2.4.12` |
| `nagiosql_version` | `3.5.0` |
| `nagiosql_livestatus_version` | `1.5.0p25` |

Each has a matching `_url` and an optional `_checksum` (a bare sha256 digest;
leave empty to skip verification).

### Layout

| Variable | Default | Description |
| --- | --- | --- |
| `nagiosql_prefix` | `/usr/local/nagios` | Nagios Core installation prefix. |
| `nagiosql_object_dir` | `/usr/local/nagiosql` | Where NagiosQL writes generated object configuration. |
| `nagiosql_web_dir` | `<docroot>/webadmin` | NagiosQL document root. |
| `nagiosql_build_dir` | `/usr/local/src/nagiosql` | Source tarballs and build trees. |
| `nagiosql_user` / `nagiosql_group` | `nagios` | Service account. |
| `nagiosql_command_group` | `nagcmd` | Group shared with the Apache user for the command pipe. |

### `nagios.cfg` generation

The `nagiosql` role owns `nagios.cfg` and renders it from
`nagiosql_cfg_files`, `nagiosql_cfg_dirs` and `nagiosql_broker_modules`.

The file is written in place, then checked with `nagios -v`. If the check
fails, the previous `nagios.cfg` is restored from a backup and the run fails,
so a bad template never reaches a running service. Ansible's `validate:`
parameter cannot be used here: `nagios -v` drops privileges to the `nagios`
user after parsing the main config file and before reading the object config,
which leaves it unable to re-open a temporary file under root's `0700` tmp
directory.

| Variable | Default | Description |
| --- | --- | --- |
| `nagiosql_manage_nagios_cfg` | `true` | Set `false` to leave `nagios.cfg` alone entirely. |
| `nagiosql_validate_nagios_cfg` | `true` | Set `false` to skip the `nagios -v` pre-flight check and its rollback. |
| `nagiosql_broker_modules` | Livestatus when enabled, else `[]` | Event broker modules. |
| `nagiosql_process_performance_data` | `false` | Enables the perfdata file directives. |

### SELinux

| Variable | Default | Description |
| --- | --- | --- |
| `nagiosql_selinux_manage` | `true` | Set `false` to leave SELinux untouched. |
| `nagiosql_selinux_booleans` | `httpd_can_network_connect`, `httpd_can_network_connect_db` | Booleans set persistently. |

### Repositories

| Variable | Default | Description |
| --- | --- | --- |
| `nagiosql_manage_epel` | `true` | Installs `epel-release` on the EL family. |
| `nagiosql_manage_crb` | `true` | Enables CodeReady Builder / PowerTools. |
| `nagiosql_crb_repo` | `crb`, or `powertools` on EL 8 | Override on RHEL proper, where the repo id is `codeready-builder-for-rhel-<ver>-<arch>-rpms`. |
| `nagiosql_dnf_plugins_package` | `dnf-plugins-core`, or `dnf5-plugins` on EL 10 | Package providing `dnf config-manager`. |

## SELinux behaviour

SELinux work is gated on `ansible_facts.selinux.status == 'enabled'`, so the
tasks are skipped cleanly on Debian and Ubuntu and on EL hosts running with
SELinux disabled. On an enforcing EL host the collection:

1. Installs `policycoreutils-python-utils` and the libselinux Python bindings.
2. Sets `httpd_can_network_connect` and `httpd_can_network_connect_db`
   persistently with `ansible.posix.seboolean`.
3. Registers file contexts with `community.general.sefcontext`:

   | Path | Type |
   | --- | --- |
   | `/usr/local/nagios(/.*)?` | `httpd_sys_content_t` |
   | `/usr/local/nagios/sbin(/.*)?` | `httpd_sys_script_exec_t` |
   | `/usr/local/nagios/libexec(/.*)?` | `httpd_sys_script_exec_t` |
   | `/usr/local/nagios/var(/.*)?` | `httpd_sys_rw_content_t` |
   | `/usr/local/nagios/etc(/.*)?` | `httpd_sys_rw_content_t` |
   | `/var/www/html/webadmin(/.*)?` | `httpd_sys_rw_content_t` |
   | `/usr/local/nagiosql(/.*)?` | `httpd_sys_rw_content_t` |

4. Relabels with `restorecon -RFv`, reporting `changed` only when files were
   actually relabelled.

Because contexts are written to the policy store rather than applied only to
inodes, they survive a full filesystem relabel.

## Idempotency

The collection is idempotent, and the Molecule sequence enforces it with an
`idempotence` stage on all eight platforms. The mechanisms:

- **Source builds are version-stamped.** Each build writes
  `<prefix>/.ansible-nagios-core-<version>` and is skipped when that file
  exists. Bumping the version variable invalidates the stamp and triggers a
  genuine rebuild, which a `creates=` guard on the installed binary could not do.
- **Packages use `state: present`,** never `latest`.
- **`command` replaces `shell`,** always with `chdir` and `creates`.
- **The database bootstrap queries the database.** The schema import is gated on
  `SHOW TABLES LIKE 'tbl_settings'` returning nothing, not on some earlier task's
  `changed` flag. The seed tables `DROP` and recreate, so they are imported only
  on first initialisation unless `nagiosql_db_force_reinit: true`.
- **`restorecon` reports honestly** via `changed_when` on its verbose output.
- **The apt cache refresh** is `changed_when: false`.

## MK Livestatus

MK Livestatus is **disabled by default**. Version 1.5.0p25 is a 2019 C++ code
base that does not compile cleanly under GCC 12 and newer, which rules out
Debian 12/13, Ubuntu 24.04 and EL 9/10 in practice. Where it does build:

```yaml
vars:
  nagiosql_livestatus_enabled: true
  nagiosql_livestatus_tcp_enabled: true   # optional TCP listener
```

Enabling it automatically appends the broker module to the generated
`nagios.cfg`. TCP access uses systemd socket activation
(`livestatus.socket` plus `livestatus@.service`) rather than xinetd, which EL 9+
and Debian 13 no longer ship. `nagiosql_livestatus_cxxflags` relaxes the C++
standard and is your first knob if the build fails.

## Testing

```bash
ansible-galaxy collection install -r requirements.yml
pip install 'molecule>=6.0' 'molecule-plugins[docker]' docker

cd extensions
MOLECULE_DISTRO=rockylinux9 molecule test
```

Valid `MOLECULE_DISTRO` values: `rockylinux8`, `rockylinux9`, `rockylinux10`,
`debian11`, `debian12`, `debian13`, `ubuntu2204`, `ubuntu2404`.

The scenario runs `dependency → destroy → syntax → create → prepare → converge
→ idempotence → verify → destroy`. Verification asserts that Nagios, Apache and
MariaDB are running, that the built binaries and plugins exist, that
`nagios -v` accepts the generated configuration, that both web interfaces
respond, and that the NagiosQL schema was imported.

The Molecule plays run with `become: false`, because the test containers
already run as root and `sudo` is broken in the Rocky Linux base images on
docker-ce and GitHub Actions runners
([sig-cloud-instance-images#56](https://github.com/rocky-linux/sig-cloud-instance-images/issues/56),
open upstream). This affects the test harness only; the collection itself is
written to run under `become: true`, as `playbooks/nagiosql.yml` does.

**SELinux cannot be tested in Docker.** Containers share the host's SELinux
state, so the CI matrix exercises the "SELinux disabled" path on Rocky. Verify
the enforcing path on a real VM.

### Continuous integration

`.github/workflows/ci.yml` runs on push, pull request and a weekly schedule:

1. **Lint** — `yamllint` plus `ansible-lint` at the `production` profile.
2. **Build** — `ansible-galaxy collection build`, uploaded as an artifact.
3. **Molecule** — the full 8-platform matrix, `fail-fast: false`.

## Releasing

`.github/workflows/release.yml` fires on any `v*` tag and:

1. Asserts the tag matches `version:` in `galaxy.yml`, failing the run otherwise.
2. Builds the collection tarball.
3. Creates a GitHub release with generated notes and the tarball attached.
4. Publishes to Ansible Galaxy.

One-time setup: create a token at <https://galaxy.ansible.com/ui/token/> and
store it as the repository secret **`GALAXY_API_KEY`**.

To cut a release:

```bash
# bump `version:` in galaxy.yml, update CHANGELOG.md, commit
git tag -a v2.0.1 -m "Release 2.0.1"
git push origin v2.0.1
```

## Notes and caveats

- **NagiosQL 3.5.0 is a beta.** It is nonetheless the first release that runs on
  PHP 8, which every supported distribution now ships. On PHP 8.2+ (Debian 13,
  Ubuntu 24.04, EL 10) expect deprecation notices in the Apache error log.
- **The password hash format** in `tbl_user` is MD5, matching NagiosQL 3.4 and
  3.5. If a future release changes the scheme, set
  `nagiosql_admin_password_hash` explicitly or complete the setup through
  `/webadmin/install/`.
- **The web installer is removed** after the first successful database
  bootstrap. Set `nagiosql_db_force_reinit: true` to keep it.
- **Upgrading Nagios Core** is a matter of raising `nagiosql_core_version`. The
  stamp file mechanism rebuilds and reinstalls; the collection does not migrate
  object configuration, which lives in the database.

## Licence

MIT.

## Author

Originally `mahadzar81/ansible-role-nagiosql`, restructured as a collection.
