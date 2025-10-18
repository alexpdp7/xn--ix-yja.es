# Proxmox notes

This document documents Proxmox networking running on a dedicated server with a single IPv4 address.

This process uses [`proxmox-ve_9.0-1.iso`](https://enterprise.proxmox.com/iso/proxmox-ve_9.0-1.iso).
The process is developed and tested as a VM on another Proxmox host.

## Initial setup

Via `ssh root@`.

### [Configure the no-subscription repository](https://pve.proxmox.com/pve-docs/chapter-sysadmin.html#sysadmin_no_subscription_repo)

```
# cat >/etc/apt/sources.list.d/proxmox.sources
Types: deb
URIs: http://download.proxmox.com/debian/pve
Suites: trixie
Components: pve-no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
^D
```

### Update and reboot

```
# apt update
# apt full-upgrade
# shutdown -r now
```

## Initial network configuration

The installer creates `/etc/network/interfaces`:

```
auto lo
iface lo inet loopback

iface ens18 inet manual

auto vmbr0
iface vmbr0 inet static
        address 10.43.43.6/25
        gateway 10.43.43.1
        bridge-ports ens18
        bridge-stp off
        bridge-fd 0


source /etc/network/interfaces.d/*
```

; `10.43.43.6` is the address in the internal network of the parent Proxmox host.
`10.43.43.1` is the address of the parent Proxmox host that acts as the gateway.
`ens18` is the virtual network interface of the Proxmox VM.
