<div align="center">
  <h1>Server Sorcery 101</h1>
  <p><strong>Chapter 1 of 8 - A scrub nurse running on cloud. A foundational DevOps journey into Infrastructure as Code, Linux Administration, and Server Security.</strong></p>
</div>

---

## Overview
Welcome to **Server Sorcery 101**. Before this project I spent years in surgical operating theatres as a scrub nurse working under stress, making sure that the environment was sterile, the equipment was ready, and nothing was left to chance before a procedure began, all for the safety of my patients. Thus, it has inspired me to use this DevOps modules to build a secure and reliable infrastructure that can be actually used in operation rooms as a study project. This is the foundation of an eight-part series where I build a production-grade cloud infrastructure from scratch, starting with
bare virtual machines and ending with a fully cloud-native,
GitOps-managed deployment on a public cloud provider (which hopefully we will see in chapter 8, the final project). The mission is to take a rudimentary network design (an application server, two web servers, and a load balancer) and bring it to life natively via **Infrastructure as Code (IaC)**.

This repository features a fully automated VM cluster configured from the ground up for strict networking, rigid security bounds, and reliable reproducibility.

### System Architecture Diagram

                         INTERNET
                            │
                       HTTP :80
                            │
                 ┌──────────▼──────────┐
                 │    HOST MACHINE     │
                 │  (macOS / VMware)   │
                 │  port 8080 → :80    │
                 └──────────┬──────────┘
                            │
        ╔═══════════════════▼════════════════════╗
        ║     PRIVATE NETWORK 192.168.56.0/24    ║
        ║         UFW: default deny incoming     ║
        ║                                        ║
        ║         ┌─────────────────┐            ║
        ║         │  loadbalancer   │            ║
        ║         │ 192.168.56.10   │            ║
        ║         │ 2 vCPU · 1G RAM │            ║
        ║         │ UFW: :80 :22    │            ║
        ║         └────────┬────────┘            ║
        ║                  │                     ║
        ║         ┌────────┴────────┐            ║
        ║         │                 │            ║
        ║  ┌──────▼──────┐  ┌───────▼─────┐      ║
        ║  │  webserver1 │  │  webserver2 │      ║
        ║  │192.168.56.11│  │192.168.56.12│      ║
        ║  │1 vCPU · 1G  │  │1 vCPU · 1G  │      ║
        ║  │ UFW: :22    │  │ UFW: :22    │      ║
        ║  └──────┬──────┘  └──────┬──────┘      ║
        ║         │                │             ║
        ║         └────────┬───────┘             ║
        ║                  │                     ║
        ║         ┌────────▼────────┐            ║
        ║         │   appserver     │            ║
        ║         │ 192.168.56.13   │            ║
        ║         │ 1 vCPU · 2G RAM │            ║
        ║         │ UFW: :22        │            ║
        ║         └─────────────────┘            ║
        ║                                        ║
        ║  SSH :22 → devops user only            ║
        ║  WireGuard VPN :51820/udp (all VMs)    ║
        ╚════════════════════════════════════════╝

**External zone**: only the host machine's port 8080 forwards into the private network, landing on the loadbalancer's port 80. Everything else is blocked at the host level.

**Private network (192.168.56.0/24)**: all four VMs live here and can talk to each other. UFW on every VM defaults to deny incoming, with explicit allow rules only for SSH (:22) and WireGuard (:51820/udp).

**Traffic flow**: internet traffic hits the loadbalancer, which distributes requests to webserver1 and webserver2. The web servers communicate with appserver for application logic. No external traffic can reach web servers or appserver directly.

**Resource allocation**: loadbalancer gets 2 vCPUs (traffic distribution is CPU-bound), appserver gets 2G RAM (application logic is memory-bound), web servers stay lean at 1 vCPU/1G.

**Security measures visible in the diagram**: SSH restricted to devops user only, WireGuard VPN on all nodes, UFW default deny, only loadbalancer externally accessible.


### Key Objectives
- **VM Creation:** Spin up four servers natively.
- **Private Subnets:** Bind all architecture under static IPs where only the perimeter has internet exposure.
- **Security Posture:** Secure SSH configuration, non-root `devops` account privileges, and robust Uncomplicated Firewall (UFW) rules.

---

## Tech Stack

| Technology | Purpose |
|------------|---------|
| **Vagrant**| Hypervisor automation and IaC lifecycle management |
| **QEMU**   | High-performance virtualization back-end (Chosen specifically to support Apple Silicon ARM64 native networking) |
| **Ubuntu** | Core Linux operating system across all instances |
| **UFW**    | Uncomplicated Firewall for precise port management |
| **Bash/SSH** | System administration & remote access |

