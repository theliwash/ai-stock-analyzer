# 📈 AI Stock Analyzer — n8n Workflow

An automated AI-powered stock analysis workflow built in **n8n** that monitors your portfolio in Google Sheets, analyzes each holding using Claude (Anthropic), and delivers a personalized **Buy / Hold / Sell** report straight to your Gmail — triggered automatically whenever you update your watchlist.

![Workflow Screenshot](./assets/workflow-screenshot.png)

---

## 🧠 How It Works

The workflow is a 3-node pipeline:

```
Google Sheets Trigger (anyUpdate)
            ↓
        AI Agent
   [Anthropic Chat Model]
            ↓
   Gmail — Send a Message
```

1. **Google Sheets Trigger** — Fires on `anyUpdate`, meaning any time you add, edit, or replace a stock ticker in your watchlist, the workflow kicks off automatically
2. **AI Agent (Claude)** — Receives the row data from the sheet, analyzes the stock fundamentals, and generates a structured recommendation report
3. **Gmail — Send a Message** — Delivers the report to your inbox instantly

The **Anthropic Chat Model** (claude-sonnet-4-6) is wired in as the Chat Model sub-node powering the AI Agent.

---

## 📊 Google Sheet Setup

Create a Google Sheet with the following columns:

| Stock Ticker | Stock Price | Market Cap | P/E Ratio | EPS | 52-High | 52-Low | Volume | Daily Avg Volume |
|---|---|---|---|---|---|---|---|---|

Use these `GOOGLEFINANCE` formulas from row 2 onwards (where `A2` is your ticker cell):

```
Stock Price:       =GOOGLEFINANCE(A2, "price")
Market Cap:        =GOOGLEFINANCE(A2, "marketcap")
P/E Ratio:         =GOOGLEFINANCE(A2, "pe")
EPS:               =GOOGLEFINANCE(A2, "eps")
52-Week High:      =GOOGLEFINANCE(A2, "high52")
52-Week Low:       =GOOGLEFINANCE(A2, "low52")
Volume:            =GOOGLEFINANCE(A2, "volume")
Daily Avg Volume:  =GOOGLEFINANCE(A2, "volumeavg")
```

### Ticker Format by Market

| Market | Format | Example |
|---|---|---|
| 🇺🇸 US | Plain symbol | `AAPL` |
| 🇬🇧 UK | `LON:` prefix | `LON:BARC` |
| 🇮🇳 India (NSE) | `NSE:` prefix | `NSE:RELIANCE` |
| 🇮🇳 India (BSE) | `BOM:` prefix | `BOM:500325` |

> ⚠️ UK stock prices via GOOGLEFINANCE are quoted in **pence**, not pounds. Divide by 100 for the £ value.

---

## 🛠️ Tech Stack

| Tool | Role |
|---|---|
| [n8n](https://n8n.io) | Workflow orchestration |
| Google Sheets | Watchlist & data source |
| `GOOGLEFINANCE()` | Live market data (15–20 min delay) |
| Claude — `claude-sonnet-4-6` | AI analysis & recommendations |
| Gmail | Report delivery |

---

## 🚀 Setup Instructions

### Prerequisites
- An [n8n](https://n8n.io) account (cloud or self-hosted)
- A Google account (Sheets + Gmail access)
- An [Anthropic API key](https://console.anthropic.com/settings/keys)

---

### Step 1 — Set Up Your Google Sheet

1. Create a new Google Sheet
2. Add the column headers listed above
3. Enter your stock tickers in column A
4. Add the `GOOGLEFINANCE` formulas to columns B–I for each row

---

### Step 2 — Import the Workflow into n8n

1. Download [`workflow/stock-analyzer.json`](./workflow/stock-analyzer.json) from this repo
2. In n8n, go to **Workflows → Import from File**
3. Select the downloaded `.json` file

---

### Step 3 — Configure Credentials

Update the following nodes inside n8n with your own credentials:

| Node | What to Configure |
|---|---|
| **Google Sheets Trigger** | Connect via Google OAuth |
| **Anthropic Chat Model** | Add your Anthropic API key |
| **Gmail — Send a Message** | Connect via Google OAuth, set recipient email |

---

### Step 4 — Point the Trigger to Your Sheet

In the **Google Sheets Trigger** node:
- Select your spreadsheet
- Select the correct sheet tab
- Event is set to `anyUpdate` — no changes needed

---

### Step 5 — Activate

Toggle the workflow to **Active** in n8n. From this point, every time you update a ticker in your sheet, the agent fires and sends you an analysis.

---

## 📬 Sample Report Output

```
📊 Stock Analysis Report — AAPL
Generated: 24 May 2026

Recommendation: ✅ HOLD

──────────────────────────────
Metric          Value
──────────────────────────────
Price           $213.40
Market Cap      $3.27T
P/E Ratio       32.4
EPS             $6.58
52-Week High    $237.23
52-Week Low     $164.08
Volume          58.2M
Avg Volume      61.4M
──────────────────────────────

Analysis:
Apple is trading in the upper half of its 52-week range, reflecting
sustained bullish sentiment. The P/E of 32.4 is elevated versus the
broader market but is supported by strong EPS of $6.58. Volume is
slightly below the daily average, suggesting no major institutional
activity at this time. Fundamentals remain solid. Recommendation is
to HOLD and reassess after the next earnings report.
```

---

## ⚠️ Known Limitations

- `GOOGLEFINANCE` data carries a ~15–20 minute delay — this is not real-time data
- P/E and EPS may return `#N/A` for some non-US stocks; the agent will note this in its output
- The `anyUpdate` trigger fires on every cell edit — if you're making bulk changes, expect multiple emails
- GOOGLEFINANCE formulas only refresh when the sheet is open; if the sheet is closed at trigger time, data may be stale

---

## 💡 Possible Extensions

- Add a **Memory** sub-node to the AI Agent so it tracks how your recommendations change over time
- Connect a **Tool** sub-node (e.g. web search) so the agent can pull recent news alongside the sheet data
- Add a **Slack** or **Telegram** node alongside Gmail for instant mobile alerts
- Build a daily scheduled trigger in addition to the sheet-update trigger

---

## 📄 License

MIT License — free to use, modify, and distribute.
