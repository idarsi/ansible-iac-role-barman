ANSIBLE-IAC-ROLE-BARMAN
=======================
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

> **Maturity Level: Experimental**<br>
> The initial package, repository, configuration, inventory-validation, and
> shared-task wrapper functionality is implemented. The role is currently
> moving from an initial integration prototype toward a production-ready
> backup-management component. Molecule and CI coverage, end-to-end backup and
> restore verification, scheduling and WAL-archiving workflows, and broader
> operating-system coverage still need to be established. Use it for
> development and controlled integration testing, and review its behavior
> carefully before using it for unattended production backups.

## Supported states

| State | Operation |
|---|---|
| `install` | Install packages and configure Barman and all declared servers |
| `uninstall` | Remove packages, configuration, and Barman data directories |
| `present` | Ensure packages and global/server configuration are present |
| `absent` | Remove packages and managed configuration |
| `servers_present` | Ensure declared server configuration files are present |
| `servers_absent` | Remove declared server configuration files |
| `validate` | Validate the inventory without changing the host |

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

## Requirements

- Ansible 2.15 or newer
- Red Hat Enterprise Linux or Rocky Linux 8, 9, or 10
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
