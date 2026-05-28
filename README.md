# Microsoft-Azure-Sentinel-Defender-Lab-SIEM-1
Microsoft Azure &amp; Sentinel Lab (SIEM) 1

# Objective

Build a SOC environment with a central log aggregation/alerting hub (Microsoft Sentinel as your SIEM), endpoint protection (Microsoft Defender for Endpoint), and a few vulnerable/target VMs to generate realistic telemetry, using Microsoft Azure as the host for our target environment.

# Itinerary

<img width="960" height="842" alt="Screenshot 2026-05-13 193551" src="https://github.com/user-attachments/assets/776af1fd-3739-4244-95e8-d8075c2c450a" />

**Phase 1 — Foundation**
Set up your Azure subscription and resource group. Everything goes in one Resource Group (e.g., rg-soc-lab) in a single region (East US or West US 2 are cheapest). This keeps networking simple and egress costs low.
**Key resources to deploy:**
- Log Analytics Workspace (LAW) — this is the data backbone for Microsoft Sentinel
- Microsoft Sentinel — enable it on top of the LAW (free 90-day trial tier)
- Azure Virtual Network (10.0.0.0/16) with two subnets: attacker and victim

**Phase 2 — Endpoints & EDR**
Deploy 2–3 small VMs and onboard them to Microsoft Defender for Endpoint (MDE):

- Windows 11 VM (victim) — B2s SKU (~$35/mo, ~$1.17/day). Onboard to MDE via the Defender portal. This is the primary endpoint to defend and attack.
- Ubuntu 22.04 VM (attacker/analyst) — B1s SKU (~$8/mo). Install tools like Nmap, Metasploit, and use it to generate attack telemetry against the Windows VM.
- Windows Server 2022 VM (optional, for AD attacks) — B2s SKU if budget allows.

**Cost tip**: Auto-shutdown VMs at night via Azure Automation. Running 8 hrs/day instead of 24 cuts VM costs by 67%.

**Phase 3 — SIEM Configuration**
1. Connect data sources to Microsoft Sentinel:

- _MDE connector_ — streams endpoint alerts, device events, and advanced hunting data
- _Azure Activity logs_ — tracks changes to your own Azure environment (great for cloud detection)
- _Windows Security Events via AMA agent_ — login events, process creation (Event ID 4688), etc.
- _Sysmon_ — install on the Windows VM for rich process/network telemetry (free, essential)
- _Linux Syslog_ — from your Ubuntu VM

2. Build your first Analytic Rules (detection logic):
- Brute force detection (multiple failed logins)
- New local admin account creation
- Base64-encoded PowerShell execution
- Lateral movement via PsExec

**Phase 4 — Attack Simulation & Detection**
_Run structured attack scenarios and hunt for them in Sentinel:_

- MITRE ATT&CK T1059 — Execute encoded PowerShell from the attacker VM, detect in Sentinel
- T1003 — Credential dumping with Mimikatz, detect via Defender alerts
- T1046 — Network scanning with Nmap, detect via Azure NSG flow logs
- T1136 — Create a local admin, detect via Sentinel rule
- Use KQL (Kusto Query Language) to hunt across SecurityEvent, DeviceProcessEvents, DeviceNetworkEvents tables

**Phase 5 — Dashboards & Reporting**
- Build a Sentinel Workbook (custom dashboard) showing alert trends, top endpoints, and MITRE coverage
- Set up Playbooks (Logic Apps) for automated response — e.g., auto-isolate a VM when a high-severity alert fires
- Document your detections and write a mini threat-hunt report

# STEPS

# Phase 1 - Foundation

## Make a Microsoft Account
- Choose to create one under Microsoft Azure Website either as personal, or under an organization.
- Make sure to choose the $200 credit option. Once that is finished with the $200, it will prompt to go to a pay account.

## Welcome to Azure - Azure Home Screen

<img width="1329" height="916" alt="1_Welcometoazure" src="https://github.com/user-attachments/assets/6c0c1cb9-3ef1-493b-841e-9edd215a5335" />

