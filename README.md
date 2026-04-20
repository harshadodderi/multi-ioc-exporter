# multi-source-ioc-exporter

A single-file browser tool to fetch, filter, and export Indicators of Compromise (IOCs) from multiple threat intelligence providers — **AlienVault OTX** (live browser fetch) and **abuse.ch** (MalwareBazaar, URLhaus, ThreatFox via PowerShell) — no backend, no install, no dependencies.

Built via free tier AI tools

---

## What It Does

Two tools in one HTML file, accessible via tabs:

- **AlienVault OTX tab** — connects directly to the OTX API using your personal API key and pulls IOC data from your subscribed pulses. All processing happens in your browser. Results are displayed in a resizable, filterable table and can be exported to CSV or JSON.

- **abuse.ch tab** — generates a ready-to-run PowerShell one-liner that fetches IOCs from MalwareBazaar, URLhaus, and ThreatFox using your abuse.ch Auth-Key. Copy the command, paste into PowerShell, and IOCs are exported to CSV and TXT files automatically. No CORS issues, no browser limitations.

---

## Providers & IOC Counts

| Provider | Method | IOCs Per Fetch | IOC Types |
|---|---|---|---|
| AlienVault OTX | Browser fetch (CORS extension) | Up to 500 pulses (10 pages × 50) | IPv4, Domain, URL, Hash, Email, etc. |
| MalwareBazaar | PowerShell (POST) | 1,000 samples max | SHA256 hashes |
| URLhaus | PowerShell (GET) | All recent URLs (last 3 days) | Malicious URLs |
| ThreatFox | PowerShell (POST) | All IOCs from last 7 days | IPs, Domains, URLs, Hashes |

---

## Features

### AlienVault OTX Tab
- **On-demand pulse pagination** — loads 50 pulses per page (OTX max per call). Pages 1–10 are available as manual buttons; each page is fetched only when you click it and cached for the session
- **Resizable columns** — drag any column border to resize, Excel-style. The table scrolls horizontally so no data is ever hidden
- **Full IOC details per row** — Date, IOC Value, Type, Pulse name, Author, TLP classification, Tags, and a direct clickable link to the source OTX pulse
- **Filter controls** — filter by IOC type (IPv4, domain, file hash, URL, email, etc.), TLP level (WHITE / GREEN / AMBER / RED), or free-text search across IOC value and pulse name
- **Color-coded badges** — IOC types and TLP levels are visually distinguished at a glance
- **Export to CSV** — includes Date, IOC Value, Type, Pulse, Pulse URL, Author, TLP, Tags, Created (full timestamp), and Description
- **Export to JSON** — full structured data for each IOC including pulse URL, suitable for scripting or pipeline ingestion
- **Stats bar** — shows total IOC count, pulse count, distinct IOC types, and distinct authors across all loaded pages

