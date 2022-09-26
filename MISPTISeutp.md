## MISP Open Source Threat Intelligence Platform to Microsoft Sentinel - Setup ##

After the announcement of the free, LIMO Threat Intelligence injestion was deemed to be End of Lofe. Many of us have been after a replacement. This guide will associate a cost, check cost workbook, and a VM cost. However, it will have an automated process of TI into your Sentinel instance. See MISP [here](https://www.misp-project.org/).


*Pre-req* - Create a Ubuntu VM, I am using Azure for this use case. Don't need anything crazy to keep the cost low for this use case. Being this is owned by the SecOps team, the VM lives within my SecOps subscription in Azure.

*Disclaimer* - You can use the default account that is created when you stand up the server, initially or create a user called MISP. Regardless, a user account named 'MISP' will be created during the install. I chose to use my default admin account and then change the password later for RDP access. 

## This installs the MISP TI Platform

```
# Ssh to the new Ubuntu VM. The following commands can be copied and pasted into the ssh session.

# Update/Upgrade System, if needed.
sudo apt-get update -y && sudo apt-get upgrade -y

//# Create a Local Password for MISP User - If MISP was created already. If not, go to next step (you will do this later). 
sudo passwd misp

# Install Firefox Browser - this needs to be installed to configure MISP to Sentinel.
sudo apt install firefox -y

# Please check the installer options first to make the best choice for your install 
wget -O /tmp/INSTALL.sh https://raw.githubusercontent.com/MISP/MISP/2.4/INSTALL/INSTALL.sh
bash /tmp/INSTALL.sh

# This will install MISP Core - Install will pause to create user MISP, chose yes to run as MISP. 
wget -O /tmp/INSTALL.sh https://raw.githubusercontent.com/MISP/MISP/2.4/INSTALL/INSTALL.sh
bash /tmp/INSTALL.sh -c
```
# Copy the content from the output in a notepad/shared file, temporarily. You need the AuthKey.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/MISPafterinstall1.png)

```
# Now, create a local password for MISP user. I chose to wait until AFTER the script ends.
sudo passwd misp

# Install Xfce4 Desktop.
sudo apt-get update
sudo DEBIAN_FRONTEND=noninteractive apt-get -y install xfce4
sudo apt install xfce4-session

# Install XRDP.
sudo apt-get -y install xrdp
sudo systemctl enable xrdp
sudo adduser xrdp ssl-cert
echo xfce4-session >~/.xsession
sudo ufw allow 3389
sudo service xrdp restart

# Reboot
sudo systemctl reboot

```

## Configure MISP TI Platform

# RDP to whichever user has xrdp permissions to the Ubuntu VM

log into the server as seen below

enter default password will prompt you to change the password

enable FEEDS

retrieve your key from earlier. if you forgot - do this cat /home/misp/MISP-authkey.txt

## Create AAD App Reg
# Open the Application Registration Portal and click New registration on the menu bar.
# Enter a name, and choose Register, other options can be left with their defaults.
# Note down the Application (client) ID and Directory (tenant) ID. You will need to enter these into the script’s configuration file.
# Under Certificates & secrets, click New client secret enter a description and click Add. A new secret will be displayed. Copy this for later entry into the script.
# Under API permissions, choose Add a permission > Microsoft Graph.
# Under Application Permissions, add ThreatIndicators.ReadWrite.OwnedBy.

## Enable the Sentinel Connector
Open your Azure Sentinel workspace, click ‘Data connectors’ and then look for the ‘Threat Intelligence Platforms’ connection. Open the connector and click Connect.

## Setup the script for MISP-Sentinel API Calls

```
sudo apt-get install python3-venv
sudo python3 -m venv mispToSentinel
cd mispToSentinel
source bin/activate
git clone https://github.com/microsoftgraph/security-api-solutions
cd security-api-solutions/Samples/MISP/
pip install -r requirements.txt
nano config.py
```

## Configure the config.py file.


