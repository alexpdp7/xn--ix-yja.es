# Infrastructure

## Using SSH

Add:

```
Include .../xn--ix-yja.es/infrastructure/ssh_config
```

to your `~/.ssh/config` before any `Host` statement.

## Bootstrapping

Ensure you can `ssh root@ñix.es`, and then:

```
uv run ansible-playbook -i ssh-root.yaml -i production.yaml site.yaml
```

You might need to `systemctl restart apache2` for the Let's Encrypt certificate.

## Running Ansible

After bootstrapping, you can run Ansible via ssh with your user and `sudo`.
After [setting your user password](roles/users/README.md):

```
uv run ansible-playbook -K -i production.yaml site.yaml
```

Or run from the same host with:

```
uv run ansible-playbook -K -i local-exec.yaml -i production.yaml site.yaml
```

## Roles

* The [users](roles/users) role to add new users
* The [git](roles/git) role to host Git repositories (https and Gitweb)
* The [vaultwarden](roles/vaultwarden) role for secret management

## Testing

With Incus installed, edit `local_incus.hosts.ñix.es.ansible_incus_project` in `incus-test-local.yaml` with your project from `incus project list`.
Then, you can use the following command to create a test VM on Incus and deploy:

```
uv run ansible-playbook -i production.yaml -i incus-test-local.yaml site.yaml
```

### Accessing

1. Obtain the VM IP address by running `incus list`.
1. Run `ssh -D 1080 $IP`.
1. Configure your browser to use a SOCKS v5 proxy on `localhost:1080` and to proxy DNS.

## Notes

### Contabo Debian 13

* `unattended-upgrades` configured by default