### abuse.ch Tab (PowerShell Generator)
- **One-click command generation** — paste your Auth-Key, select providers, click Generate, copy the command
- **Provider selection** — fetch from all three (MalwareBazaar + URLhaus + ThreatFox) or individually
- **Exports CSV + TXT** — CSV with full metadata (Date, IOC, Type, Provider, Family, Tags), TXT with raw IOC values only (one per line, ready for SIEM blocklists)
- **Timestamped output** — files saved to `.\abuse_ch_iocs\` with `iocs_YYYY-MM-DD_HH-mm-ss` naming
- **No CORS issues** — PowerShell talks directly to the APIs, bypassing all browser restrictions
- **Runs offline** — the HTML page itself is just a command generator; your Auth-Key never leaves your machine

---

## How to Use

### AlienVault OTX Tab

#### 1. Handle CORS

Because this tool runs as a local file in your browser, the OTX API will block requests by default due to browser CORS policy. You need a CORS-unblocking extension — this takes about 60 seconds to set up.

Install one of the following:

| Browser | Extension |
|---|---|
| Chrome | [CORS Unblock](https://chrome.google.com/webstore/detail/cors-unblock/lfhmikememgdcahcdlaciloancbhjino) |
| Chrome | [Allow CORS: Access-Control-Allow-Origin](https://chrome.google.com/webstore/detail/allow-cors-access-control/lhobafahddgcelffkeicbaginigeejlf) |
| Firefox | [CORS Everywhere](https://addons.mozilla.org/en-US/firefox/addon/cors-everywhere/) |

> ⚠ **Security note:** These extensions disable your browser's same-origin protection globally. Enable only while using this tool. Toggle off immediately after.

#### 2. Get Your OTX API Key

1. Go to [https://otx.alienvault.com/api](https://otx.alienvault.com/api)
2. Copy your personal API key from the **"Your API Key"** section

#### 3. Fetch IOCs

1. Open `multi-source-ioc-exporter.html` in Chrome or Firefox
2. Enable your CORS extension
3. Paste your API key into the input field
4. Click **Fetch IOCs** — pulse page 1 loads immediately (up to 50 pulses)

#### 4. Load More Pulse Pages

After the initial load, a **Pulse Pages** navigation bar appears (buttons 1–10). Each button represents the next 50 pulses from OTX. Click any page number to fetch that batch on demand. Already-loaded pages are highlighted and served from cache — no repeat API calls.

#### 5. Filter and Search

Use the controls above the table to narrow results:
- **All Types** dropdown — filter by IOC type (IPv4, DOMAIN, URL, FILEHASH-MD5, etc.)
- **All TLP** dropdown — filter by TLP classification
- **Search box** — searches across IOC value and pulse name in real time

#### 6. Export

- **Export CSV** — downloads all currently filtered IOCs as a `.csv` file
- **Export JSON** — downloads all currently filtered IOCs as a structured `.json` file

> Exports reflect the active filters. To export everything, clear all filters before exporting.

---

### abuse.ch Tab (PowerShell)

#### 1. Get Your Auth-Key

1. Go to [https://auth.abuse.ch/](https://auth.abuse.ch/)
2. Sign in with Google, GitHub, X, or LinkedIn
3. Add a second login method, click **Save profile**
4. Generate your Auth-Key under **Your API Keys**
5. One key works for all three providers (MalwareBazaar, URLhaus, ThreatFox)

#### 2. Generate the Command

1. Open `multi-source-ioc-exporter.html` and click the **abuse.ch** tab
2. Paste your Auth-Key
3. Select providers (All Three, or individually)
4. Click **Generate PowerShell Command**

#### 3. Run It

1. Click **Copy Command**
2. Open PowerShell (Win+X → Windows PowerShell)
3. Right-click to paste, hit Enter
4. IOCs are exported to `.\abuse_ch_iocs\` as CSV + TXT

#### Quick One-Liner (without the HTML tool)

```powershell
$k="YOUR-AUTH-KEY";$h=@{"Auth-Key"=$k};$d=".\abuse_ch_iocs";md $d -Force|Out-Null;$t=Get-Date -F "yyyy-MM-dd_HH-mm-ss";Write-Host "`n=== MalwareBazaar ===" -Fore Cyan;$mb=(irm "https://mb-api.abuse.ch/api/v1/" -Method Post -Headers $h -Body "query=get_recent&selector=time&limit=1000").data;Write-Host "  $($mb.Count) samples" -Fore Green;Write-Host "`n=== URLhaus ===" -Fore Cyan;$uh=(irm "https://urlhaus-api.abuse.ch/v1/urls/recent/" -Method Get -Headers $h).urls;Write-Host "  $($uh.Count) URLs" -Fore Green;Write-Host "`n=== ThreatFox ===" -Fore Cyan;$tf=(irm "https://threatfox-api.abuse.ch/api/v1/" -Method Post -Headers $h -Body '{"query":"get_iocs","days":7}' -ContentType "application/json").data;Write-Host "  $($tf.Count) IOCs" -Fore Green;$all=@();$mb|%{$all+=[pscustomobject]@{Date=$_.first_seen;IOC=$_.sha256_hash;Type="SHA256";Provider="MalwareBazaar";Family=$_.signature;Tags=($_.tags-join", ")}};$uh|%{$all+=[pscustomobject]@{Date=$_.date_added;IOC=$_.url;Type="URL";Provider="URLhaus";Family=$_.threat;Tags=($_.tags-join", ")}};$tf|%{$all+=[pscustomobject]@{Date=$_.first_seen;IOC=$_.ioc;Type=$_.ioc_type;Provider="ThreatFox";Family=$_.malware;Tags=($_.tags-join", ")}};$all|Export-Csv "$d\iocs_$t.csv" -NoType;$all|select -Expand IOC|Out-File "$d\iocs_$t.txt";Write-Host "`n  Total: $($all.Count) IOCs exported to $d\" -Fore Cyan
```

Replace `YOUR-AUTH-KEY` with your actual key.

---

## abuse.ch API Details

| Provider | Method | Endpoint | Auth | Body |
|---|---|---|---|---|
| MalwareBazaar | POST | `https://mb-api.abuse.ch/api/v1/` | Header: `Auth-Key` | `query=get_recent&selector=time&limit=1000` |
| URLhaus | GET | `https://urlhaus-api.abuse.ch/v1/urls/recent/` | Header: `Auth-Key` | none |
| ThreatFox | POST | `https://threatfox-api.abuse.ch/api/v1/` | Header: `Auth-Key` | JSON: `{"query":"get_iocs","days":7}` |

> **Why PowerShell instead of browser fetch?** abuse.ch APIs require a custom `Auth-Key` header, which triggers CORS preflight requests. abuse.ch servers do not respond to preflight — meaning no browser-based approach (CORS extension, proxy, or otherwise) works reliably from a local HTML file. PowerShell talks directly to the APIs without any browser restrictions.

---

## IOC Fields

### AlienVault OTX

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

### abuse.ch (CSV output)