## Log Analytics Workspace (LAW)
A Log Analytics workspace is a centralized data repository in Azure Monitor that collects, stores, and analyzes log data from Azure resources, non-Azure systems, and applications. It acts as a foundational, structured store using Kusto Query Language (KQL) to enable monitoring, troubleshooting, and security analytics, acting as a crucial component for services like Microsoft Sentinel and Defender for Cloud.

- Go to Log Analytic Work spaces, and create a workspace.

<img width="1173" height="431" alt="Screenshot 2026-04-28 215334" src="https://github.com/user-attachments/assets/e72d76d9-bae4-4ada-9bd5-cf823cef29a0" />

- Enter in names and descriptions, and click on review + create.

<img width="784" height="886" alt="Screenshot 2026-04-28 215505" src="https://github.com/user-attachments/assets/c41e1d3a-7485-4841-90d0-ce195f8d7326" />

- After a minute of loading, click on create to create it.

_Your deployment is complete._

<img width="1192" height="676" alt="Screenshot 2026-04-28 215816" src="https://github.com/user-attachments/assets/f8240065-7b18-4dbd-9031-88de782324f7" />

## Billing and Cost Management
- Because we want to make use of our Azure free trial, we want to be able make sure we do not exceed our spening limit.

_Azure Budjet Alert_
- go to Cost Management → Budgets
- create a $150 budget with alerts at 80% and 100%. This gives you an email warning before you overspend.

<img width="1649" height="274" alt="Screenshot 2026-05-13 161850" src="https://github.com/user-attachments/assets/a857efbd-c70d-44cf-b1b5-c16a374d950c" />

_Daily Cap on the LAW_
- Go to our LAW we just created, can access from 'Resource Groups'.
- Go to settings, Usage and estimated Costs, select 'Daily Cap'.
- Set it to something like 2 GB/day.
- If a misconfigured agent starts spamming verbose logs, this prevents runaway ingestion costs.

<img width="1091" height="551" alt="Screenshot 2026-05-13 162503" src="https://github.com/user-attachments/assets/76ca1af5-29a6-4547-bff8-31dc30eabf44" />

## Azure Virtual Network
_We want to create a virtual network with a subnet so we can have our machines connect within our network._
1. Search for and go to Virtual Networks.
2. Click create
3. Enter in information
   1. choose your subsricption
   2. Select our existing resource group we first made in this lab
   3. Enter in a unique name.
   4. select a physical region (if in US, US EAST or US WEST 2 might be the cheapest for credits.)
4. Define the IPv4 address Space or use the default (10.0.0.0/16). Then, add a subnet with similiar range, or use the default.
5. Click Review + create

<img width="844" height="863" alt="Screenshot 2026-05-13 163720" src="https://github.com/user-attachments/assets/e7fc9054-900f-4fd1-a0ce-cd9165c28ada" />

<img width="1649" height="860" alt="Screenshot 2026-05-13 164139" src="https://github.com/user-attachments/assets/f2f31451-78b9-4f8e-a5c3-daf6c3880211" />

<img width="1252" height="493" alt="Screenshot 2026-05-13 165032" src="https://github.com/user-attachments/assets/f64ccb40-84aa-4841-8c23-1737fab928a8" />

## Microsoft Sentinel
- Next, search for Microsoft Sentinal, and click on create.
- Select our workspace we just created.
- click Add

### Install tools and solutions

<img width="1562" height="835" alt="Screenshot 2026-05-13 194902" src="https://github.com/user-attachments/assets/7ef288dd-490a-4216-9661-aec9912cf64d" />

- Go to Content Management → Content Hub. This is where you install pre-built detection packs. Once clicking on conent hub, it will actually direct you to the Micsrofot Defender Portal. We will then need to connect our workspace to Defender.
1. Go to the Microsoft Defender portal and sign in.
2. Select System > Settings > Microsoft Sentinel > Connect a workspace.
3. Select the workspaces you want to connect and select Next.
4. Select the Primary workspace.
5. Read and understand the product changes associated with connecting your workspace.
6. Select Connect.

<img width="1832" height="891" alt="Screenshot 2026-05-13 200603" src="https://github.com/user-attachments/assets/e77bb0a4-bb8b-4247-b88d-6043fea6aeb9" />

