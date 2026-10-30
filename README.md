<div align="center">

# Multi-Tenant Media Response Agent

**AI-powered Instagram & Facebook DM automation — multi-tenant social media response with persistent memory, lead qualification, and automated meeting scheduling.**

<br />

<p align="center">
  <img
    src="https://readme-typing-svg.herokuapp.com?font=Fira+Code&size=24&duration=3500&pause=800&color=00FF9F&center=true&vCenter=true&width=700&lines=Multi-Tenant+Media+Response+Agent;n8n+%7C+Claude+Sonnet+4.6+%7C+Postgres+Memory;Instagram+%2B+Facebook+DMs+%7C+Zoom+%2B+Google+Calendar+Meetings"
    alt="Typing SVG"
  />
</p>

<p align="center">
  <a href="https://n8n.io/">
    <img src="https://img.shields.io/badge/n8n-Workflow_Automation-000000?style=for-the-badge&logo=n8n&logoColor=white" alt="n8n" />
  </a>
  <a href="https://www.anthropic.com/">
    <img src="https://img.shields.io/badge/Claude-Sonnet%204.6-8A2BE2?style=for-the-badge&logo=anthropic&logoColor=white" alt="Claude Sonnet 4.6" />
  </a>
  <a href="https://www.postgresql.org/">
    <img src="https://img.shields.io/badge/PostgreSQL-Chat_Memory-316192?style=for-the-badge&logo=postgresql&logoColor=white" alt="PostgreSQL" />
  </a>
  <a href="https://supabase.com/">
    <img src="https://img.shields.io/badge/Supabase-Session_Pooler-181818?style=for-the-badge&logo=supabase&logoColor=white" alt="Supabase" />
  </a>
  <a href="https://www.instagram.com/">
    <img src="https://img.shields.io/badge/Instagram-DM-E4405F?style=for-the-badge&logo=instagram&logoColor=white" alt="Instagram" />
  </a>
  <a href="https://www.facebook.com/">
    <img src="https://img.shields.io/badge/Facebook-Messages-1877F2?style=for-the-badge&logo=facebook&logoColor=white" alt="Facebook" />
  </a>
  <a href="https://zoom.us/">
    <img src="https://img.shields.io/badge/Zoom-Meetings-2D8CFF?style=for-the-badge&logo=zoom&logoColor=white" alt="Zoom" />
  </a>
  <a href="https://calendar.google.com/">
    <img src="https://img.shields.io/badge/Google_Calendar-Scheduling-4285F4?style=for-the-badge&logo=googlecalendar&logoColor=white" alt="Google Calendar" />
  </a>
  <a href="https://sheets.google.com/">
    <img src="https://img.shields.io/badge/Google_Sheets-Data-34A853?style=for-the-badge&logo=googlesheets&logoColor=white" alt="Google Sheets" />
  </a>
  <a href="https://mail.google.com/">
    <img src="https://img.shields.io/badge/Gmail-Notifications-EA4335?style=for-the-badge&logo=gmail&logoColor=white" alt="Gmail" />
  </a>
  <br />

</p>

</div>

---

## Why this exists

Real-estate investing brands running social media campaigns face fragmented inboxes across Instagram DMs and Facebook messages/comments. Leads fall through the cracks, agents repeat themselves, and meeting booking is manual and error-prone — including duplicate Zoom links when context is lost between restarts.

This n8n workflow solves it with a **multi-tenant AI response agent**: every incoming event is resolved to a `company_id`, company-specific context is injected into Claude Sonnet 4.6's system prompt from Google Sheets, and the agent handles replies, lead capture, escalation, and end-to-end Zoom + Google Calendar meeting booking with persistent conversation memory on Supabase.

---

## Feature grid

