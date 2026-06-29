# Pods Win Tracker — Dallas

An automated, public dashboard for **Pod Sales Production in Dallas**. It mirrors the **"Dallas"
tab** of the Google Sheet — exactly as the tab calculates it (no recomputation) — and publishes it
to GitHub Pages.

```
Google Sheet "Dallas" tab
        │   Apps Script reads the tab + stamps the refresh time (every 30 min)
        ▼
   data/dallas.json   ──commit──▶  GitHub repo  ──▶  GitHub Pages (this dashboard)
```

**What you see:** each pod's progress toward its weekly **SRA** (Sunrun Approved) win target, how
many weeks of the quarter the pod has hit its target, per-rep season totals, a week toggle, pod/rep
filters, and a "last refreshed" clock.

- **Weeks run Monday → Sunday, Central Time.** Week 1 = **Mon Jun 29, 2026**.
- A week only becomes viewable once its Monday has arrived. Past weeks keep their baked-in numbers.

---

## Repository layout

```
index.html              Dashboard page
assets/styles.css        Theme (edit the :root variables to rebrand)
assets/app.js            Renders data/dallas.json — display only
data/dallas.json         Published by the Apps Script (seed/sample is committed to start)
apps-script/Code.gs      Reads the Dallas tab → JSON → GitHub
apps-script/appsscript.json
CNAME.example            Rename to CNAME when you add a custom domain
```

---

## Part A — Publish the dashboard (GitHub Pages)

1. **Create a repo** on GitHub named `pods-win-tracker` (Public).
2. **Upload these files** (keep the folder structure). Easiest: *Add file → Upload files*, drag the
   whole project in, Commit.
3. **Enable Pages:** repo **Settings → Pages →** Source = *Deploy from a branch*, Branch = `main`,
   folder = `/ (root)` → **Save**.
4. After ~1 minute your site is live at:
   `https://YOUR_GITHUB_USERNAME.github.io/pods-win-tracker/`

It will show the committed **sample** data (a yellow "Sample data" banner) until the Apps Script
runs for the first time (Part B). The banner disappears automatically once real data is published.

---

## Part B — Wire up the Google Apps Script

### B1. Create a GitHub access token (PAT)
1. GitHub → your avatar → **Settings → Developer settings → Personal access tokens →
   Fine-grained tokens → Generate new token**.
2. **Repository access:** *Only select repositories* → choose `pods-win-tracker`.
3. **Permissions → Repository permissions → Contents → Read and write**.
4. Generate, then **copy the token** (starts with `github_pat_…`). You won't see it again.

### B2. Add the script
1. Open the Google Sheet → **Extensions → Apps Script**.
2. Delete the starter `Code.gs`, paste the contents of `apps-script/Code.gs`.
3. (Optional but recommended) click the **gear / Project Settings → Show "appsscript.json"**, then
   paste `apps-script/appsscript.json` over it.
4. In `CONFIG` at the top of `Code.gs`, set:
   - `GITHUB_OWNER` → your GitHub username
   - `GITHUB_REPO` → `pods-win-tracker` (if you named it differently, match it)
   - confirm `DALLAS_TAB` is exactly your tab name (`Dallas`)

### B3. Store the token as a Script Property
- Apps Script → **Project Settings (gear) → Script Properties → Add script property**
  - **Property:** `GITHUB_TOKEN`
  - **Value:** the `github_pat_…` token from B1 → **Save**

### B4. Authorize + first run
1. In the editor, select the function **`refreshAndPublish`** → **Run**.
2. Approve the permission prompt (it needs: read this spreadsheet, make external requests).
3. When it finishes, check your repo — `data/dallas.json` now has real numbers and a fresh
   `generatedAt`. Refresh the Pages URL; the sample banner is gone.

### B5. Schedule automatic refreshes
- Select **`installTrigger`** → **Run** once. This refreshes + republishes **every 30 minutes**.
  (Change the cadence by editing `everyMinutes(30)` in `installTrigger`.)

---

## If the pods/reps look wrong (parser tuning)

The script auto-locates the pod tables by finding the header labels (`Self Gen/SRA`, etc.). If your
tab is laid out differently and something parses incorrectly:

1. In the Apps Script editor, run **`debugDumpDallas`**, then open **View → Logs**. It prints the
   first 45 rows with column indices.
2. Adjust `CONFIG.METRIC_HEADERS` (the exact header text) to match your tab, then run
   `refreshAndPublish` again.

`data/dallas.json` also embeds the full raw tab grid under `raw` as a safety net, so no data is
ever lost in translation.

---

## Customizing the look (later)

- **Colors / fonts:** edit the `:root { … }` variables at the top of `assets/styles.css`.
- **Logo:** drop an image in `assets/` and swap the `.brand-mark` div in `index.html` for
  `<img src="assets/logo.svg" class="brand-logo" alt="">`.
- **Per-pod brand colors:** add entries to the `BRAND` map at the top of `assets/app.js`, e.g.
  `"DOMINATE.": "#e23b3b"`.
- **Weekly target:** the script reads each pod's `Target` row from the tab; pods without one use
  `CONFIG.DEFAULT_TARGET` (5).

---

## Custom domain (when you're ready)

1. Rename `CNAME.example` → `CNAME`; put your domain (e.g. `pods.yourcompany.com`) on line 1.
2. At your DNS provider add a **CNAME** record: `pods` → `YOUR_GITHUB_USERNAME.github.io`.
   (For an apex/root domain, add the four GitHub Pages `A` records instead — see GitHub docs.)
3. Repo **Settings → Pages → Custom domain** → enter the domain → **Save** → tick **Enforce HTTPS**.

---

## Notes
- The dashboard performs **no score calculations** — every number comes straight from the Dallas
  tab. To change scoring, change the tab.
- Past weeks are not recomputed by the dashboard; whatever the tab holds for a past week is what
  shows. Every refresh is also a Git commit, so you have a full history.
