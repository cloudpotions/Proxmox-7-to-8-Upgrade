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

(Maybe) Fix the hostname resolution - Can you not log into your site ? When I did this the first time the upgrade changed my host name cause I selected yes during the upgrade process for the server to hose websites and it auto-populated to vmi112345.contaboserver.net ... I included a fix for this at the very bottom of this post if you made the same mistake!


You May Want to Consider Using ProxMox Starter Scripts (use at your Own Risk) at https://tteck.github.io/Proxmox/

I like to use Proxmox VE Post Install
This script provides options for managing Proxmox VE repositories, including disabling the Enterprise Repo, adding or correcting PVE sources, enabling the No-Subscription Repo, adding the test Repo, disabling the subscription nag, updating Proxmox VE, and rebooting the system.

```
-c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/post-pve-install.sh)"
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

```
apt update && apt upgrade -y
```

### 2. Install Necessary Packages

```
apt install -y sudo apparmor apparmor-utils fail2ban ufw
```

### 3. Create a New Non-Root User with Sudo Privileges

```
sudo bash-c 'read -p "Enter new username: " u && adduser --gecos "" $u && usermod -aG sudo $u && echo "User $u created and added to sudo group."'
```
Also Make Sure you add this Non-Root User to ProxMox
Add the non-root user to necessary groups and configure as a Proxmox admin:
```
sudo usermod -aG sudo,adm NonRootAdminUsername && sudo pveum user add NonRootAdminUsername@pam -groups admin && (echo -e "user:NonRootAdminUsername@pam:1:0:::NonRootAdminUsername@pam:" | sudo tee -a /etc/pve/user.cfg > /dev/null) && sudo systemctl restart pvedaemon pveproxy
```
PS - Make sure you also add your non root user on the ProxMox Users and it will be in the Linux Pam Standar Authentication. 
Click on Data Center --> Users --> Add User
Also just to be safe click on Permissions --> Select path as "/" then select User then Select Administrator as the Role



### 4. Configure Fail2Ban for SSH

```
echo '[sshd]\nenabled = true\nport = ssh\nfilter = sshd\nlogpath = /var/log/auth.log\nmaxretry = 3\nbantime = 3600' | sudo tee /etc/fail2ban/jail.local > /dev/null
```

### 5. Enable and Configure Fail2Ban Service

```
sudo apt install -y fail2ban && echo '[sshd]\nenabled = true\nport = ssh\nfilter = sshd\nlogpath = /var/log/auth.log\nmaxretry = 3\nbantime = 3600' | sudo tee /etc/fail2ban/jail.local > /dev/null && sudo systemctl unmask fail2ban && sudo systemctl enable fail2ban && sudo systemctl start fail2ban

```

### 6. Configure UFW (Uncomplicated Firewall)

```
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh
ufw allow 8006/tcp  # Proxmox web interface
ufw enable
```

### 7. Configure UFW Service

```
sudo ufw enable && sudo ufw default deny incoming && sudo ufw default allow outgoing && sudo ufw allow ssh && sudo ufw allow 8006/tcp && sudo ufw status verbose
```

### 8. Change SSH Port

```
read -p "Enter new SSH port number: " NEW_SSH_PORT && sudo sed -i "s/^#*Port .*/Port $NEW_SSH_PORT/" /etc/ssh/sshd_config && sudo ufw allow $NEW_SSH_PORT/tcp && sudo ufw delete allow ssh
```

### 9. Disable Root Login over SSH

```
sed -i 's/^PermitRootLogin .*/PermitRootLogin no/' /etc/ssh/sshd_config
```

### 10. Restart SSH Service

```
systemctl restart sshd
```

### 11. Enable and Configure AppArmor

```
sudo systemctl enable apparmor && sudo systemctl start apparmor && sudo aa-status
```

### 12. Verify Service Statuses

```
systemctl is-active fail2ban && systemctl is-active ufw && systemctl is-active apparmor
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


HOSTNAME FIX FROM UPGRADE

When I upgraded the first time I ansered yes to web services and it automatically changed my hostname to vmi112345.contaboserver.net

This caused some problems! Here is how I fixed it 

Set the Correct Hostname
Set the hostname to the desired value. For this example, let's use proxmox-node as the hostname.

```
sudo hostnamectl set-hostname proxmox-node
```
Update the /etc/hosts File
Edit the /etc/hosts file to ensure it has the correct entries for hostname resolution.

```
sudo nano /etc/hosts
```
Ensure it has the following entries (where 123.456.789 is your IP address)

127.0.0.1 localhost <br />
123.456.789 proxmox-node

Update the Cloud-Init Templates (if necessary)
If Cloud-Init is overwriting your /etc/hosts file, you might need to update the templates to reflect the correct hostname.

Edit the Cloud-Init Templates
Edit /etc/cloud/templates/hosts.debian.tmpl:

```
sudo nano /etc/cloud/templates/hosts.debian.tmpl
```
Make sure the part below lookos like this where 123.456.789 is your IP address (most likely you will only need to edit one line). After you are done editing save and exit. 

127.0.1.1 {{fqdn}} {{hostname}} <br />
127.0.0.1 localhost <br />
#123.456.789 proxmox-node <br />

Now make sure you Edit the /etc/cloud/cloud.cfg file to manage the hostname properly:

```
sudo nano /etc/cloud/cloud.cfg
```
Ensure preserve_hostname is set to true:

Example: preserve_hostname: true

Restart Necessary Services <br />
Restart the necessary services to apply the changes:

```
sudo systemctl restart pve-cluster pvedaemon pveproxy
```

sudo systemctl status pve-cluster pvedaemon pveproxy

Verify the web interface: <br />
Once the pve-cluster service is running, try accessing the Proxmox web interface again using the URL provided using <br />
the following command 

```
echo "Your Proxmox VE login URL is: https://$(hostname -f):8006/"
```
Reboot Your System
```
sudo reboot
```
Special Note: Port 8006 could be different if you specified if the URL shown in above command does not work, you
can also try the command below
```
PORT=$(grep -oP '(?<=:port\s)\d+' /etc/pve/corosync.conf); echo "Your Proxmox VE login URL is: https://$(hostname -f):$PORT/"
```


