---

## Milestone Documentation
So I was also thinking of using this module as a personal learning opportunity by documenting my journey of intergrating nursing knowlegde into cloud infrastructure. If you want to read exactly how and why this architecture was built step-by-step, visit the documentation directory:
- [Milestone 1: Virtual Machine Setup](docs/milestone-1-vm-setup.md)
- [Milestone 2: Networking](docs/milestone-2-networking.md)
- [Milestone 3: Users & SSH](docs/milestone-3-users-ssh.md)
- [Milestone 4: Firewall Hardening](docs/milestone-4-firewall-hardening.md)

---

## Getting Started

Since this project is done on my MAC, it features a specialized Apple Silicon (ARM64) architecture flow. If you are also using a MAC, make sure your machine has QEMU and Vagrant ready before spinning up the infrastructure.

### 1. Prerequisites & Installation

To run this project, you will need the following core tools installed on your operating system:
- **Vagrant**: The orchestrator for the virtual machines.
- **Ansible**: The configuration management tool used to provision the servers.
- **A Hypervisor**: Either VirtualBox (Windows), VMware Fusion (Mac Standard), or QEMU (Mac Fallback).

#### 🪟 For Windows (or Intel Macs)
Windows runs natively on x86 architecture. VirtualBox is the professional standard for this stack.
1. Install **VirtualBox** and **Vagrant** directly from their official websites.
2. Install **Windows Subsystem for Linux (WSL)** (Windows only) because Ansible runs best in a Linux environment:
   ```powershell
   # Run this in PowerShell as Administrator
   wsl --install
   ```
   *Then, open your WSL Ubuntu terminal and install Ansible:*
   ```bash
   sudo apt update && sudo apt install ansible -y
   ```

#### 🍎 For macOS (Apple Silicon M1/M2/M3)
Because VirtualBox does not natively support ARM architecture, you must pick one of two hypervisor paths:

**Path A: VMware Fusion Pro (Highly Recommended)**
This allows you to fully complete the networking milestones.
1. Download **VMware Fusion Pro** (Free for personal use via Broadcom) and install the **Vagrant VMware Utility**.
2. Install Vagrant and the VMware plugin:
   ```bash
   brew install hashicorp/tap/vagrant
   vagrant plugin install vagrant-vmware-desktop
   ```

**Path B: QEMU (Lightweight Fallback)**
Use this if you don't want to install VMWare, but note that the core **Private Networking** milestone will skip the subnets entirely.
1. Install Vagrant and QEMU:
   ```bash
   brew install hashicorp/tap/vagrant
   brew install qemu
   vagrant plugin install vagrant-qemu
   ```

#### For Windows (or Intel Macs)
Windows and older Macs run natively on x86 architecture, which means VirtualBox is the standard hypervisor to use.
1. Install **VirtualBox** and **Vagrant** directly from their official websites.
2. Install **Windows Subsystem for Linux (WSL)** (Windows only) because Ansible runs best in a Linux environment:
   ```powershell
   # Run this in PowerShell as Administrator
   wsl --install
   ```
### 2. Spinning up the Environment
We have broken down the project into three strictly isolated environments so you don't have to rewrite any code. Just `cd` into the folder that matches your hardware and boot!

**For Windows (or Intel Macs):**
```bash
cd test-windows
vagrant up
```

**For Mac Apple Silicon (VMware - Recommended):**
```bash
cd test-mac-vmware
vagrant up
```

**For Mac Apple Silicon (QEMU - Fallback):**
```bash
cd test-mac-qemu
vagrant up
```
*Note: Vagrant will automatically download the correct boxes for your respective OS and provision your VMs.*

---

## How to Test the Infrastructure

To help you testing that this actually worked, prefer to this doc: [test.md](test.md)


## Bonus Tests
I have also expanded the infrastructure to include the extra features, here is how you prove they work:

### Testing Fail2Ban
1. Try to log into one of the VMs via SSH from your laptop with the wrong password 5 times in a row.
2. On the 6th try, the server should completely ghost you (*Connection refused*).
3. Log in correctly from elsewhere and run `sudo fail2ban-client status sshd`. You can see that your laptop's IP address is in the "Banned" list!

### Testing Netdata
1. Open your browser and navigate to `http://192.168.56.11:19999`.
2. You'll see a lightning-fast dashboard.
3. If you want, you can run `yes > /dev/null` on the server and watch your browser's CPU graph immediately redline.

### Testing WireGuard
1. Run `sudo wg show` on the server.
2. Even with nobody actively connected, the interface `wg0` will appear, confirming the kernel driver is active and listening securely.

---

