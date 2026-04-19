# SECOND MILESTONE: NETWORK TOPOLOGY
## The Goal:

"Goal: Guarantee that the 4 provisioned VMs share a secure private network and can successfully ping each other across the internal subnet."
   
## Research Phase (The "Why"):
**Concept 1: Private Networks vs Public Networks** - Exposing internal application servers directly to the internet is a severe vulnerability. They should communicate via a private subnet that the outside world cannot access.
**Concept 2: Hostfile Resolution (`/etc/hosts`)** - Understanding how Linux resolves names to IPs internally when not using a full DNS server.
   
## Execution Log (The "How"):
- Configured Vagrant to use a `private_network` subnet via `node.vm.network "private_network"`.
- Assigned distinct static IPs on the `192.168.56.0/24` subnet mapping to `loadbalancer`, `webserver1`, `webserver2`, and `appserver`.
- The `qemu` provider automatically manages the virtual bridges on the host machine to bind the VMs to the correct internal subnet.
- Brought the network online by completing the VM booting phase.
- Used the Ansible `blockinfile` module in `setup.yml` to programmatically inject static mappings of all 4 node IPs natively into every VM's `/etc/hosts` file, ensuring instantaneous name resolution without relying on external DNS.
   
## Verification (The "Proof"):
How do I know the network is correctly bonded?
- [x] Can `loadbalancer` accurately reach the other servers via network packets? (Verified via `ping 192.168.56.12` and received correct byte responses).
- [x] Tested full bi-directional communication (e.g., Pinged `loadbalancer` from `webserver1`, and pinged `webserver2` from `appserver`).
- [x] Do all network cards map properly? (Checked Netplan configuration and `ip a`).

> **⚠️ QEMU Limitation Exception:** If you are running on an Apple Silicon Mac using the `vagrant-qemu` plugin fallback, you **will not see the 192.168.56.x subnet in `ip a` nor be able to complete this ping test.** The QEMU community plugin silently ignores Vagrant's private network commands. To achieve this milestone fully on a Mac natively, you must use the `test-mac-vmware` environment via VMware Fusion.

## Decision Made
Chose to bind all servers strictly to the `192.168.56.x` subnet. This ensures they route traffic amongst themselves without exiting the hypervisor entirely and without internet exposure.

## Rollback Notes
If networking fails to establish, check if the Vagrant network plugin matches the hypervisor (e.g. QEMU bridge adapters). Check `/etc/netplan/` on the VMs specifically.
