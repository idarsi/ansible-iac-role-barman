# Contributing

Keep the role state-driven and idempotent. Use `state: validate` for changes
to the `iac_blueprint.barman` inventory shape and update the README and example
inventory whenever user-visible fields or states change.

The role is an independent repository under `engine/playbooks/roles/`.
Changes to the parent `engine/` repository are limited to an intentional
gitlink update after the role repository has been reviewed.

The `tasks/shared/` path is a pinned dependency checkout. Manage shared task
branches and releases in the canonical shared-tasks repository, then update
this submodule pointer separately when a newer shared revision is required.

Run these checks from the role directory:

```bash
ansible-playbook --syntax-check \
  -i docs/inventory-example.yml \
  docs/playbook-example.yml
ANSIBLE_ROLES_PATH=.. ansible-lint --profile production
```
