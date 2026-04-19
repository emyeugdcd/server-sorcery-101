# Server Sorcery Learning Notes

## Ansible Basics
Ansible uses **YAML** (Yet Another Markup Language). It is very picky about spaces, *not* tabs!

### The Anatomy of a Playbook:
- `hosts: servers`: Tells Ansible which group from your `inventory.ini` to target.
- `become: yes`: This is like saying `sudo`. It tells Ansible to run commands as a superuser.
- `tasks:`: This is a list of things to do.
- `name:`: A human-readable label. This shows up in your terminal so you know what's happening.
- **`module`** (e.g., `apt`, `user`, `ufw`): This is the most important part. Ansible doesn't just "run commands"; it uses Modules. Instead of you typing `apt-get install...`, you tell the `apt` module the state you want (e.g., `state: present`).

### Special Syntax Rules
- **Idempotency**: This is the fancy DevOps word for "If I run this 100 times, the result is the same." Ansible checks if a user exists first. If they do, it does nothing. This is safer than a script that might try to create the same user twice and error out.
- **Indentation**: Everything under `tasks:` must be indented by exactly 2 spaces. If you mess up the spaces, it breaks.

---

## The DevOps Toolkit: Essential Commands

### Vagrant (The Infrastructure Builder)
```bash
vagrant up              # Create and start the VMs.
vagrant halt            # Shut down the VMs gracefully.
vagrant destroy         # Delete the VMs entirely (wipes the "hard drive").
vagrant ssh [name]      # Log into a specific VM manually.
vagrant reload --provision # Restart the VM and re-run any setup scripts.
```

### Ansible (The Configuration Master)
```bash
# The "Pulse Check." Checks if Ansible can talk to all servers.
ansible -m ping all -i inventory.ini 

# Run your instructions.
ansible-playbook -i inventory.ini setup.yml 

# "Dry Run" mode. It tells you what would happen without actually changing anything. (Very safe!)
ansible-playbook -i inventory.ini setup.yml --check 
```

## What The Ansible Playbook Does, Phase by Phase
### Phase 1 — System baseline (tasks 1-2)

The first thing you do is update all packages to the latest versions and map hostnames into /etc/hosts. This means every VM immediately knows the names loadbalancer, webserver1 etc. without needing DNS. You're establishing a clean, up-to-date, name-aware foundation before doing anything else.

### Phase 2 — User setup (tasks 3-5)
You create the devops user with a hashed password and sudo access, create their .ssh directory with correct permissions, then copy vagrant's authorized keys into devops's account. This is the bootstrap trick — instead of generating new SSH keys, you reuse the ones Vagrant already set up, so you can immediately SSH as devops using the same key Vagrant uses.

### Phase 3 — Firewall (tasks 6-10)
You configure UFW before installing any software, which is good practice. Default deny on incoming, then punch specific holes — SSH for everyone, HTTP only on the loadbalancer. This reflects the architecture: only the loadbalancer should receive web traffic, the other VMs are internal only.

### Phase 4 — Automatic updates and security tools (tasks 11-17)
You install and configure unattended-upgrades so security patches apply automatically without manual intervention. Then Fail2Ban, which watches SSH logs and bans IPs that repeatedly fail to authenticate — this directly addresses brute-force attacks. Then Netdata for monitoring and WireGuard for VPN, both extras that strengthen your submission.

### Phase 5 — SSH hardening (tasks 18-20)
Now that everything is installed and devops is ready, you lock down SSH itself. Disable password auth entirely (key-only), allow vagrant temporarily alongside devops, set the secure umask so newly created files aren't world-readable.

### Phase 6 — Cleanup and final lockdown (tasks 21-23)
Disable unused network interfaces to reduce attack surface, then the final task removes vagrant from AllowUsers. The handler fires here and restarts SSH with the fully hardened config. From this point on, only devops with an SSH key can get in.

### How to Test Ansible
Once all the VMs are up, test connectivity first:

```
ansible -i inventory.ini servers -m ping
```

If that returns green, run the playbook:

```
ansible-playbook -i inventory.ini setup.yml
```

Add -v for verbose output if something fails — it'll show exactly which task broke and why.