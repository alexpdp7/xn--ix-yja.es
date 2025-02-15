# Instant messaging

## IRC

Ergo installs OK, but because websockets are not configured, I was only able to get TheLounge to connect without much additional configuration.

Even then, the out of the box experience is not awesome.

## Zulip

(Testing without YunoHost, although there is a [WIP package that is not on the catalog yet](https://github.com/YunoHost-Apps/zulip_ynh).)

* Installation fails if you use a Unicode domain, you must use punycode for the domain name.
