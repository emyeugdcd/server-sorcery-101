# FIRST MILESTONE: VIRTUAL MACHINES (VM) SET UP
## The Goal:

"Goal: Automate the creation of 4 Virtual Machines (application server, two web servers, and a load balancer) that can communicate on a private network, using Infrastructure as Code (IaC)."
   
## Research Phase (The "Why"):
Concepts needed to understand before building the infrastructure:
**Concept 1: Infrastructure as Code (IaC)** - Why use Vagrant instead of manually clicking through a Hypervisor GUI? (Speed, repeatability, and avoiding drift).
**Concept 2: Static IPs vs Dynamic IPs** - Why give servers fixed static IPs? (So databases and web servers don't lose each other).
**Concept 3: Architecture Challenges (Hypervisors & OS)**
   - **For Windows / Intel Macs:** Standard x86 architecture is perfectly supported by VirtualBox natively.
   - **For Apple Silicon (M1/M2/M3):** VirtualBox falls back to sluggish emulation. To achieve native ARM64 speed, VMware Fusion Pro (`vmware_desktop`) is the ideal, professional path supporting full networking. QEMU (`qemu`) is a lightweight alternative, but it sacrifices private Virtual Subnets capabilities.
   
## Execution Log (The "How"):
- Established three unique `Vagrantfile` environments to handle hardware divergence organically.
- Designed `test-windows` utilizing VirtualBox and `ubuntu/jammy64`.
- Designed `test-mac-vmware` utilizing VMware Fusion and `bento/ubuntu-22.04-arm64`.
- Designed `test-mac-qemu` utilizing QEMU and `perk/ubuntu-2204-arm64`.
- Booted all machines simultaneously via `vagrant up` inside the respective environment folder.
- Drafted an automated Ansible playbook (`setup.yml`) and `inventory.ini` to consistently deploy packages and configurations across the fleet after they boot, replacing manual bash commands.
   
## Verification (The "Proof"):
How do I know the infrastructure is correctly provisioned?
- [x] Run `vagrant up` completely without errors.
- [x] Did all 4 machines get created? (Used `vagrant status` to verify they are running).
- [x] Can I SSH into a specific machine using Vagrant's automation? (Verified via `vagrant ssh loadbalancer`).
- [x] Is the static IP assigned? (Verified via `ip a` on the VM).

## Decision Made
**Pivot to QEMU**: Switched the virtualization provider from VirtualBox to QEMU. VirtualBox emulation of x86 on Apple Silicon causes major slowdowns and compatibility crashes. QEMU and the ARM64 box allowed native performance.

## Rollback Notes
If the QEMU plugin fails, try destroying the Vagrant setup (`vagrant destroy -f`) and reinstalling the plugin.
