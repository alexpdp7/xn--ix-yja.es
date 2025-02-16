# Instant messaging

## IRC

Ergo installs OK, but because websockets are not configured, I was only able to get TheLounge to connect without much additional configuration.

Even then, the out of the box experience is not awesome.

## Zulip

(Testing without YunoHost, although there is a [WIP package that is not on the catalog yet](https://github.com/YunoHost-Apps/zulip_ynh).)

* Installation fails if you use a Unicode domain, you must use punycode for the domain name. <https://github.com/zulip/zulip/pull/33502>
* Mail support can be a bit fiddly with host names, the system host name must match the server name.
* The terminal client worked on my self-hosted Zulip instance, although it did not work on the public instance.
* Email integration is nice:
  * You can use Markdown in emails and Zulip processes the mail correctly.
  * Zulip waits for a few messages before sending a new email, and collects multiple messages in a single email.
* However, by default emailing a channel is not nice.
  You must get the channel obfuscated email address from the Zulip web interface.

Zulip could be a great option because:

* You can use the web interface, terminal interface, email interface.
* Third-party clients should be allowed?
* You can switch from their free/paid services to self-hosted plans easily.
