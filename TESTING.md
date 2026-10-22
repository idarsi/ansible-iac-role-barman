# Testing

The automated matrix targets Rocky Linux 9 and 10 and Red Hat UBI 9 and 10.
Podman systemd containers cover validation-only behavior on all four targets;
the package/configuration runtime scenario runs on Rocky 9 and 10. Runtime
includes an idempotence pass and verifies package presence and rendered
configuration. Container results do not establish complete RHEL support or
real backup/restore behavior.

Scenarios:

- `validation`: validates a representative blueprint and checks that validation
  did not install Barman or create its global configuration.
- `runtime`: installs from PGDG, configures a server, checks rendered output,
  and requires a clean idempotence pass.

Run with the approved `idarsi-ansible-test-environments` setup and its selected
Ansible environment, after installing the collections from
`molecule/requirements.yml`:

```bash
molecule test -s validation
molecule test -s runtime
```

Static checks:

```bash
ansible-playbook --syntax-check -i docs/inventory-example.yml docs/playbook-example.yml
ANSIBLE_ROLES_PATH=.. ansible-lint --profile production
```

Runtime requires outbound access to PGDG and distribution repositories. The
Molecule tests do not run backup/restore, manipulate real database servers, or
use the destructive `uninstall` state. CI runs the `validation` and `runtime`
scenarios for pull requests and pushes to `main`. No KVM/libvirt scenario is
configured.
