# Users

## Initial set up

This role creates users in the `sudo` group.
`sudo` requires a password.

To set your password, run `su -c "passwd $(whoami)"`.

From then on, use `sudo` for privileged actions.
Remember to only modify the system using Ansible.
