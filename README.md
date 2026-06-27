# 📡 Cyber News → AI Reel → Telegram

An n8n workflow that automatically fetches the latest cybersecurity news every morning, generates a short AI voiceover video, and posts it to a Telegram channel — fully hands-free.

---

## 🔄 Pipeline Flowchart

> Step-by-step execution flow, including the polling loop for video rendering.

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Daily 9AM  │───▶│ Cyber News  │───▶│ Pick Article│───▶│ Gemini 2.5  │───▶│ Parse JSON  │
│  Scheduler  │    │  RSS Feed   │    │ Strip HTML  │    │ Write Script│    │ Clean output│
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘    └──────┬──────┘
                                                                                    │
              ┌─────────────────────────────────────────────────────────────────────┘
              │
              ▼
       ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
       │Video Submit │───▶│  Wait 30s   │───▶│ Video Poll  │
       │  POST API   │    │    Pause    │    │  GET status │
       └─────────────┘    └─────────────┘    └──────┬──────┘
                                                     │
                                              ┌──────▼──────┐
                                    ┌─false───│ Render done?│
                                    │         │check status │
                                    │         └──────┬──────┘
                                    │                │ true
                                    ▼                ▼
                               (loop back)    ┌─────────────┐    ┌─────────────┐
                               to Wait 30s    │Prep Publish │───▶│Telegram Post│
                                              │Extract URL  │    │Send video   │
                                              └─────────────┘    └─────────────┘
```

---

## 🏗️ Architecture Diagram

> The three logical layers and the external services involved.

```
┌──────────────────────┐   ┌──────────────────────────┐   ┌──────────────────────┐
│    DATA SOURCES      │   │      AI PROCESSING        │   │    DISTRIBUTION      │
│                      │   │                           │   │                      │
│  ┌────────────────┐  │   │  ┌────────────────────┐  │   │  ┌────────────────┐  │
│  │  Hacker News   │  │   │  │  Gemini 2.5 Flash  │  │   │  │  Telegram Bot  │  │
│  │   RSS feed     │──┼───┼─▶│  Script + caption  │──┼───┼─▶│  sendVideo API │  │
│  └────────────────┘  │   │  └─────────┬──────────┘  │   │  └───────┬────────┘  │
│                      │   │            │              │   │          │           │
│  ┌────────────────┐  │   │  ┌─────────▼──────────┐  │   │  ┌───────▼────────┐  │
│  │ n8n Scheduler  │  │   │  │    JSON2Video       │  │   │  │Channel / Chat  │  │
│  │  Cron 9AM      │  │   │  │  Render + AI voice  │  │   │  │ End audience   │  │
│  └────────────────┘  │   │  └─────────┬──────────┘  │   │  └────────────────┘  │
│                      │   │            │              │   │                      │
└──────────────────────┘   │  ┌─────────▼──────────┐  │   └──────────────────────┘
                           │  │   Aria Neural       │  │
                           │  │  en-US voiceover    │  │
                           │  └────────────────────┘  │
                           └──────────────────────────┘

         ╔══════════════════════════════════════════════════════════════════╗
         ║           n8n orchestration layer                               ║
         ║   Coordinates all nodes, handles polling loop and error routing ║
         ╚══════════════════════════════════════════════════════════════════╝
```

---

## 📋 Node Reference

| Step | Node | Type | Description |
|------|------|------|-------------|
| 1 | **Daily 9AM** | Schedule Trigger | Fires at 09:00 every day |
| 2 | **Read Cyber News** | RSS Feed Read | Fetches [The Hacker News](https://thehackernews.com) |
| 3 | **Pick Latest Article** | Code (JS) | Strips HTML, extracts title, summary, link, date |
| 4 | **Gemini Write Script** | Google Gemini | Generates 70–90 word voiceover script + caption as JSON |
| 5 | **Parse Script JSON** | Code (JS) | Cleans and safely parses Gemini's response |
| 6 | **JSON2Video Submit** | HTTP Request | POSTs render job — Instagram Story format, Aria Neural voice |
| 7 | **Wait 30s** | Wait | Initial pause before polling |
| 8 | **JSON2Video Poll** | HTTP Request | GETs render status by project ID |
| 9 | **Render Done?** | IF | Checks `movie.status === "done"`; loops back if not |
| 10 | **Prep Publish** | Code (JS) | Extracts final video URL and caption |
| 11 | **Telegram Post** | Telegram | Sends video + caption to your channel |

---

## 🧰 Prerequisites

- [n8n](https://n8n.io) (self-hosted or cloud)
- API keys / credentials for:
  - **Google Gemini** (PaLM API) — script generation
  - **JSON2Video** — video rendering
  - **Telegram Bot** — channel posting

---

## 🚀 Setup

### 1. Import the Workflow

In n8n, go to **Workflows → Import** and upload `Cyber_News___AI_Reel___Telegram.json`.

### 2. Configure Credentials

Set up the following credentials in n8n (**Settings → Credentials**):

| Credential | Used By | Notes |
|------------|---------|-------|
| `Google Gemini (PaLM) API` | Gemini Write Script | Get key from [Google AI Studio](https://aistudio.google.com) |
| `HTTP Header Auth` (JSON2Video) | JSON2Video Submit & Poll | Add your JSON2Video API key as an `x-api-key` header |
| `Telegram API` | Telegram Post | Create a bot via [@BotFather](https://t.me/BotFather) |

### 3. Update the Telegram Chat ID

In the **Telegram Post** node, replace `chatId` with your own channel or chat ID.
You can find yours by messaging your bot and checking:
```
https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates
```

### 4. Activate

Toggle the workflow to **Active**. It will run automatically at 9AM daily.

---

## 📦 Tech Stack

| Service | Role |
|---------|------|
| **n8n** | Workflow automation & orchestration |
| **The Hacker News RSS** | Cybersecurity news source |
| **Google Gemini 2.5 Flash** | AI script & social caption generation |
| **JSON2Video** | Programmatic video rendering |
| **Aria Neural** (`en-US-AriaNeural`) | AI voiceover |
| **Telegram Bot API** | Video distribution |

---

## ⚠️ Known Limitations

- **Duplicate posts** — always picks `items[0]` from the feed. If the feed hasn't refreshed, the same article gets posted twice. Add deduplication using n8n's static data or a database node.
- **No render timeout** — the polling loop has no retry cap. If JSON2Video stalls, the workflow runs indefinitely. Add a counter node to limit retries (e.g. max 10 attempts).
- **30s wait may be too short** — complex renders can take longer. Consider increasing to 60s or making it configurable.
- **Silent parse failures** — if Gemini's JSON can't be parsed, `caption` and `title` fall back to empty strings silently.

---

## 🗂 File Structure

```
├── Cyber_News___AI_Reel___Telegram.json   # n8n workflow export
└── README.md                              # This file
```

---

## 📄 License

MIT — feel free to fork, modify, and build on this workflow.
