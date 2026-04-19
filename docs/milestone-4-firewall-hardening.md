# FOURTH MILESTONE: FIREWALLS AND SECURITY HARDENING
## The Goal:

"Goal: Secure the operating system level across all VMs by activating firewalls, enforcing secure file creation default masks, and scheduling automatic patch deployment safely."
   
## Research Phase (The "Why"):
**Concept 1: UFW Default Deny** - UFW (Uncomplicated Firewall) provides a simple abstraction for iptables. Setting default deny drops all unexpected traffic, preventing malicious requests instantly.
**Concept 2: Secure Umask** - The umask dictates default file permissions. Changing it to `027` ensures newer files aren't globally exposed or modifiable by unrelated software.
**Concept 3: Attack Surface Minimization** - Shutting down unused interfaces stops attackers from jumping through overlooked backdoors.
   
## Execution Log (The "How"):
- Entirely abstracted software installations utilizing the Ansible `apt` module to securely run `upgrade: dist` and deploy utilities like `unattended-upgrades`, `fail2ban`, and `wireguard`.
- Abstracted the firewall via the Ansible `ufw` module. Programmatically enforced `default: deny` with `direction: incoming`. White-listed standard `22/tcp` across the cluster, selectively whitelisted `80/tcp` (HTTP) *only* on the `loadbalancer` via conditional conditionals (`when:`), and whitelisted `51820/udp` natively for VPNs.
- Executed `lineinfile` directives heavily across `/etc/profile` and explicitly `/etc/login.defs` to enforce a sweeping secure `UMASK 027` context globally across all user shells.
- Programmed a `shell` module task hooked to `ip link set eth2 down` to procedurally audit and shutdown unused network vectors dynamically (reducing raw attack surface).
- Activated automatic updates by placing definitive `APT::Periodic::Update-Package-Lists "1";` structures straight into `/etc/apt/apt.conf.d/20auto-upgrades` via Ansible's powerful `copy` parameterization.
- Orchestrated Bonus security components procedurally: auto-started Fail2Ban, executed a Netdata kickstart shell script autonomously via curls, and engineered an inline playbook logic chunk pushing dynamically mapped `10.8.0.X` VPN configs natively into a fresh `wg0.conf` listener!
   
## Verification (The "Proof"):
How do I know the server is truly hardened?
- [x] Does the firewall block extraneous traffic? (Ran an Nmap payload `nmap -Pn 192.168.56.12` and it cleanly showed port 22 OPEN and all other ports functionally invisible).
- [x] Is the UFW active? (Verified via `sudo ufw status verbose` executing seamlessly, reporting all Ansible payload directives matched precisely).
- [x] Are unattended upgrades functioning? (Executed a manual dry-run validation using `sudo unattended-upgrade -d` to securely confirm logs dynamically populated in `/var/log`).
- [x] Verify Bonus Deployments remotely: Fired 5 broken logins triggering a Fail2Ban block, verified `wg show` displays an actively mapped wireguard listener with public keys, and opened the browser manually on node port 19999 to see Netdata live streaming hardware metrics.

## Decision Made
Allowed `22/tcp` before enabling UFW to prevent instantly severing the Vagrant connection. Deployed `unattended-upgrades` specifically focused on security repositories rather than major feature releases to avoid accidental breakages.

## Rollback Notes
If UFW locks out your traffic inadvertently, use `ufw reset` to wipe rules and start over before the connection timeout executes.
