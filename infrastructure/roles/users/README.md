# Users

## Creating users

This role creates users described in the [production](../../production.yaml) inventory.
Change this file to create new users.

This role creates users in the `sudo` group.
`sudo` requires a password.

Once you can ssh to your user, set your password with `su -c "passwd $(whoami)"`.

From then on, use `sudo` for privileged actions.
Remember to only modify the system using Ansible.
