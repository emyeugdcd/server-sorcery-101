# Server Sorcery 101 – Test & Validation Guide

The goal of this file is to guide you step-by-step from zero to fully validating every project requirement. Follow these steps in order!

---

## Phase 1: Boot the Environment (Vagrant)
Before checking or configuring anything, you must start up the raw Virtual Machines.

Run this from your host machine inside your project folder:
```bash
vagrant up
```
*Wait until Vagrant finishes booting all 4 VMs.*

---

## Phase 2: Deploy Configuration (Ansible)
Now that the bare base servers are running, deploy the playbook to install software, create users, configure firewalls, and lock down SSH.

Run this from your host machine to test connectivity first:

```
ansible -i inventory.ini servers -m ping
```

If that returns green, run the playbook:

```bash
ansible-playbook -i inventory.ini setup.yml
```
*Wait for the playbook to finish. Ensure there are no red failed tasks at the very end.*
Sometimes, Ansible might be locked out of the VMs if you run the playbook multiple times. In that case, you can use the following command to reset the VMs:
```bash
vagrant destroy -f
```

then run `vagrant up` again

This is actually the correct workflow, you always run the playbook against fresh VMs, not ones that were already hardened by a previous run.
---

## Phase 3: Requirement Validation
Now we verify that everything was configured correctly! 

### 1. Verify Static IP Configuration

First confirm each VM actually has the IP assigned.

Get into loadbalancer by running: 
```bash
ssh -i .vagrant/machines/loadbalancer/vmware_desktop/private_key devops@192.168.56.11
```
or exit the current VM by running `exit` and then if you want to enter other VMs, run the same command above but replace `loadbalancer` with the name of the VM you want to enter with the correct devops user and IP address (e.g. `ssh -i .vagrant/machines/webserver1/vmware_desktop/private_key devops@192.168.56.12`), check the inventory.ini for more information

Then, run:

```bash
ip a
```

Look for something like this (usually in the eth1 interface at number 3)

```text
inet 192.168.56.11/24
```

You can also run:

```bash
hostname
```

Example output:

```text
loadbalancer
```

Expected mapping:

```text
loadbalancer 192.168.56.11
webserver1 192.168.56.12
webserver2 192.168.56.13
appserver 192.168.56.14
```

If the IP is wrong, check your **Netplan config**:

```bash
sudo nano /etc/netplan/*.yaml
```

Then apply:

```bash
sudo netplan apply
```

---

# Verify Hostname Resolution

To prove that DNS / `/etc/hosts` routing works natively, we must test that the machines know each other by their string name, not just IP.

Run from `loadbalancer`:
```bash
ping webserver1
```
Expected:
```text
64 bytes from 192.168.56.12
```

---

# Test Server-to-Server Networking

The test above also checks that the VMs **can reach each other**.

From `loadbalancer`:

```bash
ping webserver1
```

Expected:

```text
64 bytes from webserver1
```

Also try:

```bash
ping 192.168.56.12
```

Repeat between all servers.

Example network test:

```text
loadbalancer → webserver1
loadbalancer → webserver2
webserver1 → appserver
```

If this works, our **static network and /etc/hosts are correct**.

---

# Verify SSH Security

The testing required:

✔ root login disabled
✔ SSH allowed

Check SSH config:

```bash
sudo cat /etc/ssh/sshd_config | grep PermitRootLogin
```

Expected:

```text
PermitRootLogin no
```

Now try switching to root:

```bash
su root
```
It will ask for password of your host machine. Either way, you should **not be able to SSH directly as root**.

If we want to check if SSH daemon is running?

```bash
sudo systemctl status ssh
```
This confirms the server is running **OpenSSH** if it shows **active**

---

# Verify devops User & Secure SSH
The `devops` user must be accessible via Key *without* a password, but running `sudo` must legally require a password for security auditing.

1. First, SSH into the machine as `devops` from your Host by pointing to Vagrant's private key (adjust `vmware_desktop` if using a different provider). This task has been done from the beginning
```bash
ssh -i .vagrant/machines/loadbalancer/vmware_desktop/private_key devops@192.168.56.11
```

2. Once logged in, prove you belong to the `sudo` group:
```bash
groups devops
```
*(Expected: `devops : devops sudo`)*

3. Test password-protected sudo execution:
```bash
sudo visudo
```
*(Expected: It will immediately prompt you for the `devops` password. Type `password` to authenticate).*
However, if you ran a sudo command right before running `sudo visudo`, Ubuntu caches your password for 15 minutes so you don't have to keep typing it back-to-back. To prove it works: clear your sudo cache by typing sudo -k. Then immediately run sudo visudo again. You should see the [sudo] password for devops: prompt re-appear perfectly!


