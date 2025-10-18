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

	# By default, the Proxmox firewall is disabled. If you intend to enable the firewall, then include the following lines from the Proxmox documentation linked above:
	post-up   iptables -t raw -I PREROUTING -i fwbr+ -j CT --zone 1
	post-down iptables -t raw -D PREROUTING -i fwbr+ -j CT --zone 1

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

## Configure dnsmasq

dnsmasq is a simple to configure DHCP/DNS integrated server.

```
root@p9:~# apt install dnsmasq
```

`/etc/dnsmasq.conf` contains configuration documentation.
By default, `/etc/default/dnsmasq` configures dnsmasq to include configuration files in `/etc/dnsmasq.d`, to leave `dnsmasq.conf` untouched.

Create `/etc/dnsmasq.d/internal`:

```
domain-needed
no-resolv
no-hosts

server=10.43.43.1  # your upstream DNS server

local=/p9net.example.com/
domain=p9net.example.com

dhcp-range=10.10.10.64,10.10.10.126,255.255.255.0,255.255.255.255,48h
dhcp-option=option:router,10.10.10.1
```

This allocates 63 addresses in the `10.10.10.64`-`10.10.10.126` for automatic VM and LXC host addresses, leaving you other ranges for other purposes.

Machines using DHCP get host names like `p9net.example.com` that cannot be used in public DNS.
If you have a domain `foo.com`, you can use a subdomain `x.y.z.foo.com`.

Edit `/etc/resolv.conf` so that the Proxmox machine uses dnsmasq and the internal domain for DNS:

```
domain p9net.example.com
search p9net.example.com
nameserver 10.10.10.1
```

By default, VMs and LXC containers get their DNS configuration from `/etc/resolv.conf` on the Proxmox host.
Therefore, use the `10.10.10.1` IP address of the Proxmox host in `/etc/resolv.conf`, so that VMs and LXC containers get a configuration that works for them.
(For example, if you use `127.0.0.1`, then VMs and LXC containers try to resolve DNS names using `127.0.0.1`, which does not work.)

Reboot to verify that everything applies correctly.
Verify DNS configuration by running `host some.domain.you.know`.

### LXC test

Create an LXC container with the web interface:

* Hostname: `lxc.p9net.example.com`
* Template: `debian-13-standard`
* IPv4: DHCP

After the container starts:

* Run `apt full-upgrade -U` to update.
  This verifies that DNS and Internet work.
* Run `ip a` to verify that you get an IP in the DHCP range.
* Run `ssh root@lxc` on the Proxmox host to verify that DNS resolution in Proxmox works.
  (By default, the Debian 13 template disables root password logins.)

### VM test

Download a live system, such as [`debian-live-13.1.0-amd64-gnome.iso`](https://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/debian-live-13.1.0-amd64-gnome.iso) to the Proxmox ISO repository.

* Name: `vm.p9net.example.com`
* ISO image: `debian-live-13.1.0-amd64-gnome.iso`

When the live image boots:

* Use Firefox to verify that DNS and Internet work.
* Run `ssh root@lxc` to verify that you can connect to other hosts in the Proxmox network.

## Firewall

Open an SSH connection to the Proxmox host to ensure you can revert changes.
If something goes wrong, edit `/etc/pve/firewall/cluster.fw` to change enable to 0.
Changes to this file apply automatically immediately.

### Examining configuration

Use `iptables-save` to list firewall rules.
Use `ipset list` to list IP sets.

By default, when you enable the Proxmox firewall, many rules appear automatically.
These include rules for the [standard IP set management](https://pve.proxmox.com/pve-docs/chapter-pve-firewall.html#_standard_ip_set_span_class_monospaced_management_span) to allow management of Proxmox:

```
# iptables-save | grep PVEFW-0-management
-A PVEFW-HOST-IN -p tcp -m set --match-set PVEFW-0-management-v4 src -m tcp --dport 8006 -j RETURN
-A PVEFW-HOST-IN -p tcp -m set --match-set PVEFW-0-management-v4 src -m tcp --dport 5900:5999 -j RETURN
-A PVEFW-HOST-IN -p tcp -m set --match-set PVEFW-0-management-v4 src -m tcp --dport 3128 -j RETURN
-A PVEFW-HOST-IN -p tcp -m set --match-set PVEFW-0-management-v4 src -m tcp --dport 22 -j RETURN
-A PVEFW-HOST-IN -p tcp -m set --match-set PVEFW-0-management-v4 src -m tcp --dport 60000:60050 -j RETURN
```

```
# ipset list
Name: PVEFW-0-management-v6
Type: hash:net
Revision: 7
Header: family inet6 hashsize 64 maxelem 64 bucketsize 12 initval 0x4d7ac321
Size in memory: 1240
References: 5
Number of entries: 0
Members:

Name: PVEFW-0-management-v4
Type: hash:net
Revision: 7
Header: family inet hashsize 64 maxelem 64 bucketsize 12 initval 0xd0331705
Size in memory: 504
References: 5
Number of entries: 1
Members:
10.43.43.0/25
```

In the host used to develop this document, `10.43.43.0/25` is the network of the parent Proxmox host.
In this case, when enabling the firewall, only management traffic from the `10.43.43.0/25` network is allowed.

Proxmox does not seem to allow configuring IP sets that allow any address; `0.0.0.0/0` and other variants are rejected.
Therefore, if your Proxmox host network interface has a public IPv4 address, then likely you cannot use the default management rules to allow management from any host on the Internet.

### Configuring the firewall

If you configure NAT, then notice that the Proxmox documentation about [Masquerading (NAT) with iptables](https://pve.proxmox.com/pve-docs/chapter-sysadmin.html#sysadmin_network_masquerading) includes the following rules in the interface configuration:

```
	post-up   iptables -t raw -I PREROUTING -i fwbr+ -j CT --zone 1
	post-down iptables -t raw -D PREROUTING -i fwbr+ -j CT --zone 1
```

In my tests, these rules were required, otherwise Proxmox does not route VM and LXC traffic to the Internet.

Additionally, if your VMs and LXC hosts use DHCP/DNS from dnsmasq, then you need to allow traffic from their network to the Proxmox host.

For example, you can create an IP set `internal` for `10.10.10.0/24` and a rule that accepts all traffic from this IP set.
