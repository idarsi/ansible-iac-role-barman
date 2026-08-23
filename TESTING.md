# Testing

The current test surface is a syntax check and production-profile Ansible
Lint. The supported platform contract is Red Hat Enterprise Linux and Rocky
Linux 8, 9, and 10; these platforms are not currently covered by automated
Molecule scenarios. Molecule scenarios should be added before enabling
package-installation CI, because repository access and Barman package
availability must be tested on each supported Enterprise Linux version.

The validation-only state can be exercised without package installation:

```bash
ansible-playbook --check --diff \
  -i docs/inventory-example.yml \
  docs/playbook-example.yml -e state=validate
```

```bash
ansible-playbook --syntax-check \
  -i docs/inventory-example.yml \
  docs/playbook-example.yml
ANSIBLE_ROLES_PATH=.. ansible-lint --profile production
```
