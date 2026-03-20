# MailOps ✉️
> AI-powered Gmail command center — manage your entire inbox through natural language chat.

MailOps is a single-file HTML application that combines the Gmail API with Claude AI to give you full conversational control over your Gmail inbox. No backend, no subscription, no data leaving your browser.

---

## Features

### 📬 Read & Search
- Search emails using natural language ("find invoices from last month")
- Read full email content and entire conversation threads
- Get inbox statistics and unread counts by category

### ✉️ Compose & Send
- Write and send new emails
- Reply to threads with full context awareness
- Forward emails with optional notes
- Save, review, and send drafts

### 🗑️ Bulk Cleanup
- Trash emails individually or in bulk by search query
- Permanently delete (e.g. empty trash)
- Archive emails (remove from inbox, keep in All Mail)
- Mark as read/unread in bulk
- Move to spam or restore from spam
- Restore emails from trash

### 🏷️ Labels & Organization
- List, create, and delete labels
- Apply or remove labels from any set of emails
- Star and unstar emails

### 🔧 Filters
- List all active Gmail filter rules
- Create new filters with full criteria and action support (auto-archive, auto-delete, auto-label, etc.)
- Delete filters

### ✍️ Signatures
- View all send-as aliases and their current signatures
- Update signatures (plain text or HTML)
- Clear signatures

### 📎 Attachments
- Download email attachments directly to your browser

### 🤖 Model Selection
- Dynamically fetches available Claude models from the Anthropic API
- Switch between models (Opus, Sonnet, Haiku) at any time from the top bar

---

## How It Works

MailOps runs entirely in your browser. There is no server, no database, and no third-party service beyond Google and Anthropic.

```
Your Browser
    │
    ├── Gmail API (Google OAuth 2.0) — read, write, delete, organize
    │
    └── Anthropic API (Claude) — understands your commands, calls Gmail tools
```

When you type a command, Claude interprets your intent, calls the appropriate Gmail API operations, and reports back — all within the chat interface.

---

## Setup

### Requirements
- A Google account with Gmail
- An [Anthropic API key](https://console.anthropic.com)
- A Google Cloud project with Gmail API enabled and OAuth credentials
- Node.js (for running a local server, one command)

### Google Cloud Setup (~5 minutes, one-time)

1. Go to [console.cloud.google.com](https://console.cloud.google.com) and create a new project
2. Navigate to **APIs & Services → Library**, search for **Gmail API** and enable it
3. Go to **APIs & Services → Credentials → Create Credentials → OAuth Client ID**
4. Select **Web Application** as the application type
5. Under **Authorized JavaScript Origins**, add your deployment URL (e.g. `http://localhost:3000` for local use)
6. Go to **OAuth Consent Screen → Test Users** and add your Gmail address
7. Copy the generated **Client ID**

### Running Locally

```bash
# Serve the file (required — Google OAuth blocks file:// origins)
npx serve .

# Then open in your browser:
# http://localhost:3000/index.html
```

### First Launch

1. Enter your **Anthropic API Key** (`sk-ant-...`)
2. Enter your **Google OAuth Client ID** (`xxxxxxxx.apps.googleusercontent.com`)
3. Click **Connect Gmail** — a Google OAuth popup will appear
4. Authorize access and you're in

---

## Deployment

MailOps is a single HTML file — it can be hosted anywhere static files are served.

### GitHub Pages (recommended)

```bash
git init
git add index.html
git commit -m "Initial MailOps"
git remote add origin https://github.YOUR-DOMAIN.com/YOUR-LOGIN/mailops.git
git push -u origin main
```

Then: **Settings → Pages → Source: main branch → Save**

> ⚠️ After deploying, add your GitHub Pages URL as an additional Authorized JavaScript Origin in your Google Cloud OAuth credentials.

### Other Options
- **Netlify** — drag and drop `index.html` at netlify.com/drop
- **Cloudflare Pages** — connect private GitHub repo, free tier supports private repos
- **Local only** — run `npx serve .` whenever you need it

---

## Cost

| Component | Cost |
|---|---|
| Gmail API | **Free** — no limits for personal use |
| Google Cloud OAuth | **Free** |
| Hosting (GitHub Pages / Netlify / Cloudflare) | **Free** |
| Anthropic API (Claude) | ~$1–5/month for typical personal use |

Claude Sonnet 4.6 is the default model — a good balance of capability and cost. Switch to Haiku for simple operations to reduce API costs further.

---

## Security

- **No backend** — credentials are entered at runtime and never stored server-side
- **OAuth scopes requested:** `gmail.modify`, `gmail.compose`, `gmail.send`, `gmail.labels`, `gmail.settings.basic`
- **Never commit your API keys** to the repository — they are entered in the browser UI only
- For personal use only — not designed for multi-user or public-facing deployment without adding a backend key proxy

---

## Supported Gmail API Operations

| Category | Operations |
|---|---|
| Messages | Search, read, send, reply, forward, trash, delete, archive, restore |
| Status | Mark read/unread, star/unstar, move to spam, restore from spam |
| Labels | List, create, delete, apply to messages |
| Filters | List, create, delete |
| Drafts | Create, list, send, delete |
| Signatures | Get, update, clear (per send-as alias) |
| Attachments | Download to browser |
| Stats | Inbox overview, unread counts by category |

---

## Tech Stack

- **Frontend:** Vanilla HTML/CSS/JS — single file, zero dependencies, zero build step
- **AI:** Anthropic Claude API (tool use / function calling)
- **Email:** Gmail REST API v1
- **Auth:** Google Identity Services (OAuth 2.0 implicit flow)
- **Fonts:** IBM Plex Mono + Syne (Google Fonts)

---

## License

Personal use. Not affiliated with Google or Anthropic.
