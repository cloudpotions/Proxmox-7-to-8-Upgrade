# Proxmox-7-to-8-Upgrade
 This guide provides a clear and concise upgrade path from Proxmox VE 7 to 8, ensuring it's easy to copy and follow the commands. This guide was inspired by the official instructions from the[Proxmox VE Wiki.](https://www.paypal.com/donate/?hosted_button_id=SCM4T6CSCP5JS)

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


















