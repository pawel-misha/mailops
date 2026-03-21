# MailOps UI/UX Redesign — Master Plan

> Created: 2026-03-21
> Status: Planning complete, implementation not started
> Start with Steps 1–3 as a continuous block, then 4–6 sequentially.

---

## Implementation Order

| Step | Feature | Complexity | Depends on | Status |
|---|---|---|---|---|
| 1 | Layout Rework | High | — | [ ] |
| 2 | Email List | Medium | Step 1 | [ ] |
| 3 | Reading Pane | Medium | Step 2 | [ ] |
| 4 | Daily Intelligence Dashboard | Medium | Steps 1–3 | [ ] |
| 5 | Thread Synthesis Banners | Low | Step 3 | [ ] |
| 6 | Gatekeeper / Semantic Quarantine | High | Step 4 | [ ] |

---

## Step 1 — Layout Rework

**Goal:** Replace the single-column chat layout with a three-column email client structure.

### New structure

```
+----------+--------------------------+-------------------------+
| Sidebar  | Center Panel             | Right Panel             |
| 200px    | flex: 1                  | 420px                   |
| fixed    | email list OR dashboard  | reading pane OR AI chat |
+----------+--------------------------+-------------------------+
| Chat input bar — full width, always visible at bottom        |
+--------------------------------------------------------------+
```

### Details

- Sidebar stays as-is (quick actions, labels) — just narrower
- Center panel has two modes: `dashboard` (default on load) and `email-list` (when Claude returns results)
- Right panel has two modes: `ai-chat` (default) and `reading-pane` (when email is clicked) — toggled by a button in topbar
- Chat input stays full-width at bottom, always visible
- AI chat messages move into the right panel
- On screens <1100px: center panel hides, right panel goes full width

### CSS changes

- `.main-layout` becomes CSS grid: `200px 1fr 420px`
- New `.center-panel`, `.right-panel` components
- Right panel toggle: `#panel-mode` = `chat` | `reader`
- Smooth transitions when switching modes

### JS changes

- `showCenterPanel(mode)` — switches between `dashboard` / `email-list`
- `showRightPanel(mode)` — switches between `chat` / `reader`
- Email list results from `search_emails` now populate center panel instead of (or alongside) chat

---

## Step 2 — Email List (Center Panel)

**Goal:** Compact, scannable email list like Superhuman — populated automatically when Claude searches.

### Row anatomy

```
  [avatar] Sender name        Subject — snippet preview...       date  *
  unread   initials/letter    bold if unread                     time
```

### Features

- Unread dot on left
- Sender avatar = coloured circle with first letter of sender name
- Subject **bold** if unread, normal if read
- Snippet preview in muted colour
- Date/time on right (today = time, older = date)
- Star icon on far right
- Hover: row highlights, quick action icons appear (archive, trash)
- Click: opens email in reading pane (right panel switches to `reader` mode)
- Selected row: amber left border

### Pagination

- Show 20 rows by default
- "Show 20 more" button at bottom of list
- Total count shown at top: `Showing 20 of 47`
- List header shows the search query that produced it: `Results for: is:unread in:inbox`

### JS changes

- `renderEmailList(emails, query, total)` — builds list in center panel
- `loadMoreEmails()` — appends next page
- Intercept `search_emails` tool result → call `renderEmailList()` alongside existing chat badge
- `selectEmail(messageId)` → triggers `read_email` and switches to reading pane

---

## Step 3 — Reading Pane (Right Panel)

**Goal:** Full email view in right panel when an email is clicked from the list.

### Layout

```
+-----------------------------------------+
| <- Back   Subject line             * ... |  <- header bar
+-----------------------------------------+
| From: Name <email>                       |
| To: me · Date · Labels                   |
+-----------------------------------------+
| [Thread Synthesis banner — Step 5]       |  <- AI summary strip
+-----------------------------------------+
|                                          |
|  <iframe sandbox> email HTML body </if>  |  <- sandboxed iframe
|                                          |
+-----------------------------------------+
| [Reply] [Forward] [Archive] [Trash]      |  <- quick actions
+-----------------------------------------+
```

### Details

- Email body rendered in sandboxed `<iframe>`: `sandbox="allow-same-origin"` — no scripts, no external forms
- Plain text fallback if no HTML body
- "Back" returns to email list, restores scroll position
- Quick action buttons call existing tools directly (no Claude needed): `archive_emails`, `trash_emails`, `star_emails`
- Reply button: pre-fills chat input with `Reply to this email: [messageId]` and focuses input
- Thread badge: if email is part of a thread, shows count; clicking loads full thread via `read_thread`
- External images blocked by default, "Show images" button to allow
- Loading skeleton while `read_email` fetches body

