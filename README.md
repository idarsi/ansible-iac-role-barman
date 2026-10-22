ANSIBLE-IAC-ROLE-BARMAN
=======================
> **Maturity State: Beta**<br>
> **RC Readiness: 84%**<br>
> Assessment includes passing runtime convergence, idempotence, and verification on Rocky Linux 9/10, the validation matrix, and Ansible Lint.<br>
> Test evidence is from the working tree and remains pending commit and CI.

**COPYRIGHT** 2026 ^(ida|arsi)$ collective<br>
**LICENSE** MIT License [LICENSE](LICENSE)<br>
**AUTHORS**
- Arsi Atomi <arsi@atomi.sh>

Overview
========

This Ansible role installs and configures [Barman](https://www.pgbarman.org/)
on Red Hat Enterprise Linux and Rocky Linux hosts. Barman packages are
installed using the PostgreSQL Global Development Group (PGDG) repository.

The role is deliberately separate from the PostgreSQL server role. It manages
the Barman host and Barman server definitions; PostgreSQL connection users,
`pg_hba.conf`, WAL archiving, and replication prerequisites remain PostgreSQL
responsibilities.

The role uses the `iac_blueprint` model to keep the desired Barman state in a
structured inventory while Ansible manages the Barman host and its server
definitions. It is intended to provide an independent backup-management layer
that can be deployed after the PostgreSQL role has prepared the database-side
connection and replication prerequisites.

> Automated validation and package/configuration runtime scenarios cover Rocky
> Linux 9 and 10 using Podman; runtime checks include idempotence and rendered
> configuration. CI is configured, but results on this revision are pending.
> End-to-end backup/restore workflows and RHEL runtime evidence remain gaps.
> Review destructive data removal behavior carefully before production use.

## Supported states

| State | Operation |
|---|---|
| `install` | Install packages and configure Barman and all declared servers |
| `uninstall` | Remove packages, configuration, and Barman data directories; requires `barman_confirm_data_removal: true` |
| `present` | Ensure packages and global/server configuration are present |
| `absent` | Remove packages and managed configuration while preserving Barman data directories |
| `servers_present` | Ensure declared server configuration files are present |
| `servers_absent` | Remove declared server configuration files |
| `validate` | Validate the inventory without changing the host |

Uninstall supports only the canonical Barman paths (`/etc/barman.d`,
`/var/log/barman`, `/var/lib/barman`, `/var/run/barman`, and
`/etc/barman.conf`). Path defaults may be customized for other operations, but
uninstall rejects noncanonical assignments, including paths swapped between
fields. Its component symlink checks reduce unsafe traversal risk but do not
eliminate time-of-check/time-of-use races; avoid concurrent filesystem changes
during uninstall.

## Quick start

```yaml
---
- hosts: barman
  become: true
  roles:
    - role: ansible-iac-role-barman
      state: install
  vars:
    iac_blueprint:
      barman:
        servers:
          - name: primary
            conninfo: "host=db01.example.org user=barman dbname=postgres"
            ssh_command: "ssh postgres@db01.example.org"
            retention_policy: "RECOVERY WINDOW OF 30 DAYS"
```

See [docs/inventory-example.yml](docs/inventory-example.yml) for a complete
inventory example.

The `iac_blueprint.barman` mapping accepts optional `global_config` and a
`servers` list. Each server requires a unique `name` and at least one of
`conninfo` or `ssh_command`; the remaining server fields are rendered to the
server configuration file when provided. Use `state: validate` to check this
input contract without installing packages or changing files.

## Requirements

- Ansible 2.15 or newer
- Red Hat Enterprise Linux or Rocky Linux 9 or 10
- Access to the PGDG repository and the distribution base repositories
- PostgreSQL-side connection, authentication, and WAL settings configured
  separately

The role does not create PostgreSQL users or modify PostgreSQL hosts. Run the
PostgreSQL role first when the Barman server definition depends on resources
that it creates.

## Shared tasks

The repository consumes `ansible-iac-shared-tasks` as a submodule under
`tasks/shared/`. Barman-specific wrappers use the shared filesystem and task
report logging implementations so common behavior remains consistent with the
other Engine roles.
