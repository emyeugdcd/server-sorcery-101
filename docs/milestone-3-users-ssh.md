# THIRD MILESTONE: SYSTEM USERS AND SSH Hardening
## The Goal:

"Goal: Create a dedicated administrator user named `devops` with sudo privileges, and tighten system access by preventing direct root SSH logins."
   
## Research Phase (The "Why"):
**Concept 1: SSH Key Pairs vs Passwords** - Passwords can be brute-forced. Cryptographic key pairs are significantly harder to crack and are standard across modern infrastructure.
**Concept 2: Disabling Root Login** - Root has infinite power. If the root account is compromised directly, the server is lost. By forcing users to log in with a separate account and *then* escalate privileges, we create an audit trail and an extra defensive layer.
   
## Execution Log (The "How"):
- Configured an Ansible playbook to entirely script the user creation.
- Leveraged the Ansible `user` module to deploy the `devops` user organically, explicitly joining them to the `sudo` group and assigning a cryptographic hashed string for their sudo authentication password.
- Bootstrapped SSH access flawlessly by using the Ansible `copy` module to dynamically mirror Vagrant's `.ssh/authorized_keys` straight into the `devops` SSH directory (`remote_src: yes`).
- Modified OpenSSH configurations via the Ansible `lineinfile` module targeting `/etc/ssh/sshd_config`. Safely regexed and replaced `PermitRootLogin no`, `PasswordAuthentication no`, and strictly enforced an `AllowUsers devops vagrant` directive.
- Converted the playbook to cleanly rely on a `notify: Restart SSH` Handler, ensuring the daemon cleanly reloads all SSH restrictions linearly without crashing Ansible midway.
- Added a final stage lock down task that strips `vagrant` from the `AllowUsers` list dynamically after Ansible finishes bootstrapping.
   
## Verification (The "Proof"):
How do I know the access rules are enforced?
- [x] Is the `devops` user created correctly? (Verified via SSHing directly from the Host Mac using `-i .vagrant/.../private_key devops@IP` and running `id devops` showing the correct UNIX groups including `sudo`).
- [x] Does the `PermitRootLogin` flag block access? (Verified by attempting `ssh root@192.168.56.11` resulting in standard rejections).
- [x] Is the OpenSSH service actually running securely? (Checked `systemctl status ssh` and verified it reports "active (running)" and requires `devops` keys only).

## Decision Made
We opted to completely block root login rather than permitting it only with keys, enforcing the best practice of sudo delegation for all administrative acts.

## Rollback Notes
If you accidentally lock yourself out of SSH, use the hypervisor console (e.g. QEMU GUI backend or `vagrant ssh` native backend if password auth is still on) to revert `/etc/ssh/sshd_config`. Or the best practice is to destroy all the vagrant machines, then `vagrant up` again and run the playbook.
