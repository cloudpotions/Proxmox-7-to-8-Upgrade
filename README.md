# Proxmox-7-to-8-Upgrade
 This guide provides a clear and concise upgrade path from Proxmox VE 7 to 8, ensuring it's easy to copy and follow the commands. This guide was inspired by the official instructions from the  [Proxmox VE Wiki.](https://pve.proxmox.com/wiki/Upgrade_from_7_to_8)

Bonus: For additional security enhancements and Proxmox optimization, refer to the Linux Security Setup recommendations and Proxmox Starter Scripts provided at the end of this guide. 

 ![Cloud Potions Wizard Proxmox 7 to 8 Picture](https://github.com/cloudpotions/Proxmox-7-to-8-Upgrade/blob/main/CP%20Proxmox%207%20to%208.jpg?raw=true)

 # Upgrading Proxmox VE from 7 to 8

This guide provides step-by-step instructions to upgrade Proxmox VE from version 7 to version 8. Inspired by the official instructions from [Proxmox VE Wiki](https://pve.proxmox.com/wiki/Upgrade_from_7_to_8).

## Prerequisites

- Ensure you are running minimum Proxmox VE 7.4-15 on all nodes.
- Have reliable access to the node (preferably via console or iKVM/IPMI).
- Valid and tested backups of all VMs and CTs.
- At least 5 GB of free disk space on the root mount point.
- A healthy cluster.

## Step-by-Step Upgrade Guide

### 1. Run the pve7to8 Checklist Script

```
pve7to8 --full
```

2. Move Important VMs and CTs
Ensure any critical VMs and CTs are migrated away from the node being upgraded.

3. Update APT Repositories
Update to the Latest Proxmox VE 7.4 Packages

Run these commands one at a time
```
apt update
```
```
apt dist-upgrade
```
```
pveversion
```
Update Debian Base Repositories to Bookworm
```
sed -i 's/bullseye/bookworm/g' /etc/apt/sources.list
```
Add Proxmox VE 8 Package Repository
For subscription repository:
```
echo "deb https://enterprise.proxmox.com/debian/pve bookworm pve-enterprise" > /etc/apt/sources.list.d/pve-enterprise.list
```
For no-subscription repository:
```
sed -i -e 's/bullseye/bookworm/g' /etc/apt/sources.list.d/pve-install-repo.list
```
Update Ceph Package Repository (for hyper-converged Ceph setups only)

For subscription repository:
```
echo "deb https://enterprise.proxmox.com/debian/ceph-quincy bookworm enterprise" > /etc/apt/sources.list.d/ceph.list
```
For no-subscription repository:
```
echo "deb http://download.proxmox.com/debian/ceph-quincy bookworm no-subscription" > /etc/apt/sources.list.d/ceph.list
```
Refresh Package Index
```
apt update
```
4. Upgrade to Debian Bookworm and Proxmox VE 8.0
```
apt dist-upgrade
```
5. Check Result & Reboot Into Updated Kernel

6. After the Proxmox VE Upgrade
Empty browser cache and/or force-reload the Web UI.
Check that all nodes in the cluster are up and running the latest package versions.

Tip: During Install it is possible that your Web URL could have been changed to see the new
login url put in the command below otherwise the default url is https://Your-Server-Ip:8006

```
pveversion -v
```

If it changed your Web Login URL then it is important that it resolves to your Server's IP address. 
To update that change 

Step 1: Restart the pve-cluster service:
After updating the /etc/hosts file, try restarting the pve-cluster service:
```
systemctl restart pve-cluster
```

Step 2: Check the status of the pve-cluster service:
Verify if the pve-cluster service is now running correctly:
```
systemctl status pve-cluster
```
Step 3: Change local IP address to the Public IP Address of your Server in the 

Example: 
127.0.1.1 vmi123456.contaboserver.net vmi123456
127.0.0.1 localhost

Should be changed to 
Your-Ip-Address vmi123456.contaboserver.net vmi123456
127.0.0.1 localhost

Step 4: Save and close the file:
Save the changes and close the file. In nano, press Ctrl+X, then Y, and finally Enter.

Restart the pve-cluster service:
After updating the /etc/hosts file, try restarting the pve-cluster service:
```
systemctl restart pve-cluster
```
Check the status of the pve-cluster service:
Verify if the pve-cluster service is now running correctly:
```
systemctl status pve-cluster
```
Verify the web interface:
Once the pve-cluster service is running, try accessing the Proxmox web interface again using the URL provided using
the following command 

```
echo "Your Proxmox VE login URL is: https://$(hostname -f):8006/"
```

Special Note: Port 8006 could be different if you specified if the URL shown in above command does not work, you
can also try the command below
```
PORT=$(grep -oP '(?<=:port\s)\d+' /etc/pve/corosync.conf); echo "Your Proxmox VE login URL is: https://$(hostname -f):$PORT/"
```

You May Want to Consider Using ProxMox Starter Scripts (use at your Own Risk) at https://tteck.github.io/Proxmox/

I like to use Proxmox VE Post Install
This script provides options for managing Proxmox VE repositories, including disabling the Enterprise Repo, adding or correcting PVE sources, enabling the No-Subscription Repo, adding the test Repo, disabling the subscription nag, updating Proxmox VE, and rebooting the system.

```
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/post-pve-install.sh)"
```

Proxmox Security Setup Guide

On any new Linux install, including Proxmox, it's crucial to implement basic security measures. This guide helps you:

Create a non-root user with sudo privileges
Install and configure AppArmor and Fail2ban
Disable root login over SSH
Change the SSH port
Enable automatic security upgrades
Ensure critical services start on boot and auto-restart if stopped

Key security services (Fail2ban, UFW, and AppArmor) are configured to:

Start automatically on system boot
Restart automatically if they are shut off for any reason

Additionally, unattended-upgrades is set up to automatically install security updates, keeping your system protected against known vulnerabilities.
Follow the steps below to secure your Proxmox installation:

## Step-by-Step Guide

### 1. Update and Upgrade the System

```bash
apt update && apt upgrade -y
```

### 2. Install Necessary Packages

```bash
apt install -y sudo apparmor apparmor-utils fail2ban ufw
```

### 3. Create a New Non-Root User with Sudo Privileges

```bash
read -p "Enter the new username: " NEW_USER
adduser $NEW_USER
usermod -aG sudo $NEW_USER
```

### 4. Configure Fail2Ban for SSH

```bash
cat <<EOF > /etc/fail2ban/jail.local
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
EOF
```

### 5. Enable and Configure Fail2Ban Service

```bash
systemctl enable fail2ban
systemctl start fail2ban
systemctl mask fail2ban.service
cat <<EOF > /etc/systemd/system/fail2ban.service
[Unit]
Description=Fail2Ban Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/fail2ban-server -f -s /var/run/fail2ban/fail2ban.sock
ExecStop=/usr/bin/fail2ban-client stop
ExecReload=/usr/bin/fail2ban-client reload
PIDFile=/var/run/fail2ban/fail2ban.pid
Restart=always

[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload
systemctl enable fail2ban.service
systemctl start fail2ban.service
```

### 6. Configure UFW (Uncomplicated Firewall)

```bash
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh
ufw allow 8006/tcp  # Proxmox web interface
ufw enable
```

### 7. Configure UFW Service

```bash
systemctl enable ufw
systemctl start ufw
cat <<EOF > /etc/systemd/system/ufw.service
[Unit]
Description=Uncomplicated Firewall
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/sbin/ufw enable
ExecStop=/usr/sbin/ufw disable
RemainAfterExit=yes
Restart=always

[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload
systemctl enable ufw.service
systemctl start ufw.service
```

### 8. Change SSH Port

```bash
read -p "Enter new SSH port number: " NEW_SSH_PORT
sed -i "s/^#*Port .*/Port $NEW_SSH_PORT/" /etc/ssh/sshd_config
ufw allow $NEW_SSH_PORT/tcp
ufw delete allow ssh
```

### 9. Disable Root Login over SSH

```bash
sed -i 's/^PermitRootLogin .*/PermitRootLogin no/' /etc/ssh/sshd_config
```

### 10. Restart SSH Service

```bash
systemctl restart sshd
```

### 11. Enable and Configure AppArmor

```bash
aa-enforce /etc/apparmor.d/*
systemctl enable apparmor
systemctl start apparmor
cat <<EOF > /etc/systemd/system/apparmor.service
[Unit]
Description=AppArmor initialization
DefaultDependencies=no
After=local-fs.target
Before=sysinit.target

[Service]
Type=oneshot
ExecStart=/usr/sbin/apparmor_parser -R /etc/apparmor.d/
ExecStart=/usr/sbin/apparmor_parser -r /etc/apparmor.d/
ExecStop=/usr/sbin/apparmor_parser -R /etc/apparmor.d/
RemainAfterExit=yes
Restart=always

[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload
systemctl enable apparmor.service
systemctl start apparmor.service
```

### 12. Verify Service Statuses

```bash
systemctl status fail2ban
systemctl status ufw
systemctl status apparmor
```
### 12. Enable Automatic Security Upgrades

To enable automatic security upgrades, run the following command:
```
apt install -y unattended-upgrades && dpkg-reconfigure -plow unattended-upgrades
```
This command will install unattended-upgrades and launch an interactive prompt to configure it. Choose 'Yes' when asked if you want to automatically download and install stable updates.

## Post-Installation Steps

After running these commands:

⚠️ ⚠️ 1. Review the changes and ensure you can still access your system before you closer your terminal ⚠️ ⚠️ 
2. Update your SSH client settings to use the new port you specified.
3. Once again make sure you can login with the new user you created before closing terminal!

## Important Notes 

- These commands should be run as root or with sudo privileges.
- Make sure to remember the new username and SSH port you set.


## Security Features Implemented

- System updates and upgrades
- New non-root user with sudo privileges
- Fail2Ban for protection against brute-force attacks
- UFW (Uncomplicated Firewall) configured for basic protection
- Changed SSH port (helps avoid automated attacks)
- Disabled root login over SSH
- AppArmor for application security
- Automatic start and restart of security services (in case they stop). 

Remember, security is an ongoing process. Regularly update your system and review your security measures to keep your Proxmox installation protected.


