# Verify Firewall Rules

Ubuntu firewall is:

UFW

Check status:

```bash
sudo ufw status verbose
```

Expected result:

```text
Status: active

22/tcp ALLOW
```

Everything else should be blocked.

Example secure policy:

```text
Default: deny incoming
```

---

### Simulate a port scan

A common testing tool is:

Nmap

Install:

```bash
sudo apt install nmap
```

Scan another VM:

```bash
nmap -Pn 192.168.56.12
```

Expected:

```text
22/tcp open
```
If other ports appear open, the firewall isn't configured properly.

---

# Verify Automatic Security Updates

Ubuntu auto-patching system:

unattended-upgrades

Check status:

```bash
sudo systemctl status unattended-upgrades
```

You should see:

```text
active (running)
```

Check strictly enforced rubric configuration:

```bash
cat /etc/apt/apt.conf.d/20auto-upgrades
```

Look for the exact required strings:

```text
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

### Prove VMs are currently fully updated
Run a validation check against the package manager to prove `setup.yml` installed all security patches at boot:

```bash
sudo apt update && sudo apt list --upgradable
```

*(Expected output: "Listing... Done". No packages should be visibly waiting for security upgrades!)*
However, third party repositories like Netdata might have newer versions available. This is expected behavior and does not indicate a failure in the auto-update configuration.

---

### Check last update logs

```bash
cat /var/log/unattended-upgrades/unattended-upgrades.log
```

Example:

```text
Packages upgraded
Security updates installed
```

However, if you encounter a missing log file, there's a simple and logical reason for it: The system just hasn't had time to run it yet!

unattended-upgrades is driven by a scheduled system timer (a systemd timer that usually fires once a day). Since we literally just spawned these VMs a few minutes ago, that timer hasn't naturally gone off yet. Additionally, because our setup.yml playbook explicitly updated everything manually with apt upgrade: dist at the very beginning of its run, there is currently nothing for unattended-upgrades to do even if it did wake up. The log file gets generated the very first time it executes a sweep.
So to make sure the log file is generated, we can run the following command:

```bash
sudo unattended-upgrade -d
```
(It will run through its checks and immediately print its logic to the terminal). After that finishes, it will have permanently generated the directory and log files!

Now we can run this command again:
```
sudo cat /var/log/unattended-upgrades/unattended-upgrades.log
```
---

# Verify the devops User

Check that the user exists:

```bash
id devops
```

Expected:

```text
uid=1001(devops)
groups=sudo
```

This confirms:

✔ user created
✔ sudo privileges granted

---

# Infrastructure Health Checklist

This is a **professional-style validation checklist**.

| Test              | Command                                | Expected        |
| ----------------- | -------------------------------------- | --------------- |
| hostname correct  | `hostname`                             | loadbalancer         |
| static IP         | `ip a`                                 | correct address |
| network works     | `ping webserver1`                         | replies         |
| SSH active        | `systemctl status ssh`                 | running         |
| root SSH disabled | `grep PermitRootLogin`                 | no              |
| firewall active   | `ufw status`                           | active          |
| SSH allowed       | `ufw status`                           | 22 allowed      |
| auto updates      | `systemctl status unattended-upgrades` | running         |

If all pass → **assignment complete**.

---

# One Very Useful DevOps Debug Skill

If something breaks, always check these **three layers**:

```
Layer 1: network
Layer 2: service
Layer 3: security
```

Example troubleshooting:

```
Can't SSH?
   ↓
Check network (ping)
   ↓
Check service (systemctl status ssh)
   ↓
Check firewall (ufw status)
```

This mental model saves **hours of debugging**.

---

💡 **Final pro tip**

Most beginners forget to test **VM-to-VM communication**.

Run something like:

```bash
ping webserver2
ssh devops@appserver
```

between machines.

---


How to "Test" Your Work (The Proof)

To prove you aren't a fraud, you must verify. Here is how you test each bonus like a Senior SRE:

1. Testing Fail2Ban

The Test: Try to log into one of your VMs via SSH with a wrong password 5 times in a row from your laptop.

The Result: On the 6th try, the server should "ghost" you (Connection refused).

The Proof: Log in (correctly) and run sudo fail2ban-client status sshd. It will show your laptop's IP address in the "Banned" list!

2. Testing Netdata

The Test: Open your browser and go to http://192.168.56.11:19999.

The Result: You should see a high-speed dashboard.

The "Pro" Test: Run a heavy command on the server (like yes > /dev/null) and watch the CPU graph in your browser spike in real-time.

3. Testing WireGuard

The Test: Run wg show on the server.

The Result: It should show the interface wg0 (even if no one is connected yet). This proves the driver is active and listening.