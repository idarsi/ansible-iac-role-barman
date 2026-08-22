# ansible-iac-role-barman

This Ansible role installs and configures [Barman](https://www.pgbarman.org/)
on Red Hat Enterprise Linux and Rocky Linux hosts. Barman packages are
installed from the PostgreSQL Global Development Group (PGDG) repository.

The role is deliberately separate from the PostgreSQL server role. It manages
the Barman host and Barman server definitions; PostgreSQL connection users,
`pg_hba.conf`, WAL archiving, and replication prerequisites remain PostgreSQL
responsibilities.

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
