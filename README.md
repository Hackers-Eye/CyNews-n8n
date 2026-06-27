# 📡 Cyber News → AI Reel → Telegram

An n8n workflow that automatically fetches the latest cybersecurity news every morning, generates a short AI voiceover video, and posts it directly to a Telegram channel — fully hands-free.

---

## 🔄 How It Works

```
Daily 9AM → RSS Feed → Pick Article → Gemini Script → JSON2Video Render → Telegram Post
```

| Step | Node | Description |
|------|------|-------------|
| 1 | **Daily 9AM** | Schedule trigger fires at 09:00 every day |
| 2 | **Read Cyber News** | Fetches latest feed from [The Hacker News](https://thehackernews.com) via RSS |
| 3 | **Pick Latest Article** | Strips HTML, extracts title, summary, link, and pub date |
| 4 | **Gemini Write Script** | Uses Gemini 2.5 Flash to write a 70–90 word voiceover script + social caption |
| 5 | **Parse Script JSON** | Cleans and parses Gemini's JSON response safely |
| 6 | **JSON2Video Submit** | Renders a vertical (Instagram Story) video with text overlay + AI voice |
| 7 | **Wait + Poll Loop** | Waits 30s, then polls JSON2Video until render status is `done` |
| 8 | **Telegram Post** | Sends the finished video + caption to your Telegram channel |

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

In the **Telegram Post** node, replace the `chatId` value with your own channel or chat ID.  
You can find your chat ID by messaging your bot and checking `https://api.telegram.org/bot<TOKEN>/getUpdates`.

### 4. Activate

Toggle the workflow to **Active**. It will run automatically at 9AM daily.

---

## 📦 Tech Stack

- **n8n** — workflow automation
- **The Hacker News RSS** — cybersecurity news source
- **Google Gemini 2.5 Flash** — AI script & caption generation
- **JSON2Video** — programmatic video rendering with AI voice (`en-US-AriaNeural`)
- **Telegram Bot API** — video distribution

---

## ⚠️ Known Limitations

- **Duplicate posts** — The workflow always picks the first RSS item. If the feed hasn't updated, the same article may be posted twice. Consider adding a deduplication step using n8n's static data or a database node.
- **No render timeout** — The polling loop has no retry limit. If JSON2Video gets stuck, the workflow will run indefinitely. Add a counter node to cap retries (e.g., max 10 attempts).
- **30s initial wait** — Complex renders may take longer. Consider increasing the wait or making the retry cap configurable.

---

## 🗂 File Structure

```
├── Cyber_News___AI_Reel___Telegram.json   # n8n workflow export
└── README.md                              # This file
```

---

## 📄 License

MIT — feel free to fork, modify, and build on this workflow.
