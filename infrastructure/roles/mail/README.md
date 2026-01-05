# Mail

## Sending mail to other local users

1. Run `mutt`
1. If `mutt` prompt you to create a `Mail` directory, then press `y`.
1. Press `m`
1. Enter the local user name of the recipient (`test`)
1. Enter the subject
1. Write your email, save and quit.
   `mutt` opens your `$EDITOR`, by default `vi`.
1. Review the email and press `y` to confirm delivery.

## Group mails

This playbook sets up group email addresses based on the `mail_groups` variable in the inventory.
With the following inventory:

```
mail_groups:
  foo:
    - bar
    - baz
```

, the mail system delivers mails addressed to `foo` to the `bar` and `baz` users.

The playbook creates a system, to whom the mail system delivers the mails too, creating a group mail archive.

TODO: expose the mail archives in a friendlier way.
I tried [public-inbox](https://public-inbox.org) but I did not manage to make it work and public-inbox provides little troubleshooting information.

## System mails

The `root_alias` variable in the inventory configures delivery of system emails.
