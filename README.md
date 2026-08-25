# Ansible Role: helm

Install helm from offical deb repository on Debian GNU/Linux.

## Requirements
The role is designed for [Debian GNU/Linux](https://debian.org).

## Role Variables

| variable | default |
|---|---|
| `helm_version` | empty, meaning the newest release in the repository |
| `helm_diff_version` | `3.15.11` |
| `helm_diff_arch` | derived from `ansible_architecture` |

The repository at `packages.buildkite.com/helm-linux/helm-debian` carries
several helm majors side by side (`3.21.3-1` and `4.2.3-1` at the time of
writing), so an unset `helm_version` means the installed major depends on the
day the machine was built. Set it to a Debian package version to decide:

```
helm_version: 4.2.3-1
```

That also writes `/etc/apt/preferences.d/helm.pref` with priority 1001, so a
later `apt upgrade` cannot move helm to another version, and a downgrade back
to the pinned one is allowed. Emptying `helm_version` removes the pin again.

Pinning matters because helm removes CLI flags between majors and Ansible's
`kubernetes.core` collection has to match: Helm 4 dropped `helm list --all`,
which `kubernetes.core` sends up to and including 6.3.0, so Helm 4 needs at
least 6.4.0 - 6.5.0 for the `helm` module itself. Note that Helm 3 has its
final feature release on 2026-09-09 and receives security fixes only until
2027-02-10.

The role also installs the [helm-diff](https://github.com/databus23/helm-diff)
plugin. Helm 4 verifies plugin signatures by default and refuses sources that
cannot carry one, so the git URL that worked under Helm 3 now fails with
`plugin source does not support verification`. The plugin is therefore
installed from the signed release tarball of `helm_diff_version`, verified
against the release signing key, which is fetched to
`/etc/helm/helm-diff-pubkey.asc` and dearmoured to
`/etc/helm/helm-diff-keyring.gpg` because helm reads binary keyrings only.
An already installed plugin is left alone, so raising `helm_diff_version` on a
machine that has the plugin does not move it - uninstall with
`helm plugin uninstall diff` first.

## Example Playbook
```
- hosts: localhost
  connection: local

  roles:
    - role: helm
```
