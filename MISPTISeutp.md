## MISP Open Source Threat Intelligence Platform to Microsoft Sentinel. Authored by: Michael Crane and Matt Larkin ##

After the announcement of the free, LIMO Threat Intelligence injestion was deemed to be End of Lofe. Many of us have been after a replacement. This guide will associate a cost, check cost workbook, and a VM cost. However, it will have an automated process of TI into your Sentinel instance. See MISP [here](https://www.misp-project.org/).


*Pre-req* - Create a Ubuntu VM, I am using Azure for this use case. Don't need anything crazy to keep the cost low for this use case. Being this is owned by the SecOps team, the VM lives within my SecOps subscription in Azure.

*Disclaimer* - You can use the default account that is created when you stand up the server, initially or create a user called MISP. Regardless, a user account named 'MISP' will be created during the install. I chose to use my default admin account and then change the password later for RDP access. 

## This installs the MISP TI Platform

```
# Ssh to the new Ubuntu VM. The following commands can be copied and pasted into the ssh session.

# Update/Upgrade System, if needed.
sudo apt-get update -y && sudo apt-get upgrade -y

# Install Firefox Browser - this needs to be installed to configure MISP to Sentinel.
sudo apt install firefox -y

# Please check the installer options first to make the best choice for your install 
wget -O /tmp/INSTALL.sh https://raw.githubusercontent.com/MISP/MISP/2.4/INSTALL/INSTALL.sh
bash /tmp/INSTALL.sh

# This will install MISP Core - Install will pause to create user MISP, chose yes to run as MISP. 
wget -O /tmp/INSTALL.sh https://raw.githubusercontent.com/MISP/MISP/2.4/INSTALL/INSTALL.sh
bash /tmp/INSTALL.sh -c
```

## Copy the content from the output in a notepad/shared file, temporarily. You need the AuthKey.

This key can be pulled later by:  cat /home/misp/MISP-authkey.txt

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

# Configure MISP TI Platform

1. RDP to whichever user has xrdp permissions to the Ubuntu VM

2. Log into the server as seen below. The default username and password were provided earlier. 

![](https://github.com/Cyberlorians/uploadedimages/blob/main/MISPlogin.png)

3. Enter default password will prompt you to change the password

4. Enable FEEDS and caching. These two are by default, you can add more from the proper MISP website [here](https://www.misp-project.org/feeds/).

![](https://github.com/Cyberlorians/uploadedimages/blob/main/MISPsetup1.png).

5. Retrieve your key from earlier. if you forgot - cat /home/misp/MISP-authkey.txt, hang on to it!

## Create AAD App Reg in Azure AD.

1. Open the Application Registration Portal and click New registration on the menu bar.
2. Enter a name, and choose Register, other options can be left with their defaults.
3. Note down the Application (client) ID and Directory (tenant) ID. You will need to enter these into the script’s configuration file.
4. Under Certificates & secrets, click New client secret enter a description and click Add. A new secret will be displayed. Copy this for later entry into the script.
5. Under API permissions, choose Add a permission > Microsoft Graph.
6. Under Application Permissions, add ThreatIndicators.ReadWrite.OwnedBy.
![](https://github.com/Cyberlorians/uploadedimages/blob/main/MISPsetup2.png).

## Enable the Sentinel Connector
Open your Azure Sentinel workspace, click ‘Data connectors’ and then look for the ‘Threat Intelligence Platforms’ connection. Open the connector and click Connect.
![](https://github.com/Cyberlorians/uploadedimages/blob/main/MISPsetup3.png)

## Setup the script for MISP-Sentinel API Calls

1. Run the below script.

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
2. After opening config.py, edit the file as seen below, replacing with your information.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/MISPsetup5.png)

3. Use CTRL+O to write-out, save and CTRL-X to exit.

4. Run the following now to sync the feeds to Sentinel.
```
python script.py
```

## Chron job setup.