_After your workspace is connected, the banner on the Home page shows that your environment is ready. The Home page is updated with new sections that include metrics from Microsoft Sentinel, like the number of data connectors and automation rules._

## Connect Microsoft Defender XDR
- Go back to the Azure Portal → Microsoft Sentinel → your workspace
- Click Configuration → Data Connectors
- Find Microsoft Defender XDR, its status should say Connected with a green indicator. If not, open connector page (it should have connected when we connected the workspace to Defender in Defender Portal).

<img width="1405" height="798" alt="Screenshot 2026-05-13 200838" src="https://github.com/user-attachments/assets/ac29a137-bb58-4469-b95a-e5c7c53c3cbc" />

- Under Sentinel, go to Logs
- Enter and run this query. It may not have anything, which is fine, since it means no incidents have fired yet. We still have to onboard our VM's.

## Connect to Azure Activity
So. . . as of July of 2025, Sentinel workspaces automatically connect to the defender portal. So from here on out, we will be accessing Sentinel and other feature from within Microsoft Defender.
- under Microsoft Defender:
- Go to Microsoft Sentinel → Configurations → Data connectors
- Search for "Azure Activity"
- If it doesn't appear, go to Content management → Content hub, search "Azure Activity", install it, then come back and refresh Data connectors.

<img width="1903" height="916" alt="Screenshot 2026-05-13 213021" src="https://github.com/user-attachments/assets/75c28b66-7201-42ca-9026-9a3ab9d273ca" />
- Open the connector and view the information

## Microsoft Defender XDR
I just want to note on here that earlier, we connected Defender XDR when we connected our workspace after going to the Defender portal FROM Sentinel. So, we should be good to go. Defender XDR is enabled.

## Install the other tools
_Install the following from Conent hub under Content management in Defender:_
- Windows Security Events — detection rules for common Windows attack patterns
- Endpoint Threat Protection Essentials — provides content to monitor, detect and investigate threats related to windows machines. The solution looks for things like suspicious commandlines, PowerShell based attacks, LOLBins, registry manipulation, scheduled tasks etc. which are some of the most commonly used techniques by attackers when targeting endpoints.
- Microsoft Entra ID

## MITRE ATT&CK Sollution — maps your coverage visually
This should actually be installed and enabled already.
- In Defender, You can verify this by going to Threat Management > MITRE ATT&CK.
- You can open any incident or alert in the Defender portal; the right-hand panel displays mapped MITRE ATT&CK techniques.
- Regardless, we should be good to deploy our virtual machines.

<img width="1553" height="811" alt="Screenshot 2026-05-13 215158" src="https://github.com/user-attachments/assets/2e6ec46b-c55f-4759-90fb-6eace4a14e1e" />

## Enable the Azure Activity Connector

- In Microsoft Defender, Go to Sentinel → Data Connectors → Azure Activity
- Click Open connector page
- Click the blue Launch Azure Policy Assignment Wizard button and follow the prompts to route your subscription's activity log into your workspace.
- Alternatively, some tenants see a simple Connect button — click that if present
_This populates the AzureActivity table and you'll have your first real data source running within minutes._

## Add Rules in our Content hub:
- Go into Azure Activity > Manage > add some rules to notify us of certain activities.
- Go into Entra ID connector and so the same.

<img width="1418" height="760" alt="Screenshot 2026-05-20 211618" src="https://github.com/user-attachments/assets/caba9be4-a685-47f4-a80d-5f73903ea2e0" />

**_We have our Virtual Network setup. Now, we should be able to add virtual machines._**

**# Phase 2 - Endpoints and EDR**

## Add Virtual Machines