### JS changes

- `openEmail(messageId, threadId)` — fetches full email, renders reading pane
- `closeReader()` — returns to list
- `quickAction(action, messageId)` — direct tool call bypassing Claude
- Iframe `srcdoc` constructed from email HTML with injected base CSS for readability

---

## Step 4 — Daily Intelligence Dashboard (Center Panel default)

**Goal:** Replace the welcome screen with an AI-generated daily briefing shown on app load.

### Layout

```
+-----------------------------------------+
| Good morning, Pawel  Fri 21 Mar     [↻] |
| You have 201 unread · 4 need action     |
+-----------------------------------------+
| PRIORITY                                |
| * Reply needed from Marta re: budget    |
| * Invoice from Kowalski — due today     |
| * Google security alert — review        |
+-----------------------------------------+
| TODAY'S BREAKDOWN                       |
| Urgent   3      FYI      12            |
| Promos  47      Events    2            |
+-----------------------------------------+
| SUGGESTED ACTIONS                       |
| [Clean 47 promos]  [Summarise urgent]   |
+-----------------------------------------+
```

### Generation logic

- Triggered automatically on app load (after auth)
- Runs 3–4 parallel Gmail searches: unread, flagged, recent, today's date range
- Sends combined results to Claude with a briefing-generation prompt
- Cached in `sessionStorage` — regenerated only on manual refresh (the refresh button)
- Clicking a priority item opens that email in reading pane
- Clicking a suggested action fires that query into chat

### JS changes

- `generateDailyIntelligence()` — async, runs after `loadProfile()`
- Shows skeleton loader while generating
- `renderDashboard(data)` — builds the HTML
- Refresh button top-right regenerates and re-caches

---

## Step 5 — Thread Synthesis Banners

**Goal:** AI-generated key-facts strip shown at top of reading pane for each email opened.

### Appearance

```
+- AI Summary ----------------------------------------+
| Invoice #2341 · EUR 4,200 · Due: 31 Mar             |
| Action needed: Approve or reject by end of week      |
+------------------------------------------------------+
```

### Details

- Generated the first time an email is opened (single Claude call, no tools needed)
- Cached in `sessionStorage` by `messageId` — not re-generated on revisit within same session
- Prompt instructs Claude to extract: amounts, dates, names, required actions, deadlines — max 2 lines
- Shows loading shimmer while generating
- Dismissable per-email (small X button)
- Estimated cost: ~$0.002 per email opened

### JS changes

- `synthesiseThread(messageId, emailContent)` — async, called after `openEmail()`
- `threadSynthesisCache` — `Map<messageId, summaryText>` stored in sessionStorage
- Banner injected into reading pane header area before iframe

---

## Step 6 — Gatekeeper (Semantic Quarantine)

**Goal:** Emails from unknown/suspicious senders get AI-evaluated before reaching the main inbox view.

### How it works

1. During Daily Intelligence generation, also scans recent emails from senders not in previous correspondence
2. Claude evaluates each: `legitimate` / `promotional` / `suspicious` / `phishing`
3. A "Quarantine" item appears in the sidebar with a count badge
4. Clicking shows the quarantine list in center panel
5. Each row shows: AI verdict, confidence score, reason
6. User actions per email: `Let through` / `Mark spam` / `Block sender` / `Delete`

### Detection signals passed to Claude

- Sender not found in any previously sent/received thread
- Display name vs actual email domain mismatch
- Urgency language patterns
- Link-heavy with minimal personal content
- Requests for credentials or payment

### UI

```
Sidebar:
  Shield  Quarantine  [3]   <- count badge

Center panel (quarantine view):
  +----------------------------------------------+
  | SUSPICIOUS  noreply@xn--paypa1.com           |
  | "Your account suspended" · 94% phishing       |
  | [Let through] [Spam] [Block] [Delete]         |
  +----------------------------------------------+
  | PROMO  deals@shop.example.com                |
  | "50% off today only" · Marketing             |
  | [Let through] [Spam] [Delete]                |
  +----------------------------------------------+
```

### JS changes

- `runGatekeeper()` — called during Daily Intelligence, runs on last 24h emails from unknown senders
- `renderQuarantine(results)` — builds quarantine center panel view
- Results cached in `sessionStorage`
- Sidebar badge updates dynamically

---

## Notes & Decisions

- Steps 1–3 must be implemented together as one block (layout without list/reader is incomplete)
- Steps 4–6 are independent of each other once 1–3 are done
- All AI calls in Steps 4–6 should track token usage via the existing `sessionUsage` system
- Prompt caching applies to all new Claude calls (same ephemeral cache_control pattern)
- Gatekeeper should have an on/off toggle in Settings drawer (some users may not want it)
