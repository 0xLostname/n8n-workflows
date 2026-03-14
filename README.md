# n8n GitHub Automation Workflows

7 ready-to-import n8n workflows connecting GitHub to Discord, Gmail, Google Sheets, and Telegram.

## Import Instructions

1. Open your n8n instance
2. Go to **Workflows** → top-right menu (⋮) → **Import from File**
3. Select any `.json` file from this folder
4. Configure credentials (see each workflow below)
5. Set up the GitHub webhook pointing to your n8n webhook URL
6. Activate the workflow

---

## Workflows

### 01 — GitHub PR → Discord Notification
**Trigger:** GitHub webhook (`pull_request` event)
Fires a rich Discord embed whenever a new PR is opened — shows author, branch, additions/deletions, and a direct link.

**Credentials needed:**
- Discord Webhook

**GitHub webhook setup:**
- Payload URL: `https://YOUR_N8N/webhook/github-pr`
- Content type: `application/json`
- Events: `Pull requests`

---

### 02 — GitHub Issues → Google Sheets Logger
**Trigger:** GitHub webhook (`issues` event)
Every new issue gets appended as a row in your Google Sheet — title, author, labels, assignees, URL, and description.

**Credentials needed:**
- Google Sheets OAuth2
- Discord Webhook (for confirmation message)

**Setup:**
- Create a Google Sheet with columns: `Issue #, Title, Author, Labels, Assignees, State, URL, Repository, Created At, Description`
- Replace `YOUR_GOOGLE_SHEET_ID` in the node with your sheet's ID (from the URL)
- GitHub webhook events: `Issues`

---

### 03 — GitHub Release → Gmail Announcement
**Trigger:** GitHub webhook (`release` event)
Sends a formatted HTML release email when a new version is published, plus a Discord announcement.

**Credentials needed:**
- Gmail OAuth2
- Discord Webhook

**Setup:**
- Replace `YOUR_EMAIL@gmail.com` with your address
- GitHub webhook events: `Releases`

---

### 04 — Daily GitHub Stats Digest → Discord
**Trigger:** Cron schedule — weekdays at 9am
Posts a morning summary to Discord: open PR count, stale PRs (>7 days old), open issues, and oldest items.

**Credentials needed:**
- GitHub Token (HTTP Header Auth — set header `Authorization: token YOUR_GITHUB_TOKEN`)
- Discord Webhook

**Setup:**
- Replace `YOUR_USERNAME/YOUR_REPO` in both HTTP Request nodes
- Adjust the cron expression if you want a different time (`0 9 * * 1-5` = 9am Mon–Fri)

---

### 05 — GitHub Issue Assigned → Gmail Alert
**Trigger:** GitHub webhook (`issues` event)
Sends you an email when a GitHub issue is assigned specifically to your account — includes priority detection from labels.

**Credentials needed:**
- Gmail OAuth2

**Setup:**
- Replace `YOUR_GITHUB_USERNAME` in the filter node with your exact GitHub username
- Replace `YOUR_EMAIL@gmail.com` with your address
- GitHub webhook events: `Issues`
- Add labels like `critical`, `urgent`, `high`, `medium` to your repo for priority detection

---

### 06 — GitHub CI Failure → Telegram Alert
**Trigger:** GitHub webhook (`workflow_run` event)
Instant Telegram message when any GitHub Actions workflow fails — shows workflow name, branch, commit, who triggered it, and a direct link. Also mirrors to Discord.

**Credentials needed:**
- Telegram Bot API
- Discord Webhook