### 1. Windows 11 Pro
- In Azure, go to Virtual Machines section
- click create
- Create: Click the "+ Create" button and select "Azure virtual machine".
- Configure Basics: Fill in essential details in the "Basics" tab:Subscription:
- Select your Azure subscription.
- Resource Group: Select or create a new resource group.
- VM Name: Give your VM a unique name.
- Region: Choose a geographical location (e.g., East US).
- Image: Select the operating system (e.g., Windows Server, Ubuntu).
- Size: Choose the VM's CPU and memory configuration. Best to keep it simple, like Standard D2als v7 (2 vcpus, 4 GiB memory).
- Set Credentials: Create an administrator username and password (or SSH key).
- Inbound Ports: Configure rules (e.g., allow RDP for Windows or SSH for Linux).
- Make sure to enable basic plan for free UNDER Microsoft Defender for Cloud.
- Review + create: Click to validate settings, then click "Create" to deploy.
_Our Windows 11 Pro Machine is deployed, This will mostly serve as our victom machine._

### Setup configuration
#### Setup sysmon

Sysmon is a free Microsoft tool that writes far richer event logs than Windows does by default. It's the difference between seeing "a process ran" and seeing the full command line, parent process, network connections, and file hashes. It's essential for realistic SOC work.
- Inside the VM via RDP, open PowerShell as Administrator and run:

##### Create a working folder
New-Item -ItemType Directory -Path "C:\Tools" -Force

##### Download Sysmon
Invoke-WebRequest -Uri "https://download.sysinternals.com/files/Sysmon.zip" -OutFile "C:\Tools\Sysmon.zip"

##### Extract it
Expand-Archive -Path "C:\Tools\Sysmon.zip" -DestinationPath "C:\Tools\Sysmon"

##### Download the SwiftOnSecurity community config (industry standard for labs)
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml" -OutFile "C:\Tools\Sysmon\sysmonconfig.xml"

##### Install Sysmon with the config
C:\Tools\Sysmon\Sysmon64.exe -accepteula -i C:\Tools\Sysmon\sysmonconfig.xml

- run the following to verify it is running
- Get-Service sysmon*

#### Data Rule with Monitor in Azure
_We want to create a data rule for events_
This is what gets Windows Security Events and Sysmon logs flowing into your Log Analytics Workspace and Sentinel. Back in the Azure Portal (portal.azure.com):

- Search for Monitor → Data Collection Rules → Create
- Fill in the basics:

<img width="674" height="260" alt="Screenshot 2026-05-19 152839" src="https://github.com/user-attachments/assets/8fd9979d-a850-4172-8ac0-290d312ef420" />

- On the Resources tab, click Add resources and select your Windows 11 VM
- On the Collect and deliver tab, click Add data source and add:

- Data source type: Windows Event Logs
- Add these channels manually under "Custom" if they don't appear automatically:

- Security — set to All
- Microsoft-Windows-Sysmon/Operational — set to All
- System — set to Warning and above

- Set the Destination to your Log Analytics Workspace (law-soc-lab-01)
- Review + Create

- we should eventually see event id's underSentinel → Advanced Hunting:
- run this KQL query:
- SecurityEvent
| where TimeGenerated > ago(30m)
| summarize count() by EventID
| order by count_ desc

_If nothing shows up, that is ok._

#### Install other software commonly leveraged by attackers
Find some software that is goinf to serve as victim software on this machine. The following are commonly used ones by attackers:

