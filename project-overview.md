# Server Sorcery 101 🧙‍♂️

Welcome to your first adventure in the world of virtualization and Linux server management. In this task, you'll be setting up a small network of virtual machines (VMs) and getting hands-on with some essential Linux administration skills.
Let's get started!🚀

## The situation 👀
You've just joined a promising tech startup as their first DevOps engineer🛠️. The company is preparing to launch its revolutionary web application, and they need your expertise to set up a robust, scalable, and secure infrastructure.

The CTO has outlined a basic architecture: an application server to host the core logic, two web servers to handle user requests, and a load balancer to distribute traffic efficiently. Your mission is to bring this vision to life using virtual machines, ensuring the environment is not only functional but also adheres to best practices in system administration and security.

## Functional requirements 📋
### Document your process and decisions for future reference. 📝
As you embark on this journey, keep a detailed log of your steps, decisions, and any challenges you encounter. This documentation will be invaluable for future reference and for sharing your knowledge with your peers. Plus, it's a great way to showcase your problem-solving skills. 🌟

### VM Creation and Networking 🔌
You start building. You'll need to create a virtual environment that mirrors a production setup, with machines that can talk to each other and be accessed as needed.

#### Creating the VMs 🖥️
You decide to create four VMs, each with a specific role and descriptive hostname to make it clear what its purpose is. As you configure each VM, you allocate appropriate resources (CPU, memory, storage) based on their roles.
The application server might need more memory, while the load balancer might need more CPU power.

#### Assigning Static IPs 📶
To ensure reliable communication, you assign static IP addresses and configure the network settings so that these IPs are fixed and do not change, ensuring stable connections.

#### Configuring Network Settings 🔧
You set up the network so that all VMs can communicate with each other. This involves configuring the network interfaces and ensuring that the VMs are on the same subnet.
You also ensure that only necessary VMs are accessible from the outside to minimize the attack surface.

### Basic Linux Administration 🐧
As you dive into your new role at the startup, you quickly realize the importance of setting up a secure and efficient environment for your virtual machines.
The CTO has emphasized the need for robust security measures and efficient user management.

#### Creating the DevOps User 👤
You start by creating a dedicated user named devops on each VM. This user will handle all administrative tasks, ensuring that the root account remains untouched for added security. You add the user to the right group, granting it the necessary privileges to perform administrative actions.

#### Securing SSH Access 🔐
To enhance security, you configure the devops user to allow only SSH key login. You take it a step further by ensuring that only the devops user is permitted to log in via SSH.

#### Disabling Unused Network Interfaces 📴
Next, you focus on minimizing the attack surface by disabling any unused network interfaces. This ensures that only the necessary network connections are active, reducing the risk of unauthorized access.

#### Activating the Firewall 🔥
You activate UFW (Uncomplicated Firewall) and configure it to allow only the necessary traffic. This helps protect your VMs from unwanted network connections and potential attacks.

#### Setting a Secure Umask 🔒
You configure secure umask to ensure that newly created files and directories are not world-readable or writable.

#### Enabling Automatic Security Updates 🔄
Finally, you enable and configure automatic security updates. This ensures that your VMs are always up-to-date with the latest security patches, protecting them from known vulnerabilities.

## Extra requirements 📚
Feeling adventurous? Here are some extra challenges to level up your DevOps game. These tasks are a bit more challenging but will give you a deeper understanding of system security and monitoring. Ready to take on the challenge? Let's go! 🚀

### Install Intrusion Prevention Software 🛡️
Choose and install one of the following intrusion prevention tools:
Fail2Ban: Protects your server from brute-force attacks by monitoring log files and banning IPs that show malicious behavior.
CrowdSec: A modern, collaborative security engine that leverages both local and global IP reputation to protect against various threats.
DenyHosts: Specifically designed to prevent SSH brute-force attacks by monitoring and blocking IP addresses.
SSHGuard: Monitors log files for malicious activity and blocks offending IP addresses using various firewall backends.

### Implement a VPN 🌐
Set up a VPN to secure your network traffic:
OpenVPN: A popular open-source VPN solution known for its security and flexibility.
WireGuard: A modern VPN protocol that is simpler and faster than traditional options.

### Set Up a Basic Monitoring Tool 📊
Install and configure a monitoring tool to keep an eye on your VMs' performance:
Grafana + Prometheus: Collects metrics and visualizes them in modern dashboards.
Netdata: Provides real-time performance monitoring with zero configuration.
Zabbix: A powerful open-source monitoring solution for networks and applications.
By tackling these bonus tasks, you'll gain valuable experience in advanced security and monitoring techniques, making you a more well-rounded DevOps engineer. Good luck, and have fun exploring these new challenges! 🎉

## Bonus functionality 🎁
You're welcome to implement other bonuses as you see fit. But anything you implement must not change the default functional behavior of your project.
You may use additional feature flags, command line arguments or separate builds to switch your bonus functionality on.

## What you'll learn 🧠
Setting up and managing VMs using different virtualization technologies.
Configuring and managing network settings for VMs.
Basic Linux server administration and security practices.

### Deliverables and Review Requirements 📁
A README file with:
- Project overview
- Setup and installation instructions
- Usage guide
- Any additional features or bonus functionality implemented

During the review, be prepared to:
Demonstrate your environment
Discuss any challenges you faced and how you overcame them