# AI Job Search Automation

Finds jobs every morning, scores each one against your profile with AI, saves the good matches, emails you a digest, and feeds a live dashboard — all running unattended on n8n.

```
8 AM daily → fetch jobs (4 sources) → AI scores each → keep fit ≥ 50 → save → email you
```

---

## What you get

| File | What it does | You edit it? |
|---|---|---|
| `AI_Job_Search_Automation.json` | The n8n workflow — import this | No (just fill placeholders in the app) |
| `SETUP_GUIDE.md` | Zero-to-running, step by step | No — follow it |
| `IMPLEMENTATION_GUIDE.md` | Deep reference: every node, the AI prompt, cost math | No — reference |
| `dashboard.html` | Live dashboard, opens in any browser | No |
| `schema.sql` | Optional PostgreSQL database | Only if you use Postgres |
| `candidate_profile_template.csv` | Headers for your profile sheet | Fill in your details |
| `jobs_output_headers.csv` | Headers for the output sheet | Paste as row 1 |

---

## Quick start (≈30 minutes)

1. **Get accounts/keys:** n8n, an OpenAI API key, a free Adzuna app_id + app_key, and a Google account.
2. **Make the Google Sheet:** two tabs — `Profile` (your details) and `Jobs` (empty, just headers).
3. **Import** `AI_Job_Search_Automation.json` into n8n.
4. **Add 3 credentials** in n8n: Google Sheets, Gmail, and OpenAI (Header Auth).
5. **Fill the placeholders** (`YOUR_…`) in a few nodes: sheet IDs, Adzuna keys, your email.
6. **Run once** to test, then flip the workflow to **Active**.

The full, click-by-click version is in **`SETUP_GUIDE.md`** — start there.

---

## Requirements

- An **n8n** instance (n8n Cloud, or self-hosted via Docker — both covered in the setup guide)
- **OpenAI API key** with a small prepaid balance
- **Adzuna** developer account (free) for the `app_id` / `app_key`
- A **Google account** (Sheets + Gmail)
- RemoteOK, Arbeitnow, Remotive need **no keys**

---

## What it costs

| Item | Cost |
|---|---|
| OpenAI (gpt-4o-mini, ~40 jobs/day) | **~$0.50 / month** |
| RemoteOK · Arbeitnow · Remotive · Adzuna | Free |
| Google Sheets + Gmail | Free |
| n8n self-hosted (tiny VPS) | ~$5 / month, or free on your own machine |
| n8n Cloud (if you prefer managed) | ~$20–24 / month |

**All-in: under ~$2/month self-hosted.** The pre-filter discards out-of-scope jobs before any AI tokens are spent — that's the main cost lever.

---

## How matching works

Each job is scored 0–100 on five dimensions (skills, experience, location, salary, growth), combined into one weighted **fit score**. Jobs scoring **≥ 50** are kept; the rest are dropped. The AI also returns your **matching** and **missing** skills per job, so the dashboard doubles as an upskilling backlog. Tune the threshold in one node: 60 = strict, 50 = balanced, 40 = high volume.

---

## Job sources

Built on **official, permitted APIs** — not site scraping:

- **RemoteOK, Arbeitnow, Remotive** — free, no key, remote-focused
- **Adzuna** — free key, aggregates many boards across 12+ countries (your Indeed-style breadth)

LinkedIn/Indeed/Glassdoor block automated access and prohibit scraping in their terms, so they're left out by design. If you need them, route through a licensed aggregator (SerpApi Google Jobs, Bright Data, Apify) — adding a source is one HTTP node plus one branch in the `Normalize Jobs` node.

---

## Troubleshooting (quick)

| Symptom | Likely fix |
|---|---|
| No email arrives | Check the Gmail node's `sendTo`, and that the run produced ≥ 1 job with fit ≥ 50 |
| `401` on the OpenAI node | Header Auth value must be `Bearer sk-…` (the word `Bearer`, a space, then the key) |
| `Fetch Adzuna` fails | `app_id` / `app_key` placeholders not replaced, or daily free quota hit |
| Sheet rows duplicate | The save node must use **Append or update** matching on `job_key` |
| One source errors but run continues | Expected — source nodes continue on error so the others still flow |
| Workflow never fires | It must be toggled **Active**, and the host timezone set so `0 8 * * *` = 8 AM local |

Full troubleshooting and node-by-node detail live in `IMPLEMENTATION_GUIDE.md` (§11).

---

*Replace every `YOUR_…` placeholder before going Active. Follow `SETUP_GUIDE.md` next.*
