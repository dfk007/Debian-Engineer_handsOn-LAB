This guide provides the complete, step-by-step workflow an enterprise system administrator would follow to install and configure a production-grade **headless Debian 13 (Trixie)** server.

The philosophy is **minimalism, security, and automation**.

------



## 1. Initial Installation & Minimal Configuration (Beginner)



The goal here is to install the absolute minimum necessary to have a remote-manageable server.



### A. Debian Installer: Key Selections



1. **Installation Media:** Use the **netinst ISO** (network install). It's smaller and forces you to fetch the latest packages during installation.
2. **Partitions:**
	* For a modern server, choose **Guided partitioning - Use entire disk and setup LVM**.
	* **LVM (Logical Volume Manager)** allows for flexible disk resizing and creating snapshots later without downtime, which is essential in an enterprise setting.
	* Create a separate small partition for `/boot` (e.g., 500MB) and allocate the rest to a root volume group (`/` or separate `/home`, `/var`, etc., depending on the application).
3. **Software Selection:** This is the most crucial step for a production server.
	* **Deselect** (uncheck) the **"Debian desktop environment"** option.
	* **Deselect** all other specific desktop environments (GNOME, KDE, etc.).
	* **Keep Selected** the two most important options:
		* ☑️ **SSH server** (for remote access).
		* ☑️ **standard system utilities** (for basic tools).



### B. First Login and Root Access Policy



After the reboot, the server will drop to a command-line login prompt.

1. **Initial Login:** Log in as the non-root user you created during the installation.

2. **System Update:** Perform a full package update.

	Bash

	```
	sudo apt update && sudo apt upgrade -y
	```

3. **Verify Hostname and Timezone:** Ensure the server has a correct **Fully Qualified Domain Name (FQDN)** and time setting.

	Bash

	```
	hostnamectl set-hostname your-fqdn.example.com
	sudo timedatectl set-timezone Region/City
	```

------



## 2. Server Hardening and Access Control (Intermediate)



This is where the server is secured for production use.



### A. Secure Shell (SSH) Hardening



The SSH protocol is the administrative front door, and it must be secured immediately.

1. **Use SSH Keys (Essential):** Generate an SSH key pair on your admin workstation and copy the public key to the server.

	Bash

	```
	# On your admin workstation:
	ssh-keygen -t ed25519 
	ssh-copy-id your_user@server_ip
	
	# On the server (after successful login):
	# Verify the key is in ~/.ssh/authorized_keys
	```

2. **Disable Password and Root Login:** Edit the SSH configuration file (`/etc/ssh/sshd_config`).

	Bash

	```
	sudo nano /etc/ssh/sshd_config
	
	# Change these lines:
	PermitRootLogin no
	PasswordAuthentication no
	
	# Optional: Change the default port 22 to a random high port (e.g., 2222)
	Port 2222 
	
	# Restart the SSH service:
	sudo systemctl restart sshd
	```

	> **Note:** **DO NOT** close your current SSH session until you successfully log in using a new terminal window with your SSH key.



### B. Firewall Configuration



A production server must follow the **principle of least privilege** for network traffic.

1. **Install Firewall:** Debian often uses `nftables` or allows for simpler tools like **UFW (Uncomplicated Firewall)** for quick setup.

	Bash

	```
	sudo apt install ufw
	```

2. **Configure Rules:** Allow only the necessary ports (SSH, HTTP/HTTPS, etc.).

	Bash

	```
	sudo ufw default deny incoming
	sudo ufw default allow outgoing
	sudo ufw allow ssh   # Use 'allow 2222' if you changed the port
	sudo ufw allow http  # Port 80
	sudo ufw allow https # Port 443
	sudo ufw enable
	```



### C. Automated Security Updates



Set up the system to automatically install security patches.

1. **Install Unattended Upgrades:**

	Bash

	```
	sudo apt install unattended-upgrades
	```

2. **Configure:** The default configuration is typically sufficient for security updates. Run the following command to enable it:

	Bash

	```
	sudo dpkg-reconfigure unattended-upgrades
	```

------



## 3. Enterprise Administration and Advanced Hardening (Advanced)



A senior sysadmin is responsible for ensuring the long-term stability, performance, and compliance of the server.



### A. Kernel and Filesystem Hardening (System Integrity)



1. **Disable Unnecessary Modules:** Edit the `/etc/modprobe.d/blacklist.conf` to blacklist kernel modules not required by the server.

2. **Sysctl Tuning:** Apply security-focused kernel parameters via `/etc/sysctl.conf`.

	* **Example (Anti-Spoofing and Network Hardening):**

		Bash

		```
		net.ipv4.conf.all.rp_filter=1      # Reverse path filtering
		net.ipv4.conf.default.rp_filter=1
		net.ipv4.ip_forward=0              # Disable IP forwarding (if not a router)
		net.ipv4.icmp_echo_ignore_all=1    # Ignore broadcast ICMP requests
		```

	* Apply the changes: `sudo sysctl -p`

3. **Integrity Checks:** Install and configure tools to detect unauthorized changes.

	Bash

	```
	# Install Aide (Advanced Intrusion Detection Environment)
	sudo apt install aide
	sudo aideinit 
	# Move the new DB to the default location after first run
	sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db
	# Set up a cron job for daily checks.
	```



### B. Auditing and Monitoring



1. **Log Aggregation:** Configure the local `systemd-journald` to forward logs to a central **SIEM (Security Information and Event Management)** or log aggregation service (e.g., ELK stack, Splunk, Graylog).

2. **Intrusion Detection:**

	* Install **Fail2Ban** to scan log files and dynamically ban IP addresses that show malicious signs (e.g., multiple failed SSH logins).

	Bash

	```
	sudo apt install fail2ban
	# Configure it by creating a file in /etc/fail2ban/jail.d/
	```

	* Configure **monitoring agents** (e.g., Prometheus Node Exporter, Zabbix agent, Nagios/Icinga agent) to report performance metrics and service health back to the central monitoring system.



### C. Professional Workflow (Infrastructure as Code)



In a true enterprise environment, the manual steps above are replaced by **automation** to ensure consistency and speed across hundreds of servers.

| **Phase**         | **Tool/Technology**              | **Sysadmin Role**                                            |
| ----------------- | -------------------------------- | ------------------------------------------------------------ |
| **Provisioning**  | **Terraform** / Cloud-init       | Define the server infrastructure.                            |
| **Configuration** | **Ansible** / Puppet / SaltStack | Write playbooks to apply all hardening, install applications (e.g., Web Server, Database), and configure monitoring agents consistently. |
| **Deployment**    | **Git** / CI/CD Pipeline         | Manage the code for the playbooks/configurations and automatically deploy changes to the server fleet. |

The core advanced skill is writing and maintaining **Ansible playbooks** that codify steps 2 and 3 of this entire workflow, turning a one-time manual process into a repeatable, auditable, and automated one.