**Setup:**
- Create a Telegram bot via [@BotFather](https://t.me/BotFather) and get your bot token
- Get your chat ID by messaging [@userinfobot](https://t.me/userinfobot)
- Replace `YOUR_TELEGRAM_CHAT_ID` in the Telegram node
- GitHub webhook events: `Workflow runs`

---

### 07 — Discord Command → Create GitHub Issue
**Trigger:** Discord bot/webhook message containing `!issue`
Type `!issue Fix the login bug | Users can't log in on mobile` in Discord — ARIA creates the GitHub issue and replies with a link.

**Command format:**
```
!issue <title> | <description>
!issue <title>
```

**Credentials needed:**
- GitHub Token (HTTP Header Auth)
- Discord Webhook

**Setup:**
- Replace `YOUR_USERNAME/YOUR_REPO` in the GitHub API node
- Configure your Discord bot to forward messages to the webhook URL
- Webhook URL: `https://YOUR_N8N/webhook/discord-to-github`
- Issues are automatically tagged with label `discord-created`

---

### 08 — Smart PR Review Enforcer
**Trigger:** GitHub webhook (`pull_request` event — opened)

The most powerful PR workflow. When a PR is opened it:
1. Fetches every changed file via the GitHub API
2. Scans file paths against 5 critical zones: **auth**, **payments**, **config/secrets**, **database migrations**, **infrastructure/CI**
3. Calculates a risk level: LOW / MEDIUM / HIGH based on how many critical zones are touched
4. Auto-assigns the right reviewers per zone (configurable owner map)
5. Applies labels like `needs-security-review`, `needs-db-review` automatically
6. Posts a detailed checklist comment directly on the PR with zone-specific warnings
7. Fires a colour-coded Discord alert with risk level and file summary

**Credentials needed:** GitHub Token, Discord Webhook

**Setup:**
- Replace the reviewer usernames in the `CRITICAL_PATHS` owner map inside the Code node
- GitHub webhook events: `Pull requests`
- Requires the label names to already exist in your repo (create them once manually)

---

### 09 — Full Issue Triage Pipeline
**Trigger:** GitHub webhook (`issues` event — opened)

End-to-end automated issue handling. On every new issue it:
1. Scans title + body with keyword scoring to detect category: bug / feature / performance / security / docs / question
2. Calculates a priority score (0–100) from signals like "crash", "production", "data loss", "steps to reproduce"
3. Maps category → owner and auto-assigns the issue
4. Applies two labels: category label + priority label
5. Posts a triage report comment on the issue with SLA due date
6. Logs everything to Google Sheets (Issue #, category, priority score, assignee, SLA)
7. Sends a Discord notification with all triage info
8. If score ≥ 40 (CRITICAL), fires an immediate Telegram alert

**Credentials needed:** GitHub Token, Google Sheets OAuth2, Discord Webhook, Telegram Bot

**Setup:**
- Replace all `YOUR_*_USER` placeholders in the `OWNER_MAP` inside the Code node with real GitHub usernames
- Replace `YOUR_GOOGLE_SHEET_ID` — sheet needs columns: `Issue #, Title, Author, Category, Priority, Priority Score, Assigned To, SLA Due, URL, Created At, Status`
- GitHub webhook events: `Issues`

---

### 10 — Release Pipeline Orchestrator
**Trigger:** GitHub webhook (`release` event — published)

Full release broadcast pipeline. On every published release it:
1. Fetches the last 2 releases and recent commits from the GitHub API in parallel
2. Parses commit messages using Conventional Commits format (feat:, fix:, perf:, docs:, refactor:, chore:)
3. Groups commits into sections and builds a full Markdown changelog
4. Calculates stats: total commits, feature count, bug fix count, unique contributors
5. Sends a rich HTML email via Gmail with stats dashboard + full changelog
6. Posts a Discord announcement with a one-line summary
7. Logs the release to Google Sheets for trend tracking
8. Sends a Telegram message with key stats

**Credentials needed:** GitHub Token, Gmail OAuth2, Discord Webhook, Google Sheets OAuth2, Telegram Bot

**Setup:**
- Replace `YOUR_EMAIL@gmail.com` with your address
- Sheets needs columns: `Tag, Name, Published At, Author, Commits, Features, Bug Fixes, Contributors, Pre-release, URL`
- GitHub webhook events: `Releases`

---

### 11 — Repo Health Monitor
**Trigger:** Cron — every Monday at 8am

Weekly automated health audit. Pulls 5 data sources in parallel every Monday:
1. Repo info (stars, forks)
2. All open PRs
3. All open issues
4. Last 20 CI runs
5. Last 30 commits

Then calculates a **health score out of 100** with weighted penalties:
- CI pass rate below 100% → up to -30 points
- PRs older than 30 days → up to -20 points
- Unanswered issues → up to -15 points
- Bus factor ≤ 2 → up to -15 points
- No commits this week → -10 points
- High open issue count → up to -10 points

Outputs a letter grade (A–F), a list of recommendations, and:
- Posts the full report to Discord every week
- Logs all metrics to Google Sheets for trend graphing over time
- If score ≤ 60, sends an immediate Telegram alert

**Credentials needed:** GitHub Token, Discord Webhook, Google Sheets OAuth2, Telegram Bot

**Setup:**
- Replace `YOUR_USERNAME/YOUR_REPO` in all 5 HTTP Request nodes
- Sheets needs columns: `Week, Score, Grade, Open PRs, Stale PRs, Open Issues, Unanswered Issues, CI Pass Rate %, Commits (7d), Commits (30d), Bus Factor, Stars, Forks, Penalties`
- Adjust the cron (`0 8 * * 1` = Monday 8am) to your timezone

| Credential Name | Type | Used In |
|---|---|---|
| `Discord Webhook` | Discord Webhook API | 01, 02, 03, 04, 06, 07 |
| `Google Sheets OAuth2` | Google Sheets OAuth2 | 02 |
| `Gmail OAuth2` | Gmail OAuth2 | 03, 05 |
| `GitHub Token` | HTTP Header Auth (`Authorization: token ghp_xxx`) | 04, 07 |
| `Telegram Bot` | Telegram API | 06 |

---

## GitHub Webhook Tips

- Go to your repo → **Settings → Webhooks → Add webhook**
- Set content type to `application/json`
- For each workflow, only enable the specific events listed — don't use "Send me everything" or you'll get noise
- The webhook URL format is: `https://YOUR_N8N_DOMAIN/webhook/WEBHOOK_PATH`

---

## Placeholders to Replace

Search each file for these and replace before activating:

| Placeholder | Replace with |
|---|---|
| `YOUR_EMAIL@gmail.com` | Your Gmail address |
| `YOUR_GITHUB_USERNAME` | Your GitHub username |
| `YOUR_USERNAME/YOUR_REPO` | e.g. `0xLostname/ARIA-AI-Assistant` |
| `YOUR_GOOGLE_SHEET_ID` | ID from your sheet URL |
| `YOUR_TELEGRAM_CHAT_ID` | Your Telegram chat/user ID |