| Feature | Description |
|---|---|
| Multi-tenant isolation | Each event resolves to a `company_id`; only that company's context is injected — never another tenant's data |
| Context-aware replies | Company config (brand identity, services, offers, brand voice, FAQ, escalation rules, restricted topics, language mode) injected into the AI system prompt |
| Lead qualification | AI classifies leads as `potential`, `qualified`, or `not_qualified` with reasoning, captured to Google Sheets |
| Human escalation | Conversations flagged for escalation are logged to a dedicated escalations sheet with reason |
| Automated meeting scheduling | Zoom meeting created → Google Calendar event with Zoom link → internal Gmail notification → customer confirmation |
| Persistent memory | Postgres Chat Memory on Supabase with session key `company_id_platform_senderId`, context window 20 |
| Social-safe formatting | Markdown stripped from replies; URLs protected; plain-text output optimized for Instagram/Facebook DMs |

---

## Architecture

The node graph below is derived directly from the exported workflow. Webhooks ingest Instagram and Facebook events, a processor normalizes them, tenant context is looked up from Google Sheets, and the LangChain AI agent orchestrates replies and tool calls with persistent memory.

```mermaid
graph TD
    subgraph Ingestion["Channel Ingestion"]
        IW[Instagram Webhook] --> R1[Respond to Webhook2/4]
        FW[Facebook Webhook] --> R2[Respond to Webhook1/3]
        R1 --> MP[Message Processor]
        R2 --> MP
    end

    subgraph Tenant["Multi-Tenant Resolution"]
        MP --> PDL[Page Directory Lookup<br/>Google Sheets]
        PDL --> CML[Company Memory Lookup<br/>Google Sheets]
        CML --> BC[Build Context<br/>Set node]
    end

    subgraph AI["AI Core"]
        BC --> CR[Channel Router<br/>Switch]
        CR -->|Instagram DM| CM1[Clean the Message1]
        CR -->|Facebook DM| CM2[Clean the Message]
        CM1 --> IDS[Instagram DM Sender<br/>HTTP Request]
        CM2 --> FDS[Facebook DM Sender<br/>HTTP Request]
        CR --> AA[AI Agent<br/>LangChain]
    end

    subgraph Model["Model & Memory"]
        ACM[Anthropic Chat Model<br/>Claude Sonnet 4.6] -.-> AA
        PCM[Postgres Chat Memory<br/>Supabase session pooler] -.-> AA
    end

    subgraph Tools["Agent Tools"]
        AA <-> LCT[Lead Capture Tool<br/>Google Sheets]
        AA <-> ET[Escalation Tool<br/>Google Sheets]
        AA <-> WFT[Website Fetch Tool<br/>HTTP Request]
        AA <-> AST[Appointment Scheduler<br/>Google Calendar]
        AA <-> ZMT[Zoom Meeting Tool<br/>HTTP Request]
        AA <-> GMT[Gmail Meeting Notify<br/>Gmail]
    end

    subgraph Profile["Sender Profile"]
        BC --> GSP[Get Sender Profile<br/>HTTP Request]
        GSP --> RSN[Resolve Sender Name<br/>Code node]
        RSN --> AA
    end

    style IW fill:#e1306c,stroke:#333,stroke-width:2px
    style FW fill:#1877f2,stroke:#333,stroke-width:2px
    style AA fill:#8a2be2,stroke:#333,stroke-width:2px,color:#fff
    style PCM fill:#181818,stroke:#3ECF8E,stroke-width:2px,color:#fff
```

### Meeting-booking sequence

```mermaid
sequenceDiagram
    autonumber
    participant C as Customer
    participant WA as Webhook Agent
    participant AA as AI Agent (Claude 4.6)
    participant ZM as Zoom Meeting Tool
    participant GC as Google Calendar Tool
    participant GM as Gmail Meeting Notify Tool

    C->>WA: Confirms meeting date & time via DM
    WA->>AA: Intent = book meeting + start/duration/notes
    AA->>ZM: Create Zoom meeting (join_url returned)
    ZM-->>AA: Meeting details incl. join_url
    AA->>GC: Create Calendar event with Zoom link in description
    GC-->>AA: Calendar event created
    AA->>GM: Send internal notification to sales team
    GM-->>AA: Notification sent
    AA->>C: Reply with confirmed time + Zoom link (plain text)
```

