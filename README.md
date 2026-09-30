# n8n Malicious IP Alert

Automated threat detection workflow built with **n8n**. It collects security alerts, checks every public IP on **VirusTotal**, picks the most malicious one, and sends an AI-written HTML alert email to the SOC team.

> Built as part of my 4-month training at NTI (National Telecommunication Institute).

---

## Overview

Manually checking every IP in a list of alerts is slow and error-prone. This workflow automates the whole process, from raw alerts to a ready-to-act email, with no manual steps.

## How It Works

1. **Collect:** Authenticates with the alerts platform, fetches the alerts (24 items), and extracts source and destination IPs.
2. **Filter:** Removes duplicates and private ranges (10.x, 172.16-31.x, 192.168.x, 127.x), leaving 12 public IPs.
3. **Enrich:** Checks each IP on the VirusTotal API v3.
4. **Prioritize:** Sorts by malicious detections and keeps the top IP.
5. **Decide:** If the top IP has more than 5 malicious detections, the workflow continues. Otherwise it stops.
6. **Alert:** Google Gemini writes a professional HTML email and it is sent via SMTP.

## Tech Stack

| Tool | Purpose |
|---|---|
| n8n | Workflow automation |
| VirusTotal API v3 | IP reputation lookup |
| Google Gemini | AI-generated email body |
| SMTP (Gmail) | Email delivery |

## Workflow Nodes

| Node | Type | Purpose |
|---|---|---|
| Get Token / Get Alerts | HTTP Request | Authenticate and fetch alerts |
| Split Out All alerts | Split Out | One item per alert |
| Get All Info about alert | HTTP Request | Full alert details |
| Split Out dest & src | Split Out | Separate source and destination IPs |
| Aggregate src & dest without null | Aggregate | Collect all IPs, ignore empty values |
| All Public IPs | Code | Deduplicate and drop private IPs |
| VirusTotal HTTP Request | HTTP Request | Look up each IP |
| Format VT Results | Edit Fields | Keep the useful fields |
| Sort Malicious IPs | Sort | Order by `malicious`, descending |
| Top Malicious IP | Limit | Keep the first item only |
| Malicious > 5 | IF | Threshold check |
| Write Alert Email | Basic LLM Chain + Gemini | Generate the HTML email |
| Send Alert Email | Send Email (SMTP) | Deliver the alert |

## Results

Out of 12 public IPs, four had detections:

| IP | Malicious | Owner |
|---|---|---|
| 185.220.101.42 | **17** | Stiftung Erneuerbare Freiheit |
| 94.102.49.190 | 8 | IP Volume inc |
| 103.224.182.251 | 6 | Trellian Pty. Limited |
| 45.142.214.89 | 2 | Clouvider Limited |

The most dangerous IP (`185.220.101.42`) triggered the alert email:


## Setup

1. Import `workflow.json` into n8n (**Workflows → Import from file**).
2. Create these credentials:
   - VirusTotal API key
   - Google Gemini API key
   - SMTP (for Gmail, use an **App Password**, not your regular password)
3. Replace the placeholders: server IP, token values, and email addresses.
4. Unpin all nodes and click **Execute workflow**.

## Notes

- The free VirusTotal tier allows 4 requests per minute. Use batching (1 item per batch, 15 s interval) for larger lists.
- The threshold (`5`) can be changed in the `Malicious > 5` node.
- Never commit API keys, tokens, or passwords.


- Replace the manual trigger with a Schedule Trigger.
- Send one email that includes all dangerous IPs.
- Add Telegram or Slack notifications.


**Your Name**
[LinkedIn](https://linkedin.com/in/your-profile)
