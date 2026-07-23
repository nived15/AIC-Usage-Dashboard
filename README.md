# AI Credit Usage Dashboard

Small local web app with two panels:

1. **AI Credit Usage** — pulls GitHub Enterprise AI Credit usage report exports and visualizes them in a dashboard with charts and a searchable/sortable table.
2. **Copilot Seat Management & Activity** — fetches Copilot seat assignments for an organization and evaluates each seat's activity status.

---

## REST API Endpoints Used

### AI Credit Usage (API version `2026-03-10`)

- `POST /enterprises/{enterprise}/settings/billing/reports` — [Create a usage report export](https://docs.github.com/en/enterprise-cloud@latest/rest/billing/usage-reports?apiVersion=2026-03-10#create-a-usage-report-export)
- `GET /enterprises/{enterprise}/settings/billing/reports/{report_id}` — [Get a usage report export](https://docs.github.com/en/enterprise-cloud@latest/rest/billing/usage-reports?apiVersion=2026-03-10#get-a-usage-report-export)

### Copilot Seat Management

- `GET /orgs/{org}/copilot/billing/seats` — [List all Copilot seat assignments for an organization](https://docs.github.com/en/rest/copilot/copilot-user-management#list-all-copilot-seat-assignments-for-an-organization)

---

## Requirements

- Node.js **18+** (uses native `fetch` and `crypto.randomUUID`)
- For **AI Credit Usage**: a GitHub PAT (classic) with the **`manage_billing:enterprise`** scope, owned by an enterprise admin or billing manager
- For **Copilot Seat Management**: a GitHub PAT (classic) with the **`manage_billing:copilot`** scope (or `read:org` for read-only access), owned by an org owner or billing manager

---

## Install & Run

```bash
npm install
npm start
```

Then open <http://localhost:3000>.

---

## AI Credit Usage

Fill in:

- **Enterprise slug** (the URL slug, not the display name)
- **PAT** with `manage_billing:enterprise` scope (kept in memory only, forwarded to `api.github.com`)
- **Start date** (required, from May 1 2026) and optional **End date**

Click **Generate Dashboard**. The server will:

1. Create the AI credit usage report export.
2. Poll `GET .../reports/{id}` every few seconds until `status = completed`.
3. Download the CSV(s) from `download_urls`, parse them, and return JSON.

While waiting, you'll get playful progress updates (report generation can take several minutes for large enterprises).

### AI Credit Features

- 📊 Timeline chart (usage over time)
- 🏷️ Top models / SKUs — click a bar to drill down
- 🏢 Usage by organization (doughnut)
- 👤 Top users
- 🔎 Searchable, sortable, paginated raw records table (DataTables)
- 📥 Download combined CSV of all fetched records
- 🎭 Fun, engaging progress messages while polling

The dashboard auto-detects common column names (`date`, `quantity`, `gross_amount`, `net_amount`, `sku`/`product`/`model`, `organization`, `user`/`username`/`actor`). If a field isn't present in the report, that chart shows a friendly "not detected" message but the rest of the dashboard still works.

---

## Copilot Seat Management & Activity

Fill in:

- **Organization** (org login, not display name)
- **PAT** with `manage_billing:copilot` scope
- **Activity window (days)** — number of days to look back when classifying a seat as Active or Inactive (default: 30)

Click **Fetch Seats**. The server will:

1. Page through `GET /orgs/{org}/copilot/billing/seats` (100 seats per page) until all seats are collected.
2. Classify each seat as **Active** (last activity within the window) or **Inactive**.
3. Return summary counts and per-seat details.

### Seat Management Features

- 🪑 Summary stats: total seats, active seats, inactive seats, pending cancellation
- 🔎 Searchable, sortable, paginated seat table showing user, team, activity status, last activity timestamp, last editor, pending cancellation date, and seat creation date
- 📥 Download seat data as CSV
- ⏱️ Configurable activity window — adjust what "active" means for your team

### Troubleshooting

| Error | Likely cause |
| --- | --- |
| `404 Not Found` | The org has no Copilot subscription, the org name is wrong, or the PAT lacks `manage_billing:copilot` scope |
| `403 Forbidden` | The PAT owner is not an org owner or billing manager |
| `401 Unauthorized` | The PAT is invalid or expired |

---

## Security Notes

- PATs are sent from the browser to the **local** Node server only, forwarded as `Authorization: Bearer …` to GitHub, and **never persisted** to disk.
- Run this locally. Do not expose port 3000 publicly without adding authentication.