---

## Project structure

```
Multi-Tenant Media Response Agent/
├── Agent 6 _ Media Response Agent.json   # n8n workflow export (27 nodes, AI agent + tools)
├── workflow.sanitized.json                # Workflow copy with credentials, IDs, URLs, and emails stripped
├── README.md                              # This file
├── .gitignore                             # Ignore credential files, .env, node_modules
└── docs/                                  # Optional: additional setup docs (create as needed)
```

---

## Engineering highlights

### Persistent memory migration

**Problem:** The original in-process Simple Memory lost conversation history on every workflow restart or save. Returning users were treated as new, causing forced re-greetings and duplicate Zoom meeting creation.

**Root cause:** Simple Memory is ephemeral and tied to the n8n execution context; it does not survive across runs.

**Fix:** Replaced with **Postgres Chat Memory** backed by Supabase (session pooler). Session key: `company_id_platform_senderId`. Context window: 20. History now persists across restarts, so returning customers are recognized and existing meetings are not duplicated.

### Duplicate-meeting prevention

Before creating a new Zoom meeting, the agent checks conversation history via Postgres memory. If a meeting already exists for the current session/context, it confirms the existing time or offers rescheduling — no duplicate Zoom links are generated.

### Prompt compression with identical behavior

The system prompt compresses all company context into a single structured block injected at runtime from Google Sheets. Behavior remains identical across tenants because each company receives only its own context — no cross-tenant leakage.

### Company isolation

Strict `company_id` scoping is enforced at lookup time. The agent uses only information for the resolved `company_id`; it never guesses or invents company information and escalates when data is missing or restricted.

### Safe Zoom-failure fallback

If Zoom meeting creation fails, the agent does not invent a join URL. It escalates the conversation to a human agent via the Escalation Tool instead of fabricating a link.

---

<details>
<summary>Node &amp; tool reference (generated from workflow JSON)</summary>

| Node | Type | Purpose |
|---|---|---|
| Instagram Webhook | n8n-nodes-base.webhook | Receives Instagram webhook events |
| Facebook Webhook | n8n-nodes-base.webhook | Receives Facebook webhook events |
| Respond to Webhook1 | n8n-nodes-base.respondToWebhook | Facebook webhook challenge response |
| Respond to Webhook2 | n8n-nodes-base.respondToWebhook | Instagram webhook challenge response |
| Respond to Webhook3 | n8n-nodes-base.respondToWebhook | Facebook webhook JSON acknowledgment |
| Respond to Webhook4 | n8n-nodes-base.respondToWebhook | Instagram webhook JSON acknowledgment |
| Message Processor | n8n-nodes-base.code | Parses webhook body into structured events (platform, sender, message, etc.) |
| Page Directory Lookup | n8n-nodes-base.googleSheets | Looks up page by platform + page_id to resolve company |
| Company Memory Lookup | n8n-nodes-base.googleSheets | Looks up company config by company_slug |
| Build Context | n8n-nodes-base.set | Builds context assignments fed into the AI agent prompt |
| Channel Router | n8n-nodes-base.switch | Routes to Instagram DM or Facebook DM based on platform + event_type |
| Clean the Message / Clean the Message1 | n8n-nodes-base.code | Strips Markdown, protects URLs, outputs plain text |
| Instagram DM Sender | n8n-nodes-base.httpRequest | Sends reply via Instagram Graph API |
| Facebook DM Sender | n8n-nodes-base.httpRequest | Sends reply via Facebook Graph API |
| AI Agent | @n8n/n8n-nodes-langchain.agent | LangChain agent orchestrating replies and tool calls |
| Anthropic Chat Model | @n8n/n8n-nodes-langchain.lmChatAnthropic | Claude Sonnet 4.6 language model |
| Postgres Chat Memory | @n8n/n8n-nodes-langchain.memoryPostgresChat | Persistent conversation memory on Supabase |
| Get Sender Profile | n8n-nodes-base.httpRequest | Fetches sender profile from Meta Graph API |
| Resolve Sender Name | n8n-nodes-base.code | Resolves real sender name from profile response |
| Lead Capture Tool | n8n-nodes-base.googleSheetsTool | Appends qualified lead data to Lead_Capture sheet |
| Escalation Tool | n8n-nodes-base.googleSheetsTool | Appends escalation records to Escalation sheet |
| Website Fetch Tool | n8n-nodes-base.httpRequestTool | Fetches company WordPress page content via REST API |
| Appointment Scheduler Tool | n8n-nodes-base.googleCalendarTool | Creates Google Calendar event with meeting details |
| Zoom Meeting Tool | n8n-nodes-base.httpRequestTool | Creates Zoom meeting via Zoom API |
| Gmail Meeting Notify Tool | n8n-nodes-base.gmailTool | Sends internal meeting notification email |
| Sticky Note10 | n8n-nodes-base.stickyNote | Workflow annotation |

