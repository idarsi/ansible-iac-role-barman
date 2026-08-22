# Testing

The initial test surface is a syntax check and production-profile Ansible
Lint. Molecule scenarios should be added before enabling package-installation
CI, because repository access and Barman package availability must be tested
on each supported Enterprise Linux version.

```bash
ansible-playbook --syntax-check \
  -i docs/inventory-example.yml \
  docs/playbook-example.yml
ANSIBLE_ROLES_PATH=.. ansible-lint --profile production
```
