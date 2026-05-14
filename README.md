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

## Microsoft Sentinel
- Next, search for Microsoft Sentinal, and click on create.
- Select our workspace we just created.
- click Add

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

_We have our Virtual Network setup. Now, we should be able to add virtual machines._

# Phase 2 - Endpoints and EDR
_We will be deploying some virtual machines and onboard them to Microsoft Defender._
_After that, we will be setting up data collection rules. For our Windows VM's, data we should collect is:_
- Security event log — all events (or at minimum, audit success + failure)
- System event log — warnings and errors
- Application event log — errors only
- Microsoft-Windows-Sysmon/Operational — all events (this is the gold mine)

- 
