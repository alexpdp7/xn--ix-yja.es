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

## Configure NAT

Refer to [Masquerading (NAT) with iptables](https://pve.proxmox.com/pve-docs/chapter-sysadmin.html#sysadmin_network_masquerading).

Edit `/etc/network/interfaces` to make the private network on `vmbr0`.
Like the Proxmox documentation, this snippet uses the `10.10.10.0/24` network, with 256 addresses `10.10.10.0`-`10.10.10.255`.

```
# See https://pve.proxmox.com/pve-docs/chapter-sysadmin.html#sysadmin_network_masquerading
auto lo
iface lo inet loopback

auto ens18
iface ens18 inet static
	address 10.43.43.6/25
	gateway 10.43.43.1

auto vmbr0
iface vmbr0 inet static
	address  10.10.10.1/24
	bridge-ports none
	bridge-stp off
	bridge-fd 0

	post-up   echo 1 > /proc/sys/net/ipv4/ip_forward
	post-up   iptables -t nat -A POSTROUTING -s '10.10.10.0/24' -o ens18 -j MASQUERADE
	post-down iptables -t nat -D POSTROUTING -s '10.10.10.0/24' -o ens18 -j MASQUERADE

source /etc/network/interfaces.d/*
```

Reboot at this point to verify that networking on startup applies correctly.

After rebooting, verify the network configuration:

```
root@p9:~# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: ens18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether bc:24:11:6e:bf:7c brd ff:ff:ff:ff:ff:ff
    altname enp0s18
    altname enxbc24116ebf7c
    inet 10.43.43.6/25 scope global ens18
       valid_lft forever preferred_lft forever
    inet6 fe80::be24:11ff:fe6e:bf7c/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever
3: vmbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether 3e:2d:f2:57:7e:0c brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.1/24 scope global vmbr0
       valid_lft forever preferred_lft forever
    inet6 fe80::3c2d:f2ff:fe57:7e0c/64 scope link proto kernel_ll
       valid_lft forever preferred_lft forever
```

VMs and LXC containers should be able to use `10.10.10.x` addresses and connect to the Internet through Proxmox.
