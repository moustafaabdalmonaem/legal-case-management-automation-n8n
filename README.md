# Legal Case Management Automation (n8n)

An n8n workflow that automates legal case tracking for a bank's legal affairs department. It eliminates manual data entry and ensures no court hearing is ever missed.

## What It Does

This workflow combines two automated processes that share a single Google Sheet as the source of truth:

### 1. Daily Hearing Reminders
Every day at 8:00 AM, the workflow scans the case sheet for any active case with a hearing scheduled **the next day** and sends an email reminder with the full case details. Once sent, the case is marked so the reminder isn't repeated.

### 2. Photo-to-Sheet Case Intake (AI-Powered)
Instead of manually typing case data, the employee simply photographs a physical document (court notice, case file, etc.) and sends it to a Telegram bot. The workflow then:
1. Downloads the photo from Telegram
2. Sends it to an AI vision model (GPT-4o) to extract structured case data
3. Appends the extracted data as a new row in the Google Sheet
4. Reads the full caseload and calculates a quick summary (total active cases, hearings due in the next 7 days)
5. Emails the summary along with the new case details to the manager
6. Replies to the employee on Telegram confirming the data was saved

## Architecture

```
┌─────────────────┐         ┌──────────────────────┐
│ Schedule Trigger │         │  Telegram Trigger     │
│   (Daily 8 AM)   │         │  (Photo message)      │
└────────┬─────────┘         └──────────┬────────────┘
         │                              │
         ▼                              ▼
   Read Sheet                    Download Photo
         │                              │
         ▼                              ▼
  Filter Due Cases              Convert to Base64
         │                              │
         ▼                              ▼
  Send Reminder Email          AI Extract Data (GPT-4o)
         │                              │
         ▼                              ▼
  Update Alert Status           Parse AI Response
                                        │
                                        ▼
                                 Append to Google Sheet
                                        │
                                        ▼
                                 Analyze Caseload
                                        │
                              ┌─────────┴─────────┐
                              ▼                    ▼
                    Email Manager          Confirm to Employee
```

## Google Sheet Structure

Create a sheet named `Cases` with the following columns (in order):

| Column | Description |
|---|---|
| `case_number` | Unique case identifier |
| `client_name` | Client or opposing party name |
| `case_type` | e.g. enforcement, dispute, collection |
| `court` | Court handling the case |
| `next_hearing_date` | Format: `YYYY-MM-DD` |
| `lawyer` | Responsible lawyer |
| `status` | `Active` or `Closed` |
| `notes` | Free text |
| `alert_sent` | `Yes` / `No` — used internally to avoid duplicate reminders |

## Requirements

| Service | Purpose | Where to get it |
|---|---|---|
| n8n | Workflow engine | [n8n.io](https://n8n.io) (self-hosted or cloud) |
| Telegram Bot | Photo intake | [@BotFather](https://t.me/BotFather) on Telegram |
| OpenAI API | AI data extraction (GPT-4o vision) | [platform.openai.com](https://platform.openai.com) |
| Google Sheets | Data storage | Google Cloud OAuth2 credentials |
| Gmail | Email notifications | Google Cloud OAuth2 credentials |

## Setup

1. Create a Google Sheet with the structure above and copy its Sheet ID from the URL
2. Create a Telegram bot via BotFather and note the bot token
3. Get an OpenAI API key
4. In n8n: **Workflows → Import from File** and select `legal_case_full_automation_n8n.json`
5. Open each node containing `REPLACE_WITH...` and set:
   - Your Google Sheet ID
   - Your email address (for hearing reminders)
   - Manager's email address (for case intake summaries)
   - Credentials: Telegram Bot, OpenAI, Google Sheets OAuth2, Gmail OAuth2
6. Test manually (send a photo to the bot, or run the schedule branch manually) before activating
7. Toggle the workflow to **Active**

## Notes & Limitations

- AI extraction accuracy depends on photo quality — clear lighting and a flat, well-framed shot work best
- Currently the extracted data saves automatically without a manual review step; a confirmation step can be added if higher accuracy control is needed
- Designed for Arabic-language documents; extraction prompt can be adjusted for other languages or document formats

## Files

- `legal_case_full_automation_n8n.json` — the complete n8n workflow (import this)

<img width="1280" height="638" alt="Workflow 9" src="https://github.com/user-attachments/assets/1f6480cd-485c-495f-ab3c-19e189c4fee5" />