##### Install Chocolatey package manager first
- Set-ExecutionPolicy Bypass -Scope Process -Force
- [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
- iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

##### Install common software attackers target
- choco install googlechrome -y
- choco install 7zip -y
- choco install notepadplusplus -y

<img width="1387" height="538" alt="Screenshot 2026-05-14 215659" src="https://github.com/user-attachments/assets/a63494dd-2064-4562-907a-4b65f3398f4a" />

### 2. Ubuntu
_Create Ubuntu Attack server_
_We will be deploying some virtual machines and onboard them to Microsoft Defender._
- Go to Azure Portal → Virtual Machines → Create → Azure Virtual Machine:
- select the following for the cheapest option (The size shown might not be available, I chose a v1 option instead)

<img width="761" height="461" alt="Screenshot 2026-05-19 155620" src="https://github.com/user-attachments/assets/e352ff8a-090c-4c79-aa0f-bc38c34a9599" />

- Under Management tab — enable Auto-shutdown just like the Windows VM.
- Under Networking tab — make sure it's on the same Virtual Network as your Windows VM. This is critical so the two VMs can reach each other for attack simulation.

<img width="1101" height="837" alt="Screenshot 2026-05-19 160211" src="https://github.com/user-attachments/assets/c52f48ee-a0a0-429f-b181-2b8c9e12ef53" />

_In order to SSH into our Ubuntu Server, you need to restrict access to your private key file so that only your user account can read it. Windows permissions are often inherited from parent folders, which makes them too open by default for SSH._
Here is how to secure your key file:
- Move your downloaded file to whatever path you want it to be in on your local computer
- Right-click your private key file (PrivateKeySSH Ubuntu) and select Properties.
- Go to the Security tab and click the Advanced button.
- Click Disable inheritance (and choose to convert inherited permissions into explicit permissions or remove them, depending on your Windows version).
- Click Add -> Select a principal, type your Windows username, click Check Names, and hit OK.
- Under Basic Permissions, check Read and Full Control (or Modify), and click OK.Remove all other users, groups (e.g., "Users"), or inherited accounts from the list so that only your account and SYSTEM remain.

- Next fill out information under SSH comman under our Native SSH section for our Ubuntu Virtual Machine:
- Then, copy the bash query provided and run from our local Windows Powershell

<img width="856" height="837" alt="Screenshot 2026-05-19 163754" src="https://github.com/user-attachments/assets/5412719f-7d9d-4908-96c9-17aafcfc9b0d" />

### Alternatively....
- Under 'Connect', Choose the 'Reset password' mode instead of SSH (if you are having trouble with SSH, you might just want to choose this)

<img width="875" height="862" alt="Screenshot 2026-05-19 164505" src="https://github.com/user-attachments/assets/f1b0659b-f799-494d-91ac-17c0c1aeb310" />

- Then run SSH -i 'path' again, enter password

<img width="774" height="768" alt="Screenshot 2026-05-19 173300" src="https://github.com/user-attachments/assets/9dd2028b-eabd-45b7-9432-f520155ae673" />

#### Update the System and Install Attack Tools
Once connected via SSH, run:

##### Update everything first
sudo apt update && sudo apt upgrade -y

##### Install essential tools
sudo apt install -y nmap netcat-traditional curl wget git python3 python3-pip

##### Install Metasploit Framework
curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall
chmod 755 msfinstall
sudo ./msfinstall

##### Initialize Metasploit database
msfdb init

##### Install CrackMapExec (lateral movement and AD enumeration)
sudo apt install -y crackmapexec

##### Install Impacket (Python tools for Windows/AD attacks)
pip3 install impacket

##### Install Hydra (brute force)
sudo apt install -y hydra

##### Install enum4linux (SMB/Windows enumeration)
sudo apt install -y enum4linux

_This gives us a realistic attacker toolkit covering reconnaissance, exploitation, credential attacks, and lateral movement — all mapped to MITRE ATT&CK techniques._

#### Test Ubunto attack server connection to Windows 11 machine
Run the following to test the connection to Windows 11 machine (We will try to gain access and attack later):

# Basic ping test
ping -c 4 <WINDOWS_PRIVATE_IP>

# Port scan to confirm Windows services are visible
nmap -sV <WINDOWS_PRIVATE_IP>

# Phase 3 SIEM Configuration

## Connect Linux Syslog from Ubuntu VM
- In the Defender portal, go to Microsoft Sentinel > Configurations > content hub
- Search for and install Sysmon and everything under it.
- Now go to data connectors under Configuration.
- Search for Syslog via AMA. Click Open connector page.
- Click Create data collection rule.
- Name it dcr-linux-syslog. Associate it with your Ubuntu VM. Set the destination to law-soc-lab-01.

<img width="591" height="632" alt="Screenshot 2026-05-20 214114" src="https://github.com/user-attachments/assets/b0e32eb8-0835-4450-94e7-ef07b1f0a39d" />

- Select which syslog facilities to collect (auth, syslog, daemon are a good start).
- Click Create.

In order to see activity coming back onto our Ubuntu Attack server, we will add Sysmon to it and attach a DCR wizard to it from Wiondows Defender.

## Cleanup with Defender XDR
_It seems tables in my Advanced hunting Module in Defender were empty. No queries at all were running. LKooks like Defender XDR was not running in Azure > Sentinel. (hoonestly, this migration from Sentinel to Defender has made this entire lab and process quite cumbersome)_
I had to turn on Defender XDR in the Sentinel product hub in Azure and turn on everything in there. I then added more rules.

<img width="1693" height="878" alt="Screenshot 2026-05-26 215609" src="https://github.com/user-attachments/assets/66739ea3-aacd-4c16-8ede-00a21a6779cc" />

Step 3.3 — Enable Analytic Rules (Detections)
Analytic Rules are the detection engine — they run KQL queries on a schedule and create incidents when matches are found.

Go to Microsoft Sentinel > Configurations > Analytics.
Click Rule templates to see all rules installed from Content Hub.
Enable the following rules to start (click each, then Create rule):
Failed logon attempts (potential brute force) — T1110
New local admin user created — T1136
Suspicious PowerShell command line — T1059.001
Rare subscription-level operations in Azure

<img width="1880" height="900" alt="Screenshot 2026-05-26 220957" src="https://github.com/user-attachments/assets/eff510e2-28e0-45f6-a043-7574ae732aa2" />

<img width="1836" height="846" alt="Screenshot 2026-05-26 221627" src="https://github.com/user-attachments/assets/3a0a7d67-993d-43ac-be40-166077427648" />


Step 3.4 — Write Your First Custom Detection Rule
In addition to the template rules, write a custom rule to detect Nmap network scanning:

Go to Microsoft Sentinel > Configurations > Analytics > Create > Scheduled query rule.
Name: Network Port Scan Detected. Severity: Medium. MITRE Tactic: Discovery (T1046).
Paste this KQL as the rule query:
DeviceNetworkEvents
| where Timestamp > ago(5m)
| summarize PortsScanned = dcount(RemotePort), Connections = count() by DeviceName, RemoteIP, bin(Timestamp, 5m)
| where PortsScanned > 15
| project Timestamp, DeviceName, RemoteIP, PortsScanned, Connections
Set the query schedule to run every 5 minutes, look back 5 minutes.
Set alert threshold to greater than 0 results.
Click Save.

<img width="1840" height="864" alt="Screenshot 2026-05-26 221853" src="https://github.com/user-attachments/assets/4f12678f-a1fb-438d-aabc-87e3d6ab2c73" />

# Step 4.1 — Attack Exercise: Brute Force (T1110)
Run the attack from Ubuntu VM:
sudo apt install -y wordlists && sudo gunzip /usr/share/wordlists/rockyou.txt.gz
hydra -l Administrator -P /usr/share/wordlists/rockyou.txt rdp://<WINDOWS_PRIVATE_IP> -t 4 -w 3

Detect it in Sentinel — run in Advanced Hunting:
SecurityEvent
| where EventID == 4625
| where TimeGenerated > ago(1h)
| summarize FailedAttempts = count() by TargetAccount, IpAddress, bin(TimeGenerated, 5m)
| where FailedAttempts > 10
| order by FailedAttempts desc

Expected outcome:
Multiple Event ID 4625 entries and a Defender alert for brute force activity.

Step 4.2 — Attack Exercise: Network Scan (T1046)
Run the attack from Ubuntu VM:
nmap -sV -p 1-1000 <WINDOWS_PRIVATE_IP>

Detect it in Sentinel:
DeviceNetworkEvents
| where Timestamp > ago(30m)
| where RemoteIP == "<UBUNTU_PRIVATE_IP>"
| summarize PortsScanned = dcount(RemotePort), Count = count() by DeviceName, RemoteIP, bin(Timestamp, 5m)
| where PortsScanned > 10

Step 4.3 — Attack Exercise: Create Local Admin Account (T1136)
Run from inside Windows VM (via RDP or PowerShell):
net user backdoor P@ssw0rd123! /add
net localgroup administrators backdoor /add

Detect it in Sentinel:
SecurityEvent
| where EventID == 4720
| project TimeGenerated, TargetUserName, SubjectUserName, Computer

