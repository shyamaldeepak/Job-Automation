# Setup Guide — step by step

Follow these in order. Each step ends with a **✓ Check** so you know it worked before moving on. Total time: ~30–40 minutes the first time.

Files referenced here are the ones delivered alongside this guide.

---

## Phase 1 — Get your accounts and keys

### Step 1. OpenAI API key
1. Go to **platform.openai.com** and sign in.
2. Open **API keys** → **Create new secret key**. Copy it somewhere safe (you can't see it again).
3. Go to **Billing**, add a small prepaid balance ($5 is plenty), and set a **monthly budget limit** + email alert.

**✓ Check:** you have a key starting with `sk-…`.

### Step 2. Adzuna keys (free)
1. Go to **developer.adzuna.com** → register.
2. Create an application. Copy the **App ID** and **App Key**.

**✓ Check:** you have both an `app_id` and an `app_key`.

### Step 3. Google account
You'll use Google **Sheets** (storage) and **Gmail** (sending). Any normal Google account works — no setup needed yet.

**✓ Check:** you can open sheets.google.com.

### Step 4. n8n
Pick one:

- **Easiest — n8n Cloud:** sign up at **n8n.io**, click **Get started**. Done.
- **Free — self-host with Docker:**
  ```bash
  docker volume create n8n_data
  docker run -d --name n8n -p 5678:5678 \
    -e GENERIC_TIMEZONE="Europe/London" \
    -e TZ="Europe/London" \
    -v n8n_data:/home/node/.n8n \
    docker.n8n.io/n8nio/n8n
  ```
  Then open **http://localhost:5678**. Set `GENERIC_TIMEZONE`/`TZ` to your timezone so 8 AM means *your* 8 AM. For 24/7 running, host it on an always-on VM (Railway, Fly.io, a small VPS) rather than your laptop.

**✓ Check:** the n8n dashboard loads in your browser.

---

## Phase 2 — Build the Google Sheet

### Step 5. Create the spreadsheet and two tabs
1. New Google Sheet. Name it **Job Search**.
2. Rename the first tab to **`Profile`**.
3. Add a second tab named **`Jobs`**.

### Step 6. Fill the Profile tab
1. Open `candidate_profile_template.csv` and copy its two rows.
2. Paste into the `Profile` tab so **row 1 = headers**, **row 2 = your details**:
   `name, skills, experience, preferred_roles, preferred_countries, preferred_salary, preferred_work_type`
3. Edit row 2 with your real info. Keep `skills`, `preferred_roles`, `preferred_countries` comma-separated. The **first** country drives which Adzuna region is searched.

### Step 7. Prepare the Jobs tab
1. Open `jobs_output_headers.csv`, copy the single header row.
2. Paste it into **row 1** of the `Jobs` tab. Leave the rest empty — the workflow fills it.

### Step 8. Grab your Sheet ID
The Sheet ID is the long string in the URL between `/d/` and `/edit`:
`https://docs.google.com/spreadsheets/d/`**`THIS_PART`**`/edit`
Copy it — you'll paste it into two nodes later.

**✓ Check:** `Profile` has your data, `Jobs` has only headers, and you've copied the Sheet ID.

---

## Phase 3 — Import the workflow

### Step 9. Import the JSON
1. In n8n: **Workflows** → **Import from File** (or the **⋯** menu → Import).
2. Choose `AI_Job_Search_Automation.json`.

**✓ Check:** you see a canvas with ~20 connected nodes, starting at **Daily 8AM Trigger**.

---

## Phase 4 — Create credentials

You'll make three credentials once, then attach them to nodes.

### Step 10. Google Sheets credential
1. **Credentials** → **New** → search **Google Sheets OAuth2**.
2. Follow the connect flow and authorize your Google account.

### Step 11. Gmail credential
1. **New** → **Gmail OAuth2** → authorize the same Google account.

### Step 12. OpenAI credential (Header Auth)
1. **New** → search **Header Auth**.
2. Set **Name** = `Authorization`.
3. Set **Value** = `Bearer ` + your key, e.g. `Bearer sk-abc123…` (the word **Bearer**, one space, then the key).
4. Save it as e.g. "OpenAI Bearer".

**✓ Check:** three credentials exist — Google Sheets, Gmail, Header Auth.

---

## Phase 5 — Fill in the placeholders

Open each node below and replace its `YOUR_…` values. (Double-click a node to open it.)

### Step 13. Load Candidate Profile
- **Credential:** pick your Google Sheets credential.
- **Document ID:** paste your Sheet ID.
- **Sheet name:** `Profile`.

### Step 14. Fetch Adzuna
- In **Query Parameters**, replace `YOUR_ADZUNA_APP_ID` and `YOUR_ADZUNA_APP_KEY` with your two Adzuna values.

### Step 15. OpenAI Score Job
- **Credential / Authentication:** select your **Header Auth** credential.
- Leave the model (`gpt-4o-mini`) and body as-is.

### Step 16. Save to Google Sheets
- **Credential:** your Google Sheets credential.
- **Document ID:** your Sheet ID.
- **Sheet name:** `Jobs`.
- Confirm the operation is **Append or update** matching on **`job_key`** (already set).

### Step 17. Send Daily Email
- **Credential:** your Gmail credential.
- **To:** your email address (replace `YOUR_EMAIL@example.com`).

### Step 18. Email Error Alert
- **Credential:** your Gmail credential.
- **To:** your email address.

**✓ Check:** no node still shows a `YOUR_…` value or a missing-credential warning.

---

## Phase 6 — Test, then schedule

### Step 19. Run it once manually
1. Click **Test workflow** (or **Execute Workflow**).
2. Watch the nodes light up left to right. Open a few to inspect their output:
   - `Normalize Jobs` → a list of jobs in a clean shape.
   - `Dedupe + Pre-filter` → fewer jobs (out-of-scope ones removed).
   - `Parse AI Result` → each job now has a `fit_score` and `missing_skills`.

**✓ Check:** the `Jobs` tab has new rows, and a digest email arrives in your inbox.

### Step 20. Turn on the schedule
1. Toggle the workflow to **Active** (top-right).
2. In **Settings**, set this workflow as its own **Error Workflow** so failures email you.

**✓ Check:** the workflow shows **Active**. It will now run every day at 8 AM your time.

---

## Phase 7 — Wire up the dashboard

### Step 21. Publish the Jobs tab
1. In Google Sheets: **File → Share → Publish to web**.
2. Choose the **`Jobs`** tab and format **CSV**. Publish, then copy the link.

### Step 22. Open the dashboard
1. Open `dashboard.html` in any browser (double-click it).
2. Paste the published CSV link into the input box and click **Load**.

**✓ Check:** the KPI cards and charts fill with your real job data. (It opens on sample data first, so it's never blank.)

---

## If something doesn't work

| Problem | Fix |
|---|---|
| OpenAI node `401 Unauthorized` | The Header Auth value must be exactly `Bearer ` + key |
| `Fetch Adzuna` red/erroring | App ID/Key not replaced, or you hit the free daily quota — try again tomorrow |
| No email | Confirm `sendTo` is your address, and that ≥ 1 job scored ≥ 50 this run |
| Email arrives but is empty | No jobs cleared the threshold today — lower it in the **Fit Score ≥ 50** node to 40 |
| Sheet rows duplicating | Save node must be **Append or update** on `job_key` |
| Schedule never fires | Workflow must be **Active**; set host timezone so `0 8 * * *` = local 8 AM |
| Dashboard won't load data | The sheet must be **Published to web as CSV** (not just shared by link) |

For anything deeper — adding a job source, switching to PostgreSQL, editing the AI prompt, or cost tuning — see `IMPLEMENTATION_GUIDE.md`.

---

## Optional: use PostgreSQL instead of (or alongside) Sheets

1. Run `schema.sql` on your Postgres database.
2. In n8n, create a **Postgres** credential.
3. Enable the **Save to Postgres (optional)** node, attach the credential, point it at the `jobs` table.

The schema includes ready-made views (`v_dashboard_summary`, `v_best_companies`, `v_skill_gaps`) you can plug into Metabase, Grafana, or Looker Studio.

---

*That's the whole setup. Once it's Active, you do nothing — just read the morning email.*
