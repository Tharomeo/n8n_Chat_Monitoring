<table>
  <tr>
    <td><img src="https://img.icons8.com/fluency/64/chat.png" width="60"/></td>
    <td><h1>Chat Inactivity Monitor</h1></td>
  </tr>
</table>

> Automated inactivity detection for support chats — triggers escalating email alerts when clients, technicians, or assignments are left unattended.

![n8n](https://img.shields.io/badge/n8n-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-3ECF8E?style=for-the-badge&logo=supabase&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![Status](https://img.shields.io/badge/Status-Active-28a745?style=for-the-badge)

---

## 📌 About

This workflow continuously monitors open support chats and detects inactivity based on who is expected to respond. It tracks three distinct scenarios and sends escalating email alerts at 25 and 35 minutes of inactivity — escalating from management to directors to ensure no ticket is left unattended.

Alert history is persisted in Supabase to avoid duplicate notifications.

---

## ✨ Features

- 🔁 **Scheduled polling** — runs automatically at regular intervals
- 👤 **Client inactivity detection** — alerts when a client hasn't replied
- 🧑‍💻 **Technician inactivity detection** — alerts when a technician hasn't responded
- 🚨 **Unassigned chat detection** — alerts when no technician has been assigned
- ⏱️ **Escalating alerts** — first alert to management at 25 min, escalated to directors at 35 min
- 🔔 **Duplicate prevention** — Supabase tracks which alerts have already been sent
- 🧹 **Automatic cleanup** — closed chats are removed from the tracking database

---

## 🔄 How it works

```
Schedule Trigger
      │
      ▼
Fetch all open chats (HTTP Request)
      │
      ▼
Loop over each chat
      │
      ▼
Identify chat state
      │
      ├── Waiting for CLIENT response
      │         │
      │   Check alert history (Supabase)
      │         ├── 25 min → Alert to Management
      │         └── 35 min → Escalate to Board / Directors
      │
      ├── Waiting for TECHNICIAN response
      │         │
      │   Check alert history (Supabase)
      │         ├── 25 min → Alert to Management
      │         └── 35 min → Escalate to Board / Directors
      │
      └── NO TECHNICIAN assigned
                │
          Check alert history (Supabase)
                ├── 25 min → Alert to Management
                └── 35 min → Escalate to Board / Directors
```

A second scheduled trigger runs periodically to **clean up resolved chats** from the Supabase tracking table.

---

## 📧 Alert Levels

| Time | Escalation | Recipients |
|---|---|---|
| **25 min** | ⚠️ First warning | Management |
| **35 min** | 🔴 Final escalation | Board / Directors |

Each scenario sends a specific email with context about the chat, making it easy to act immediately.

---

## 🗄️ Database

Alert state is tracked in **Supabase** to prevent duplicate notifications. One table per scenario, each storing the chat ID and the highest alert level already sent.

| Column | Description |
|---|---|
| `chat_id` | Identifier of the monitored chat |
| `alert_level` | Highest alert already sent (1 = management, 2 = directors) |
| `created_at` | Timestamp of the first alert |

Records are automatically deleted when the chat is resolved or closed.

---

## 🛠️ Stack

| Tool | Role |
|---|---|
| **n8n** | Workflow automation engine |
| **Support platform API** | Fetch open chats via HTTP |
| **Supabase** | Alert history and deduplication |
| **SMTP / Resend** | Email alert delivery |

---

## 🚀 Setup

**1. Import the workflow into your n8n instance**

- `Chat_Inactivity_Monitor.json`

**2. Configure credentials in n8n**

- `HTTP Request` — add your support platform API key
- `Supabase` — connect your Supabase project
- `SMTP` — configure your email sender

**3. Create the tracking tables in Supabase**

```sql
-- One table per scenario (client / technician / unassigned)
create table chat_alerts (
  id uuid primary key default gen_random_uuid(),
  chat_id text,
  alert_level int,
  created_at timestamptz default now()
);
```

**4. Activate the workflow**

<div align="center">

![built with n8n](https://img.shields.io/badge/Built_with-n8n-EA4B71?style=flat-square&logo=n8n&logoColor=white)

</div>
