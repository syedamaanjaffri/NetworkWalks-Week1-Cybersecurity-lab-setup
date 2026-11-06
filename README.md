<div align="center">

# 🔐 Cybersecurity Lab Environment Setup

**An isolated virtual lab built with VirtualBox and Kali Linux for ethical hacking practice**

*Week 1 | Project Module 1 (WK1-PM1) — Cybersecurity Internship at Networkwalks*

</div>

---

## 📌 Overview

This project documents how I built a **virtual cybersecurity and penetration-testing lab** on my laptop using **Oracle VirtualBox** and **Kali Linux**.

The lab gives me a safe, isolated environment to practice reconnaissance, scanning and vulnerability assessment without touching real-world systems. It runs on a private virtual network, so more machines (such as vulnerable targets for CTF practice) can be added in later weeks.

> ⚠️ **Ethical use:** This lab is for education only. Security tools must only be used on systems I own or have explicit permission to test.

---

## 🎯 Objectives

- Install the latest VirtualBox as the hypervisor.
- Import the latest Kali Linux VM as the attacker machine.
- Create a custom **NAT Network** using the `10.0.0.0/24` subnet.
- Assign the static IP `10.0.0.2/24` to Kali Linux.
- Enable clipboard sharing, drag & drop and a shared folder (host `Downloads`).
- Confirm full Internet access from Kali.
- Take a clean snapshot for recovery.

---

## 🏗️ Lab Architecture

```text
┌─────────────────────────────────────────────────────────────┐
│  Host Machine (Windows)                                     │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │  Oracle VirtualBox                                  │   │
│   │                                                     │   │
│   │   NAT Network: "NatNetwork"  (10.0.0.0/24)          │   │
│   │   ┌──────────────────────┐   ┌───────────────────┐  │   │
│   │   │  Kali Linux (VM)     │   │ Future target VMs │  │   │
│   │   │  10.0.0.2/24         │   │ 10.0.0.3 – .99    │  │   │
│   │   └──────────────────────┘   └───────────────────┘  │   │
│   │            Gateway: 10.0.0.1  ──►  Internet         │   │
│   └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## ⚙️ Lab Configuration

| Component | Configuration |
| --- | --- |
| 🧰 Hypervisor | Oracle VirtualBox 7.x |
| 🐉 Attacker OS | Kali Linux 2026.2 (amd64) |
| 🧠 Kali RAM | 2048 MB |
| 🌐 Virtual Network | NAT Network (`NatNetwork`) |
| 📡 Network Address | 10.0.0.0/24 (DHCP enabled, IPv6 disabled) |
| 🐧 Kali IP Address | 10.0.0.2/24 (static) |
| 🚪 Default Gateway | 10.0.0.1 |
| 🌍 DNS Server | 8.8.8.8 |
| 📁 Shared Folder | Host `Downloads` folder (Full access, Auto-mount) |
| 🔮 Reserved for Future VMs | 10.0.0.3 – 10.0.0.99 |

---

## 🪜 Setup Procedure

### Step 1. Install 7-Zip
The Kali VirtualBox image comes as a `.7z` archive, so I installed 7-Zip to extract it.
🔗 https://7-zip.org/download.html

### Step 2. Install VirtualBox
I installed the latest version of VirtualBox on my Windows machine.
🔗 https://virtualbox.org/wiki/Downloads

### Step 3. Create the NAT Network
In VirtualBox: **File → Tools → Network → NAT Networks → Create**

| Setting | Value |
| --- | --- |
| Name | `NatNetwork` |
| IPv4 Prefix | `10.0.0.0/24` |
| Enable DHCP | ✅ Yes |
| Enable IPv6 | ❌ No |

**Why a NAT Network and not plain NAT?** With plain NAT every VM is isolated. A NAT Network lets VMs on the same network talk to each other *and* reach the Internet, which is exactly what an attacker + target lab needs.

### Step 4. Download and Import Kali Linux
I downloaded the pre-built Kali VirtualBox image from the official site, extracted it with 7-Zip and opened it in VirtualBox.
🔗 https://kali.org/get-kali

Network adapter (`Settings → Network → Adapter 1`):

```text
Attached to:   NAT Network
Name:          NatNetwork
Adapter Type:  Intel PRO/1000 MT Desktop (82540EM)
```

Memory (`Settings → System → Motherboard`):

```text
Base Memory: 2048 MB
```

### Step 5. Enable Clipboard, Drag & Drop and Shared Folder

Clipboard and drag & drop (`Settings → General → Advanced`):

```text
Shared Clipboard: Bidirectional
Drag'n'Drop:      Bidirectional
```

Shared folder (`Settings → Shared Folders → Add`):

```text
Folder Path:  Downloads folder of the host machine
Access:       Full
Auto-mount:   Enabled
```

This makes it easy to move files between Windows and Kali.

### Step 6. Configure a Static IP in Kali Linux
Inside Kali: **Network icon → Edit Connections → Wired connection 1 → IPv4 Settings**

```text
Method:   Manual
Address:  10.0.0.2
Netmask:  24
Gateway:  10.0.0.1
DNS:      8.8.8.8
```

> 💡 If the Internet does not work with `8.8.8.8`, try `10.0.0.1` as the DNS server.

### Step 7. Take a Clean Snapshot
After confirming everything worked, I powered off the VM and took a snapshot (`Ctrl + Shift + T`):

```text
Snapshot name: My Fresh Kali Linux after installation
```

If a future experiment breaks the VM, I can restore this clean baseline.

---

## 🔎 Verification

| Test | Command | Expected Result |
| --- | --- | --- |
| Check IP address | `ip a` | `10.0.0.2/24` shown |
| Test gateway | `ping -c 4 10.0.0.1` | Successful replies |
| Test Internet | `ping -c 4 8.8.8.8` | Successful replies |
| Test DNS | `nslookup kali.org` | Domain resolves |
| Verify Nmap | `nmap --version` | Version displayed |

---

## 🐞 Troubleshooting: No Internet After Setting a Static IP

This is a known issue with VirtualBox 7 and Kali Linux 2026.1 or newer. NetworkManager's duplicate address detection can delay the connection. The fix is:

```bash
sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```

Then restart Kali and test again. Connection names can differ between systems, so run `nmcli connection show` first to check yours.

If the Internet still fails, also check that the NAT Network exists, the IP settings are correct, no other VM on the network uses `10.0.0.2`, and try restarting the VMs and the host.

---

## 💡 What I Learned

- **NAT vs NAT Network:** how each one affects communication between VMs and the Internet.
- **Virtual networking:** how VirtualBox adapters connect VMs to networks.
- **Static IP configuration:** setting address, netmask, gateway and DNS in Kali, including with `nmcli`.
- **Subnet planning:** reserving address ranges for future machines.
- **Snapshots:** take a clean one *before* any risky experiment.
- **Documentation:** recording steps and fixes the way a professional would.

---

## 👤 Author

**Syed Amaan Jaffri**
Final-Year BS Cybersecurity Student, FAST NUCES, Karachi

🔗 [GitHub](https://github.com/syedamaanjaffri) • [LinkedIn](https://linkedin.com/in/syed-amaan-jaffri-571577291)