| Field | Description |
|---|---|
| Date | First seen date |
| IOC | The indicator value (hash, URL, IP, domain) |
| Type | IOC type (`SHA256`, `URL`, `ip:port`, `domain`, etc.) |
| Provider | Source provider (`MalwareBazaar`, `URLhaus`, `ThreatFox`) |
| Family | Malware family or threat name |
| Tags | Associated tags |

---

## Architecture

```
multi-source-ioc-exporter.html
│
├── Tab Bar (AlienVault OTX | abuse.ch)
│
├── Tab 1: AlienVault OTX
│   ├── CORS setup guide (collapsible)
│   ├── API key input
│   ├── Progress bar + status line
│   ├── Stats bar (IOC count, pulse count, types, authors)
│   ├── Pulse page nav (1–10, on-demand fetch)
│   ├── Filter controls (type, TLP, search)
│   ├── Resizable IOC table with pagination
│   └── JS: fetchPulsePage() → extractIOCs() → renderTable() → exportCSV/JSON()
│
└── Tab 2: abuse.ch (PowerShell Generator)
    ├── Auth-Key input
    ├── Provider selection (All / MalwareBazaar / URLhaus / ThreatFox)
    ├── Generate button → builds PowerShell one-liner
    ├── Copy-to-clipboard output box
    └── Step-by-step instructions
```

**No frameworks. No npm. No server.** Pure HTML, CSS, and vanilla JavaScript in a single file.

---

## File Naming

Suggested repository name: **`multi-source-ioc-exporter`**

Suggested file name: **`multi-source-ioc-exporter.html`**

---

## Use Cases

- SOC analysts pulling fresh IOCs from subscribed AlienVault threat feeds for daily triage
- Exporting abuse.ch IOCs (hashes, URLs, C2 IPs) for SIEM detection rules in Elastic Security, Splunk, or Microsoft Sentinel
- Building blocklists (IP, domain, hash) for EDR or firewall policies
- Cross-referencing MITRE ATT&CK-tagged pulses during incident response
- Quickly pivoting to source pulse context via the OTX Pulse link during investigation
- Bulk-fetching ThreatFox IOCs with malware family classification for threat hunting

---

## Limitations

- **AlienVault OTX tab** requires a CORS browser extension to function (browser security restriction on local files)
- **abuse.ch tab** requires PowerShell (Windows) — the HTML page generates the command, PowerShell executes it
- AlienVault fetches up to 500 pulses per session (10 pages × 50 pulses). For larger OTX subscriptions, use the OTX Python SDK or DirectConnect API
- abuse.ch APIs require a free Auth-Key from [auth.abuse.ch](https://auth.abuse.ch/)
- No authentication storage — API keys are not saved between sessions
- Internet connection required (both tabs call APIs live)

---

## ⚠ Security Considerations

### CORS Extension (AlienVault Tab Only)

This tab requires a CORS-unblocking browser extension to communicate with the OTX API from a local file. Be aware of the following risks:

#### What the extension does
CORS (Cross-Origin Resource Sharing) is a browser security mechanism that prevents websites and local files from making requests to external APIs without explicit permission. A CORS-unblocking extension bypasses this by injecting `Access-Control-Allow-Origin: *` into every response — globally, across all tabs.

#### Risks while the extension is enabled

- **Any website you visit can make cross-origin requests freely** — malicious sites can silently call APIs, internal network endpoints, or services that rely on CORS as a security boundary
- **Internal network exposure** — if you are on a corporate VPN or internal network, enabled CORS extensions can allow pages to probe and reach internal services (dashboards, admin panels, APIs) that are normally protected by same-origin policy
- **Session token abuse** — authenticated sessions (cookies, tokens) stored in your browser can be leveraged by malicious scripts to call third-party services on your behalf
- **No tab isolation** — the extension does not limit its bypass to just this tool; every open tab is affected simultaneously

#### How to stay safe

| Practice | Why |
|---|---|
| Enable the extension only when actively using this tool | Minimizes your exposure window |
| Close all other tabs before enabling | Prevents other pages from exploiting the open CORS policy |
| Disable immediately after exporting | Do not leave it on during normal browsing |
| Do not use on a work machine connected to internal networks | Risks exposing internal endpoints |
| Use a separate browser profile for this tool | Isolates cookies and sessions from your main profile |

> **Recommended approach:** Use a dedicated Chrome or Firefox profile exclusively for this tool, with the CORS extension installed only in that profile. This fully isolates the risk from your primary browsing session.

### abuse.ch Tab (PowerShell)

- Your Auth-Key is embedded in the generated PowerShell command — do not share the command with others
- The HTML page runs entirely offline for command generation; your key never leaves your machine via the browser
- PowerShell sends the Auth-Key directly to abuse.ch APIs over HTTPS — no third-party proxy or intermediary

---

## License

This is free and unencumbered software released into the public domain.
Anyone is free to copy, modify, publish, use, compile, sell, or distribute this software, for any purpose, commercial or non-commercial, and by any means. No attribution required.
See https://unlicense.org for details.
