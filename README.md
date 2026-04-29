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
_We will next connect this with Entra ID, which helps enables audit logs and sign-in._
- Go to content hub for Sentinal
- Search for Entra ID, click, and select install.

<img width="1196" height="868" alt="Screenshot 2026-04-28 220545" src="https://github.com/user-attachments/assets/c7be2b65-7d08-4cfa-869d-8351b5ac9f40" />

- Go to our Entra ID portal
- Go to Diagnostic settings under Monitoring and health
- Make new setting
- Check Auditlogs and Signin logs.
- include subscription and workspace

<img width="1395" height="898" alt="Screenshot 2026-04-28 230507" src="https://github.com/user-attachments/assets/3e150465-d06f-4809-ae2f-0ba60bfc83f6" />

- Go back to Azure portal tab
- select Manage on bottom right
_Will want to take of sign-in logs soon after because charges will accumilate fast._
- Click on Microsoft Entra
- Open Connector for Microsoft Entra ID

<img width="1196" height="862" alt="Screenshot 2026-04-28 230836" src="https://github.com/user-attachments/assets/7352253a-34cb-4b87-85c2-13be67034faf" />

- Check Sign-in logs and Audit Logs.
- Apply Changes

_Confirm our logs are running_
- Sign out of Entra ID and back in a few different times so we can get som activity logs for sign-ins.

- Go back to Sentinal
- click on our lab we created
- Click on logs

<img width="1166" height="863" alt="Screenshot 2026-04-28 231710" src="https://github.com/user-attachments/assets/decf3e01-1bc0-457e-a934-37d806997c92" />

Use the following KQL query:
SigninLogs
| where ResultType != 0
| take 10 

- That may or may not get us anything if we didn't fail to login.
- Go to Sentinal > Analytics, and click on Go to Defneder Portal.
- Select create under Analytics. Then schedule query rule
- For Mitre attack, we are selecting credential access and Brute Force.

<img width="1123" height="816" alt="Screenshot 2026-04-28 234313" src="https://github.com/user-attachments/assets/fa67c72c-9375-4777-a33c-e7cce80af48a" />

- Next Rule Logic
- Enter are query

<img width="1107" height="818" alt="Screenshot 2026-04-28 234721" src="https://github.com/user-attachments/assets/679a0345-9f3a-4453-aa48-ad0b818a2f07" />

- Group out alerts in Incident Settings
- Review + Create
- hit save

_We now have created a rule, which will notify us of alerts if there is any suspicous activity of Bruteforce login attacks._

<img width="1617" height="760" alt="Screenshot 2026-04-28 235127" src="https://github.com/user-attachments/assets/d23b7d27-2fd3-4c78-a9a8-9d8f5914a3ae" />

Next, we will review and investigate failed login attacks.
