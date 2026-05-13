# Microsoft-Azure-Sentinel-Lab-SIEM-1
Microsoft Azure &amp; Sentinel Lab (SIEM) 1


# Steps

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
