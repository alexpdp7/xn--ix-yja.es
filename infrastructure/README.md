# Infrastructure

## Bootstrapping

Ensure you can `ssh root@ñix.es`, and then:

```
uv run ansible-playbook -i ssh-root.yaml -i production.yaml site.yaml
```

## Running Ansible

After bootstrapping, you can run Ansible via ssh with your user and `sudo`.
After [setting your user password](roles/users/README.md):

```
uv run ansible-playbook -K -i production.yaml site.yaml
```

## Notes

### Contabo Debian 13

* `unattended-upgrades` configured by default
