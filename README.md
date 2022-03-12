# Setup Splunk All-in-One in Azure VM

> Austin Lai | March 12th, 2022

---

<!-- Description -->

Basic scripts and key step to setup Setup Splunk All-in-One in Azure VM.

You may pair with IaC (Infrastructure as Code) such as Terraform and Ansible to build full automation of deploying Splunk All-in-One in Azure VM.

<!-- /Description -->

<br />

## Table of Contents

<!-- TOC -->

- [Setup Splunk All-in-One in Azure VM](#setup-splunk-all-in-one-in-azure-vm)
    - [Table of Contents](#table-of-contents)
    - [Bash Scripts to create Splunk All-in-One in Azure VM.](#bash-scripts-to-create-splunk-all-in-one-in-azure-vm)

<!-- /TOC -->

## Bash Scripts to create Splunk All-in-One in Azure VM.

**Disclaimer:**

- Script does not take security into consideration, you may need to harden the system according to NIST or CIS hardening guide.
- The build only for **development usage** and **NOT** for **PRODUCTION**

```bash
sudo yum update -y
sudo systemctl enable --now cockpit.socket

# find disk
sudo lsblk -o NAME,HCTL,SIZE,MOUNTPOINT | grep -i "sd"
sudo lvmdiskscan

# check disk space
sudo df -H

# Expand root space
sudo lvextend -L +25G /dev/mapper/rootvg-rootlv
sudo lvextend -L +5G /dev/mapper/rootvg-homelv
sudo lvextend -L +5G /dev/mapper/rootvg-tmplv

# Online resize
sudo xfs_growfs /dev/mapper/rootvg-rootlv
sudo xfs_growfs /dev/mapper/rootvg-homelv
sudo xfs_growfs /dev/mapper/rootvg-tmplv

# Download splunk installation file
wget -O splunk-8.2.4-87e2dda940d1-linux-2.6-x86_64.rpm 'https://download.splunk.com/products/splunk/releases/8.2.4/linux/splunk-8.2.4-87e2dda940d1-linux-2.6-x86_64.rpm'

# copy the installation file to tmp and change the permission
cp splunk-8.2.4-87e2dda940d1-linux-2.6-x86_64.rpm /tmp/
chmod 777 /tmp/splunk-8.2.4-87e2dda940d1-linux-2.6-x86_64.rpm

# Install splunk
sudo rpm -i /tmp/splunk-8.2.4-87e2dda940d1-linux-2.6-x86_64.rpm

# Change password for user splunk
sudo passwd splunk

# Export SPLUNK_HOME or set Splunk home directory.
# Setup splunk environment variable
echo export SPLUNK_HOME=/opt/splunk >> ~/.bashrc
echo source /opt/splunk/bin/setSplunkEnv >> ~/.bashrc

# Set ulimit
ulimit -n 64000
ulimit -u 16000
ulimit -d 12000
ulimit -f 64000

# Setup basic firewall
sudo firewall-cmd --permanent --zone=public --add-port=8089/tcp
sudo firewall-cmd --permanent --zone=public --add-port=8000/tcp
sudo firewall-cmd --reload

# Change permission of SPLUNK_HOME and add splunk user to wheel
sudo chown -R splunk:splunk $SPLUNK_HOME
sudo usermod -aG wheel splunk

# Switch user to splunk and start splunk with enable boot start
su - splunk
sudo ./bin/splunk start --accept-license 
sudo ./bin/splunk enable boot-start
```

<br />

---

> Do let me know any command or step can be improve or you have any question you can contact me via THM message or write down comment below or via FB
