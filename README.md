# Microsoft-Azure-Sentinel-Lab-SIEM-1
Microsoft Azure &amp; Sentinel Lab (SIEM) 1


# Steps

## Make a Microsoft Account
- Choose to create one under Microsoft Azure Website either as personal, or under an organization.
- Make sure to choose the $200 credit option. Once that is finished with the $200, it will prompt to go to a pay account.

## Welcome to Azure - Azure Home Screen

<img width="1329" height="916" alt="1_Welcometoazure" src="https://github.com/user-attachments/assets/6c0c1cb9-3ef1-493b-841e-9edd215a5335" />

- Go to Log Analytic Work spaces, and create a workspace.

<img width="1173" height="431" alt="Screenshot 2026-04-28 215334" src="https://github.com/user-attachments/assets/e72d76d9-bae4-4ada-9bd5-cf823cef29a0" />

- Enter in names and descriptions, and click on review + create.

<img width="784" height="886" alt="Screenshot 2026-04-28 215505" src="https://github.com/user-attachments/assets/c41e1d3a-7485-4841-90d0-ce195f8d7326" />

- After a minute of loading, click on create to create it.

_Your deploment is complete._

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

