# otx-ioc-exporter

A single-file browser tool to fetch, filter, and export Indicators of Compromise (IOCs) from your subscribed [AlienVault OTX](https://otx.alienvault.com) threat intelligence pulses — no backend, no install, no dependencies.

Built via free tier AI tools 

---

## What It Does

Connects directly to the OTX API using your personal API key and pulls IOC data from your subscribed pulses. All processing happens in your browser. Results are displayed in a resizable, filterable table and can be exported to CSV or JSON for SIEM ingestion, blocklist creation, or further analysis.

---

## Features

- **On-demand pulse pagination** — loads 50 pulses per page (OTX max per call). Pages 1–10 are available as manual buttons; each page is fetched only when you click it and cached for the session
- **Resizable columns** — drag any column border to resize, Excel-style. The table scrolls horizontally so no data is ever hidden
- **Full IOC details per row** — Date, IOC Value, Type, Pulse name, Author, TLP classification, Tags, and a direct clickable link to the source OTX pulse
- **Filter controls** — filter by IOC type (IPv4, domain, file hash, URL, email, etc.), TLP level (WHITE / GREEN / AMBER / RED), or free-text search across IOC value and pulse name
- **Color-coded badges** — IOC types and TLP levels are visually distinguished at a glance
- **Export to CSV** — includes Date, IOC Value, Type, Pulse, Pulse URL, Author, TLP, Tags, Created (full timestamp), and Description
- **Export to JSON** — full structured data for each IOC including pulse URL, suitable for scripting or pipeline ingestion
- **Stats bar** — shows total IOC count, pulse count, distinct IOC types, and distinct authors across all loaded pages
- **No install required** — open the `.html` file directly in Chrome or Firefox

---

## How to Use

### 1. Handle CORS

Because this tool runs as a local file in your browser, the OTX API will block requests by default due to browser CORS policy. You need a CORS-unblocking extension — this takes about 60 seconds to set up.

Install one of the following:

| Browser | Extension |
|---|---|
| Chrome | [CORS Unblock](https://chrome.google.com/webstore/detail/cors-unblock/lfhmikememgdcahcdlaciloancbhjino) |
| Chrome | [Allow CORS: Access-Control-Allow-Origin](https://chrome.google.com/webstore/detail/allow-cors-access-control/lhobafahddgcelffkeicbaginigeejlf) |
| Firefox | [CORS Everywhere](https://addons.mozilla.org/en-US/firefox/addon/cors-everywhere/) |

> ⚠ **Security note:** These extensions disable your browser's same-origin protection globally. Enable only while using this tool. Toggle off immediately after.

### 2. Get Your OTX API Key

1. Go to [https://otx.alienvault.com/api](https://otx.alienvault.com/api)
2. Copy your personal API key from the **"Your API Key"** section

### 3. Fetch IOCs

1. Open `otx-ioc-exporter.html` in Chrome or Firefox
2. Enable your CORS extension
3. Paste your API key into the input field
4. Click **Fetch IOCs** — pulse page 1 loads immediately (up to 50 pulses)

### 4. Load More Pulse Pages

After the initial load, a **Pulse Pages** navigation bar appears (buttons 1–10). Each button represents the next 50 pulses from OTX. Click any page number to fetch that batch on demand. Already-loaded pages are highlighted and served from cache — no repeat API calls.

### 5. Filter and Search

Use the controls above the table to narrow results:
- **All Types** dropdown — filter by IOC type (IPv4, DOMAIN, URL, FILEHASH-MD5, etc.)
- **All TLP** dropdown — filter by TLP classification
- **Search box** — searches across IOC value and pulse name in real time

### 6. Export

- **Export CSV** — downloads all currently filtered IOCs as a `.csv` file, ready for Excel, Splunk, Elastic, or any SIEM
- **Export JSON** — downloads all currently filtered IOCs as a structured `.json` file

> Exports reflect the active filters. To export everything, clear all filters before exporting.

---

## IOC Fields

| Field | Description |
|---|---|
| Date | Indicator creation date (`YYYY-MM-DD`) |
| IOC Value | The raw indicator (IP, domain, hash, URL, email, etc.) |
| Type | OTX indicator type (e.g. `IPv4`, `DOMAIN`, `FILEHASH-SHA256`) |
| Pulse | Name of the OTX pulse the IOC belongs to |
| Pulse URL | Direct link to the pulse on OTX |
| Author | OTX username of the pulse author |
| TLP | Traffic Light Protocol level (`WHITE`, `GREEN`, `AMBER`, `RED`) |
| Tags | Threat tags associated with the pulse |
| Created | Full ISO 8601 timestamp of the indicator |
| Description | Pulse description |

---

## Architecture

```
otx-ioc-exporter.html
│
├── UI Layer (HTML + CSS)
│   ├── CORS setup guide (collapsible)
│   ├── API key input
│   ├── Progress bar + status line
│   ├── Stats bar (IOC count, pulse count, types, authors)
│   ├── Pulse page nav (1–10, on-demand fetch)
│   ├── Filter controls (type, TLP, search)
│   └── Resizable IOC table with pagination
│
└── Logic Layer (Vanilla JS)
    ├── fetchPulsePage(key, pageNum)  → OTX API /pulses/subscribed?page=N
    ├── extractIOCs(pulse)            → normalises indicators into flat IOC objects
    ├── renderTable()                 → filters + paginates allIOCs into the table
    ├── renderPulseNav()              → builds the 1–10 pulse page buttons
    ├── initColResize()               → attaches drag-to-resize handlers on <th> elements
    ├── exportCSV()                   → builds and downloads a CSV blob
    └── exportJSON()                  → builds and downloads a JSON blob
```

**No frameworks. No npm. No server.** Pure HTML, CSS, and vanilla JavaScript in a single file.

---

## File Naming

Suggested repository name: **`otx-ioc-exporter`**

Suggested file name: **`otx-ioc-exporter.html`**

---

## Use Cases

- SOC analysts pulling fresh IOCs from subscribed threat feeds for daily triage
- Exporting IOC lists to ingest into Elastic Security, Splunk, or Microsoft Sentinel detection rules
- Building blocklists (IP, domain, hash) for EDR or firewall policies
- Cross-referencing MITRE ATT&CK-tagged pulses during incident response
- Quickly pivoting to source pulse context via the OTX Pulse link during investigation

---

## Limitations

- Requires a CORS browser extension to function (browser security restriction on local files)
- Fetches up to 500 pulses per session (10 pages × 50 pulses). For larger OTX subscriptions, use the OTX Python SDK or DirectConnect API
- No authentication storage — API key is not saved between sessions
- Internet connection required (calls OTX API live)

## ⚠ Security Considerations (CORS Extension)

This tool requires a CORS-unblocking browser extension to communicate with the OTX API from a local file. Be aware of the following risks before use:

### What the extension does
CORS (Cross-Origin Resource Sharing) is a browser security mechanism that prevents websites and local files from making requests to external APIs without explicit permission. A CORS-unblocking extension bypasses this by injecting `Access-Control-Allow-Origin: *` into every response — globally, across all tabs.

### Risks while the extension is enabled

- **Any website you visit can make cross-origin requests freely** — malicious sites can silently call APIs, internal network endpoints, or services that rely on CORS as a security boundary
- **Internal network exposure** — if you are on a corporate VPN or internal network, enabled CORS extensions can allow pages to probe and reach internal services (dashboards, admin panels, APIs) that are normally protected by same-origin policy
- **Session token abuse** — authenticated sessions (cookies, tokens) stored in your browser can be leveraged by malicious scripts to call third-party services on your behalf
- **No tab isolation** — the extension does not limit its bypass to just this tool; every open tab is affected simultaneously

### How to stay safe

| Practice | Why |
|---|---|
| Enable the extension only when actively using this tool | Minimizes your exposure window |
| Close all other tabs before enabling | Prevents other pages from exploiting the open CORS policy |
| Disable immediately after exporting | Do not leave it on during normal browsing |
| Do not use on a work machine connected to internal networks | Risks exposing internal endpoints |
| Use a separate browser profile for this tool | Isolates cookies and sessions from your main profile |

> **Recommended approach:** Use a dedicated Chrome or Firefox profile exclusively for this tool, with the CORS extension installed only in that profile. This fully isolates the risk from your primary browsing session.
---

## License

This is free and unencumbered software released into the public domain.
Anyone is free to copy, modify, publish, use, compile, sell, or distribute this software, for any purpose, commercial or non-commercial, and by any means. No attribution required.
See https://unlicense.org for details.