</details>

---

## Setup guide

### Prerequisites

- n8n instance (self-hosted or n8n cloud)
- Google Cloud project with Google Sheets, Google Calendar, and Gmail APIs enabled
- Meta app with Instagram and Facebook webhook permissions
- Zoom OAuth app
- Anthropic API account with Claude access
- Supabase project with PostgreSQL (session pooler)

### Import the workflow

1. Export this workflow to your n8n instance via **Import from file**.
2. Use `workflow.sanitized.json` as the clean starting template (all credentials, IDs, URLs, and emails already stripped).
3. Open each node that uses credentials and create the credential entries below.

### Credential types to create

| Credential type | Service | Fields to configure |
|---|---|---|
| googleSheetsOAuth2Api | Google Sheets | client id, client secret |
| googleCalendarOAuth2Api | Google Calendar | client id, client secret |
| zoomOAuth2Api | Zoom | account id, client id, client secret |
| gmailOAuth2 | Gmail | client id, client secret |
| anthropicApi | Anthropic | API key |
| postgres | PostgreSQL (Supabase) | host, port, database, user, password |

> **Note:** Credential names and values are intentionally omitted from this repo. Create them in the n8n UI and select them in each node's credentials panel.

### Google Sheets structure

Four sheets are referenced in the workflow. Create them in a single Google Sheet and share with your service account:

**Sheet 1 — Page Directory** (lookup by `platform` + `page_id`)

| Column | Purpose |
|---|---|
| platform | instagram or facebook |
| page_id | Meta page / Instagram account ID |
| company_slug | Unique company identifier |
| page_access_token | Page access token for outbound replies |

**Sheet 2 — Company Information** (lookup by `Company Name` → `company_slug`)

| Column | Purpose |
|---|---|
| company_slug | Unique company identifier |
| business_name | Display name |
| assistant_name | Assistant name |
| website_url | Company website |
| services | Services offered |
| products | Products offered |
| target_audience | Target audience |
| faqs | FAQ knowledge |
| approved_answers | Approved response templates |
| lead_qualification_rules | Qualification rules |
| cta | Default call-to-action |
| contact_info | Contact information |
| escalation_rules | Escalation criteria |
| brand_voice | Brand tone/voice |
| language_mode | Response language |
| not_allowed | Restricted topics |
| leads_sheet_id | Linked leads sheet ID |
| calendar_id | Linked Google Calendar ID |

**Sheet 3 — Lead_Capture** (append)

