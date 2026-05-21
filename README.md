# Shipper Research

A single-lookup AI research tool for classifying shipper companies in freight-forwarding
spreadsheets. Built to surface info about the lesser-known Chinese trading and
manufacturing companies that ordinary search engines tend to under-cover.

Companion tool to the commodity classifier — the research log here exports in the
format that classifier's **Company Directory** can import.

## What it does

Enter a company name. The tool:

1. Searches the web (via [Tavily](https://tavily.com)) with queries tuned for company research
2. Feeds the search results to [Qwen 3 32B](https://groq.com) via Groq for structured extraction
3. Returns a result card with:
   - **Identity**: canonical name, country, headquarters, year established, status
   - **Business**: one-sentence description, main products, business type (manufacturer vs trader vs forwarder), industry sector
   - **Classification**: suggested commodity group + commodity label (from your existing schema), with a confidence rating and reason
   - **Trade signals** (when visible): top customers, top HS codes, top destination markets, approximate size
   - **Risk**: sanctions flags, suspicious patterns
   - **Sources**: every URL the synthesis was based on, for verification

All fields are editable inline before saving. Saved entries accumulate in a local
research log that you can export as JSON for the commodity classifier.

## Setup (5 minutes)

You need two free API keys:

### 1. Tavily

- Go to <https://app.tavily.com/>, sign in (Google sign-in works)
- Copy the API key from the dashboard (`tvly-...`)
- Free tier: 1,000 queries/month

### 2. Groq

- Go to <https://console.groq.com/keys>, sign in
- Click "Create API Key"
- Copy the key (`gsk_...`)
- Free tier: generous rate limits on Qwen 3 32B

### 3. Open the tool

- If you've deployed it to GitHub Pages, just visit the URL
- If running locally, open `index.html` in a browser
- Click ⚙️ Settings, paste both keys, click Save
- Keys are stored in your browser's localStorage — they never leave your machine

## How to use it

### Researching a single company

1. Type the company name in the search box, exactly as it appears in your data
2. Click Research (or press Enter)
3. Wait 10-30 seconds while the tool searches and synthesises
4. Review the result card:
   - Edit any field if you disagree (the dropdowns for commodity group / label can be
     changed; the canonical name is editable too)
   - Click **Save to log** to add it to your research history
   - Click **Export this** to download just this entry as JSON
   - Click **Discard** to throw it away

### The research log

The sidebar on the right shows everything you've saved. Click any entry to revisit.
Two actions at the top:

- **Export** — downloads the entire log as a JSON file in the format that the
  commodity classifier's Company Directory can import (it's a `{COMPANY_DIRECTORY_LIST: [...]}` shape).
- **Clear** — wipes the log (with a confirmation prompt).

### Using results in the commodity classifier

After exporting your research log:

1. Open the commodity classifier
2. Go to the Rules tab → Import rules
3. Pick the exported JSON file
4. The shipper → group mappings populate the Company Directory

Now those companies auto-classify on future runs.

## Deployment

### Local

Just open `index.html` in a browser. CORS is handled fine; the API calls go directly
to Tavily and Groq from your browser.

### Hosted (GitHub Pages)

1. Create a new GitHub repository (e.g., `shipper-research`)
2. Upload `index.html` and `README.md`
3. Settings → Pages → Source: Deploy from a branch → main / root
4. Visit the URL GitHub gives you
5. Each user enters their own API keys in Settings on first use

No backend, no secrets in the repo. Each user's keys live in their own browser only.

## Limitations and honest expectations

- **For well-known companies (Hytera, Hikvision, Xiaomi, Ralph Lauren)**: high
  confidence, accurate results expected.
- **For mid-tier Chinese companies with English websites**: usually good,
  occasional name confusion (e.g., "B&B Group Limited" — there are several).
- **For shell-like generic trading names**: the tool will often correctly say "low
  confidence, couldn't determine specific product line" rather than guess. This is
  the right behaviour — guessing wrong is worse than admitting uncertainty.
- **Real-time shipment volumes**: not available. Paid trade-data subscriptions
  (Panjiva, ImportGenius) have this; this free tool relies on what's visible in
  search result snippets.

The tool is designed to surface information that exists; it won't conjure information
that doesn't. Some companies are genuinely invisible in public sources, and the tool
should be honest about that rather than fabricate.

## Cost

- **Tavily**: free tier covers ~33 lookups/day or 1,000/month
- **Groq**: free tier covers 1,000+ requests/day with Qwen 3 32B
- **Hosting**: free (GitHub Pages or local file)
- **Total**: $0/month at typical use

If you exceed the Tavily free tier, their paid tiers start around $30/month.

## File structure

```
shipper-research/
├── README.md       this file
└── index.html      the whole app (single file, no build step)
```

## Customisation

Open `index.html` and look for the `COMMODITY_GROUPS` and `COMMODITY_LABELS` constants
near the top of the script section. These are the controlled vocabulary the AI is
forced to pick from. Edit them to match your team's classification scheme exactly
if it differs from the defaults.

The `SYSTEM_PROMPT` constant just below is the instruction set for Qwen. If results
aren't great, tweaking this prompt is the most powerful way to improve them —
no rebuild required, just refresh the page after editing.

## License

Use it however helps your team.
