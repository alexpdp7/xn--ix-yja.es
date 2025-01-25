# Setup

## Domain

[Gandi is one of the supported registrars for YunoHost](https://doc.yunohost.org/en/providers/registrar).

* Buy the domain from Gandi.
* Visit https://admin.gandi.net/
* Click on your user in the top right of the page, then "User Settings".
* Click "Create a token" in the "Personal Access Token (PAT)" section.
* Select the organization that matches your user name.
* Fill the form:
  * Token name: "yunohost domain automation"
  * Activate "Manage domain name technical configurations"
* Copy the token for later use.

## Server

### Initial setup

* Get a "VPS 1 SSD" from Contabo, with Debian 12.
* Disable VNC from the "new" Contabo control panel.
* Log in as `root` via SSH.
* Run the YunoHost installer command.
  The installation might fail because `cloud-init` installs software on the first boot of the server, you can retry.
  Accept SSH configuration changes.
* Follow the link that the installer prints to access YunoHost.
* Follow the wizard.
  * "I want to add a domain I own", type the domain *in punycode if needed* (e.g. `example.com`).
  * Create your user, store the credentials.
* After being redirected to the login page, login might not behave correctly.
  If this happens, then edit the URL and remove everything after the IP address.

### Domain setup

* Log in to the YunoHost admin interface at https://yourdomain/yunohost/admin/
* https://github.com/YunoHost/issues/issues/2405 Gandi instructions are outdated, updating the DNS records automatically is not currently possible without patching.

### Reverse DNS

This seems to be possible on the old Contabo control panel.

* Visit https://my.contabo.com/
* Click "Reverse DNS Management"
* Edit the IPv4 and IPv6 records.
  Use the punycode version if needed.

### Diagnosis

* Run the internal diagnosis as per the YunoHost installation instructions.
  If you followed the previous steps, then everything is green.

### Set up a certificate

* Log in to the YunoHost admin interface at https://yourdomain/yunohost/admin/
* "Domains", select your domain, "Certificate"
* "Install Let's Encrypt certificate"

## TODO

* The default Contabo Debian unattended-upgrades configuration does not update everything, because YunoHost adds more apt sources.
  Check if the unattended-upgrades *YunoHost* application updates everything OOB.

## Notes

* After initial setup, `root` password might only be useful on the VNC console for recovery?
* For a non-ASCII domain name, I had to enter it in the installer as punycode, or the installer entered an infinite loop.
* After installing YunoHost, `fail2ban` is enabled.

### Unicode issues

* Outlook for Android does not allow sending emails to non-UTF8 domains.