| Column | Source |
|---|---|
| lead_id | AI-generated unique ID |
| company_id | Resolved company ID |
| company_name | Company name |
| platform | instagram / facebook |
| sender_id | Customer sender ID |
| sender_name | Resolved sender name |
| message | Customer message |
| lead_status | potential / qualified / not_qualified |
| lead_reason | Qualification reasoning |
| property_address | Extracted from conversation |
| property_type | Extracted from conversation |
| asking_price | Extracted price |
| motivation | Seller motivation |
| timeline | Selling timeline |
| contact_information | Phone/email if provided |
| created_at | Timestamp |

**Sheet 4 — Escalation** (append)

| Column | Purpose |
|---|---|
| escalation_id | Unique escalation ID |
| company_id | Resolved company ID |
| company_name | Company name |
| platform | instagram / facebook |
| sender_id | Customer sender ID |
| sender_name | Resolved sender name |
| message | Customer message |
| escalation_reason | AI-generated reason |
| status | pending |
| created_at | Timestamp |

### Supabase / Postgres setup

- Enable the **session pooler** in Supabase (port 6543 typically).
- Require SSL (`sslmode=require`).
- Keep `max_connections` small (pooler manages concurrency).
- Create a table for chat message history (n8n Postgres Chat Memory manages schema automatically).
- Use the `postgres` credential type in n8n with your Supabase connection details.

### Meta webhook configuration

1. Create a Meta app with **Instagram** and **Facebook** products.
2. Configure webhook subscriptions for `messages`, `messaging_postbacks`, and `comments`.
3. Set the callback URL to your n8n webhook endpoints:
   - Instagram webhook path: `/webhook/Instagram`
   - Facebook webhook path: `/webhook/facebook_debug`
4. Complete the challenge handshake (n8n `Respond to Webhook` nodes handle `hub.challenge`).
5. Subscribe your app to the relevant Instagram/Facebook pages.

### Activate & test

1. Open the imported workflow in n8n.
2. Select credentials for every credential-bearing node.
3. Set the correct Google Sheet document ID and sheet names/tabs in each Google Sheets node.
4. **Activate** the workflow.
5. Send a test DM from Instagram and a test message/comment from Facebook.
6. Verify: context lookup → AI reply → DM sender delivery.
7. Test the meeting flow end-to-end (Zoom → Calendar → Gmail → confirmation).
8. Test a returning user to confirm memory persistence and no duplicate meeting creation.

### Test checklist

- [ ] Instagram webhook receives and responds to challenge
- [ ] Facebook webhook receives and responds to challenge
- [ ] Page Directory Lookup resolves company_id correctly
- [ ] Company Memory Lookup injects company context
- [ ] AI Agent replies with correct brand voice
- [ ] Zoom meeting creation returns a valid join_url
- [ ] Google Calendar event includes the Zoom link
- [ ] Gmail notification is sent to the sales team
- [ ] Returning user is recognized (no duplicate greeting)
- [ ] Duplicate meeting request is handled gracefully
- [ ] Escalation logs to the Escalations sheet
- [ ] Lead capture logs to the Lead_Capture sheet

---

## Configuration reference

Configure the following in the n8n UI (credential names only — no values):

- `googleSheetsOAuth2Api` — used by Page Directory Lookup, Company Memory Lookup, Lead Capture Tool, Escalation Tool
- `googleCalendarOAuth2Api` — used by Appointment Scheduler Tool
- `zoomOAuth2Api` — used by Zoom Meeting Tool
- `gmailOAuth2` — used by Gmail Meeting Notify Tool
- `anthropicApi` — used by Anthropic Chat Model (Claude Sonnet 4.6)
- `postgres` — used by Postgres Chat Memory

---

## Example conversation

**Customer (Instagram DM):** Hi, I'm interested in selling my property. Can we schedule a call?

**Agent:** Thanks for reaching out! I'd be happy to help. Could you share the property address and your ideal time for a call?

**Customer:** It's 123 Main St. Tomorrow at 2pm works for me.

