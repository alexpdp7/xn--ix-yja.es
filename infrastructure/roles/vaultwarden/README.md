# Vaultwarden

## Registering to Vaultwarden

Registration is limited to the `@localhost` domain so that only local users can register.
Registration requires email validation.

Visit `/vaultwarden`, select "create account", then use `$USER@localhost` as your email address.

1. Run `mutt`.
1. If this is the first execution of `mutt`, press y to create the mail directory.
1. Locate the "verify your email" message.
1. Press enter to open the email.
1. Press ctrl+b to open the URL viewer.
1. In the URL viewer, you can copy the long verification URL without line breaks.
1. Press q and any key to exit URL viewer.
1. Press d to delete the "verify your email" message.
1. Press d to delete the "welcome" message.
1. Press d to delete the "new device" message.
1. Press q and y to exit and purge deleted messages.

## Security

[The Bitwarden Security Whitepaper](https://bitwarden.com/help/bitwarden-security-white-paper/) says that Bitwarden clients, such as the browser extension, never pass the master password that can decrypt passwords to the Bitwarden server.
Note that root on the system can tamper with the Vaultwarden web vault, but the browser extensions are controlled by Bitwarden.

Therefore, we recommend changing the master password *before* entering any sensitive data in Vaultwarden, to ensure that the password cannot be snooped by root on the system.
