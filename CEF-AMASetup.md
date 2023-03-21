## Stream Comment Event Format (CEF) with Azure Monitor Agent (the helping hand guide). Authored by: Michael Crane and Lorenzo Ireland. ##

Many folks using Sentinel have issues with clarity around the Common Event Format (CEF) via AMA and rightfully so. This article deems to clear any confusion in both Azure Commercial and Goverment tenants. See CEF-AMA [here](https://learn.microsoft.com/en-us/azure/sentinel/connect-cef-ama).

*Disclaimer* - In Microsoft Sentinel, the CEF connector is only giving you instructions to create a DCR and looking for the ingestion on a flag. It is NOT a true connector. You will have manual work to do and this solution does work in Azure Government. 

*Pre-req* - Create a Ubuntu VM, I am using Azure for this use case. Don't need anything crazy to keep the cost low for this use case. Being this is owned by the SecOps team, the VM lives within my SecOps subscription in Azure.

## CEF Setup on Ubuntu 22.04

```
# Ssh to the new Ubuntu VM. The following commands can be copied and pasted into the ssh session.

# Update/Upgrade System, if needed.
sudo apt-get update -y && sudo apt-get upgrade -y

# Reboot
sudo systemctl reboot

# Check if Python 2.7/3 is installed and syslog-ng or rsyslog (rsyslog by default) 
sudo apt install python3
sudo apt install rsyslog

```

## Creating the Data Collection Rule.

The DCR rule has to be in place first. Just create a simple syslog DCR and call it a day, as we will reconfiguring it later.

*Instructions* - [here](https://learn.microsoft.com/en-us/azure/sentinel/forward-syslog-monitor-agent)

## Run the following on your CEF machine, AFTER you have created the DCR rule. 

```
# Azure Commercial. The installation script configures the rsyslog or syslog-ng daemon to use the required protocol and restarts the daemon
sudo wget -O Forwarder_AMA_installer.py https://raw.githubusercontent.com/Azure/Azure-Sentinel/master/DataConnectors/Syslog/Forwarder_AMA_installer.py
sudo python3 Forwarder_AMA_installer.py 

```
# Edit the rsyslog or syslog-ng conf file. 
On the Ubuntu server you will see it has been changed to CEF by uncommented modules and inputs. Confirm changes at: 'cat /etc/rsyslog.conf'
![](https://github.com/Cyberlorians/uploadedimages/blob/main/cefmagrsyslog.png)

# Now, create a local password for MISP user. I chose to wait until AFTER the script ends.
sudo passwd misp

# Install Xfce4 Desktop.
sudo DEBIAN_FRONTEND=noninteractive apt-get -y install xfce4
sudo apt install xfce4-session

# Install XRDP.
sudo apt-get -y install xrdp
sudo systemctl enable xrdp
sudo ufw allow 3389
sudo service xrdp restart
sudo adduser misp ssl-cert
echo xfce4-session >~/.xsession
sudo service xrdp restart

# Install Firefox Browser - this needs to be installed to configure MISP to Sentinel Feeds.
sudo apt install firefox -y

```

# Configure MISP TI Platform

1. RDP to the Ubuntu server with the MISP user created during the MISP install.

2. Log into the server as seen below. The default username and password were provided earlier. 

![](https://github.com/Cyberlorians/uploadedimages/blob/main/MISPlogin.png)

3. Enter default password will prompt you to change the password

4. After logging in, navigate to 'Sync Actions'>'List Feeds'. Select both default feeds, enable and cache. Once done, click 'Fetch and store all feed data'. These two are by default, you can add more from the proper MISP website [here](https://www.misp-project.org/feeds/).

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

## Setup the script for MISP-Sentinel API Calls. RDP as MISP first. 

1. Run the below script.

```
sudo apt-get install python3-venv
python3 -m venv mispToSentinel
cd mispToSentinel
source bin/activate
git clone https://github.com/microsoftgraph/security-api-solutions
cd security-api-solutions/Samples/MISP/
pip3 install requests requests-futures pymisp
nano config.py
```
2. After opening config.py, edit the file as seen below, replacing with your information. Remember your Auth Key is at: cat /home/misp/MISP-authkey.txt

![](https://github.com/Cyberlorians/uploadedimages/blob/main/mispconfig.png)

3. Use CTRL+O to write-out, save and CTRL-X to exit.

4. Run the following now to sync the feeds to Sentinel.
```
python3 script.py
```
Confirm ingestion by navigating to the TI workbook in Sentinel.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/MISPsetup7.png)

## Chron job setup.

Below is a CRONTAB entry example of running the script every day at midnight. You can use the generator [here](https://crontab-generator.org/).
```
* 0 * * * home/misp/mispToSentinel/security-api-solutions/Samples/python3 script.py >/dev/null 2>&1
```

