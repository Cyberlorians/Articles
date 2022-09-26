## MISP Open Source Threat Intelligence Platform to Microsoft Sentinel - Setup ##

After the announcement of the free, LIMO Threat Intelligence injestion was deemed to be End of Lofe. Many of us have been after a replacement. This guide will associate a cost, check cost workbook, and a VM cost. However, it will have an automated process of TI into your Sentinel instance. See MISP [here](https://www.misp-project.org/).


*Pre-req* - Create a Ubuntu VM, I am using Azure for this use case. Don't need anything crazy to keep the cost low for this use case. Being this is owned by the SecOps team, the VM lives within my SecOps subscription in Azure.

## This installs the MISP TI Platform


```
# Ssh to the new Ubuntu VM. The following commands can be copied and pasted into the ssh session.

# Update/Upgrade System, if needed
sudo apt-get update -y && sudo apt-get upgrade -y

# Create a Local Password for MISP User
sudo passwd misp

# Install XRDP
sudo apt install xrdp -y
sudo adduser xrdp ssl-cert
sudo systemctl restart xrdp
sudo ufw allow 3389

# Install Xfce4 Desktop
sudo DEBIAN_FRONTEND=noninteractive apt-get -y install xfce4

# Install Firefox Browser
sudo apt install firefox -y

# Install MISP with install.sh
wget -O /tmp/INSTALL.sh https://raw.githubusercontent.com/MISP/MISP/2.4/INSTALL/INSTALL.sh
bash /tmp/INSTALL.sh -c

```
