# 🔐 Cybersecurity Lab: Network Setup on Kali Linux

**Building a private NAT Network in VirtualBox, assigning a persistent static IP to Kali Linux, verifying connectivity and DNS, and saving a clean snapshot.**

![Skill](https://img.shields.io/badge/Skill-Cybersecurity-404040?style=flat-square&labelColor=C00000)
![VirtualBox](https://img.shields.io/badge/VirtualBox-Virtualization-0070C0?style=flat-square&labelColor=000000)
![Kali Linux](https://img.shields.io/badge/OS-Kali%20Linux-E87500?style=flat-square&labelColor=000000&logo=kalilinux&logoColor=white)
![Network](https://img.shields.io/badge/Network-10.0.0.0%2F24-238F89?style=flat-square&labelColor=000000)

---

## 📌 Overview

As part of Week 1 of my cybersecurity internship at NetworkWalks, I set up the networking for a Kali Linux virtual machine in VirtualBox.

The project covers creating a private NAT Network with DHCP disabled, assigning a static IP address, testing Internet and DNS access, fixing a configuration that disappeared after a reboot, and taking a clean snapshot for recovery.

## 🎯 Objectives

- Create a private NAT Network in VirtualBox
- Connect the Kali Linux VM to that network
- Configure a static IPv4 address manually (DHCP disabled)
- Verify routing, Internet access and DNS resolution
- Make the configuration permanent with NetworkManager
- Create a clean snapshot as a recovery point

## ⚙️ Lab Configuration

| Component | Configuration |
|---|---|
| Host OS | Windows 11 |
| Hypervisor | Oracle VirtualBox `[7.2.22]` |
| Guest OS | Kali Linux 
| Network Type | NAT Network |
| Network Name | `NatNetwork` |
| Network Address | `10.0.0.0/24` |
| DHCP | Disabled |
| Kali IP Address | `10.0.0.10/24` |
| Gateway | `10.0.0.1` |
| DNS Server | `8.8.8.8` |

## 🪜 Implementation

### Step 1: Existing Kali Linux Environment

VirtualBox and the Kali Linux VM were set up and used as the base for this lab.

![Kali in VirtualBox](screenshots/01-virtualbox-kali.png)

### Step 2: Create a NAT Network

Created through **File → Tools → Network Manager → NAT Networks**.

- **Name:** `NatNetwork`
- **IPv4 Prefix:** `10.0.0.0/24`
- **DHCP:** Disabled (addresses are assigned manually)
- **IPv6:** Disabled

![NAT Network Settings](screenshots/02-nat-network.png)

### Step 3: Connect Kali to the NAT Network

In the VM settings, **Adapter 1** is set to *NAT Network* and the name `NatNetwork` is selected.

![Adapter Settings](screenshots/03-adapter-settings.png)

### Step 4: Configure a Static IP Address

Settings applied to `eth0`:

- **IP Address:** `10.0.0.10`
- **Subnet Mask:** `255.255.255.0` (`/24`)
- **Gateway:** `10.0.0.1`
- **DNS Server:** `8.8.8.8`

Created as a NetworkManager profile so it is applied automatically at every boot:

```bash
sudo nmcli connection add type ethernet ifname eth0 con-name "NatNetwork-static" \
  ipv4.method manual ipv4.addresses 10.0.0.10/24 \
  ipv4.gateway 10.0.0.1 ipv4.dns 8.8.8.8 connection.autoconnect yes
sudo nmcli connection up "NatNetwork-static"
```

![Static IP Configuration](screenshots/04-ip-config.png)

### Step 5: Verify Network Connectivity

```bash
ip a show eth0
ip route
ping -c 4 8.8.8.8
ping -c 4 google.com
```

- `ip a` shows `inet 10.0.0.10/24` on `eth0`
- `ip route` shows `default via 10.0.0.1 dev eth0 proto static`
- Both pings returned 4/4 replies with 0% packet loss, confirming Internet access and DNS resolution

![Connectivity Tests](screenshots/05-ping-tests.png)

### Step 6: Troubleshoot the Lost Configuration

**Problem:** After the VM and the host were restarted, `ping 8.8.8.8` returned *Network is unreachable* and `eth0` had no IP address. Earlier, adding a default route also failed with *Nexthop has invalid gateway*.

**Cause:** The address had first been added with `ip addr add`, which is temporary and is lost at reboot. The gateway error appeared because the route was added before the address was in place.

**Solution:** Removed the old profile and created a permanent NetworkManager profile (Step 4), then confirmed the IP and Internet access were still working.

```bash
nmcli connection show
sudo nmcli connection delete "Wired connection 1"
sudo nmcli connection add type ethernet ifname eth0 con-name "NatNetwork-static" ...
sudo nmcli connection up "NatNetwork-static"
```

> **Note:** Connection names differ between systems, and Linux commands are case-sensitive. Check the exact name with `nmcli connection show`.

### Step 7: Create a Clean Snapshot

Created a VirtualBox snapshot named `Clean-kali-week1`, taken once the network and DNS tests were passing.

![Snapshot](screenshots/06-snapshot.png)



## 💡 What I Learned

- **NAT vs NAT Network:** a NAT Network lets several VMs talk to each other and reach the Internet through a private, isolated range.
- **Static IP configuration:** address, netmask, gateway and DNS must all be consistent, and the address must exist before a route through the gateway can be added.
- **Persistence:** `ip addr add` is temporary; NetworkManager (`nmcli`) keeps settings across reboots.
- **Troubleshooting:** `ip a`, `ip route`, `ping` and `nmcli` narrow down where a network problem is.
- **Snapshots:** they provide a safe recovery point before further experiments.

## 🔐 Ethical Use

This lab is for educational purposes. Any future penetration testing will only be done on systems I own or am explicitly authorized to test.

---

**Program:** NetworkWalks Cybersecurity Internship
**Week:** 01
**Project:** Cybersecurity and Pentesting Lab Setup
**Author:** AMAL CHEBBI 