**Agent:** Got it — I've booked a Zoom meeting for tomorrow at 2:00 PM UTC. I'll send the join link shortly. In the meantime, could you share your asking price?

**[Behind the scenes: Zoom meeting created → Calendar event created → Gmail notification sent to sales team]**

**Agent:** Your meeting is confirmed for tomorrow at 2:00 PM. Here's your Zoom link: https://zoom.us/j/.... Looking forward to speaking with you!

---

**[Two days later — same customer messages again]**

**Customer:** Hi, just following up on my property sale.

**Agent:** Hi! Great to hear from you again. As discussed, your meeting is scheduled for tomorrow at 2:00 PM. Is there anything else you'd like to cover before then?

**[Behind the scenes: Postgres memory recognized `company_id_instagram_senderId`; no new meeting created; no duplicate Zoom link]**

---

## Roadmap (planned)

- Tool-level duplicate meeting guard backed by a dedicated Meetings sheet in Google Sheets
- Webhook message-ID deduplication to prevent reprocessing the same event
- Reschedule and cancel flow for existing Zoom + Calendar meetings

All items above are **planned** and not yet implemented.

---

## Tech & Skills Showcase

### AI / Automation
<p align="left">
  <img src="https://img.shields.io/badge/n8n-Workflow_Automation-000000?style=for-the-badge&logo=n8n&logoColor=white" />
  <img src="https://img.shields.io/badge/LangChain-AI_Agent-2EA44F?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Claude-Sonnet%204.6-8A2BE2?style=for-the-badge&logo=anthropic&logoColor=white" />
  <img src="https://img.shields.io/badge/Prompt_Engineering-System%20Prompt-ff6b6b?style=for-the-badge" />
</p>

### Backend / Data
<p align="left">
  <img src="https://img.shields.io/badge/PostgreSQL-Chat_Memory-316192?style=for-the-badge&logo=postgresql&logoColor=white" />
  <img src="https://img.shields.io/badge/Supabase-Session_Pooler-181818?style=for-the-badge&logo=supabase&logoColor=white" />
  <img src="https://img.shields.io/badge/JavaScript-Code_Nodes-f7df1e?style=for-the-badge&logo=javascript&logoColor=black" />
  <img src="https://img.shields.io/badge/REST%2FWebhooks-HTTP_Request-ff4500?style=for-the-badge" />
</p>

### Integrations
<p align="left">
  <img src="https://img.shields.io/badge/Instagram-DM-E4405F?style=for-the-badge&logo=instagram&logoColor=white" />
  <img src="https://img.shields.io/badge/Facebook-Messages-1877F2?style=for-the-badge&logo=facebook&logoColor=white" />
  <img src="https://img.shields.io/badge/Zoom-Meetings-2D8CFF?style=for-the-badge&logo=zoom&logoColor=white" />
  <img src="https://img.shields.io/badge/Google_Calendar-4285F4?style=for-the-badge&logo=googlecalendar&logoColor=white" />
  <img src="https://img.shields.io/badge/Google_Sheets-34A853?style=for-the-badge&logo=googlesheets&logoColor=white" />
  <img src="https://img.shields.io/badge/Gmail-EA4335?style=for-the-badge&logo=gmail&logoColor=white" />
</p>

---

## Author

**[Your Name]** — AI Engineer  
Multi-agent automation, LLM orchestration, n8n workflow engineering

[![GitHub](https://img.shields.io/badge/GitHub-TODO-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/TODO)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-TODO-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/TODO)
[![Email](https://img.shields.io/badge/Email-TODO-EA4335?style=for-the-badge&logo=gmail&logoColor=white)](mailto:TODO)

---

> **Open to freelance automation work** — AI agents, n8n workflows, lead capture & meeting booking systems. Reach out via the links above.

---

<footer align="center">

License: TODO — add your preferred license  
⭐ **Star this repo** if you found it useful

</footer>
