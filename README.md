# Multi-Source IOC Exporter

A single-file browser tool to fetch, filter, and export Indicators of Compromise (IOCs) from multiple threat intelligence providers — **AlienVault OTX** (live browser fetch) and **abuse.ch** (MalwareBazaar, URLhaus, ThreatFox via PowerShell) — no backend, no install, no dependencies.

Built via free tier AI tools

---

## Quick Links

**Open the tools directly:**

- 🔗 **[Multi-Source IOC Exporter (OTX and abuse.ch)](https://harshadodderi.github.io/multi-ioc-exporter/Multi-IOC-Exporter.html)** — generate PowerShell commands for MalwareBazaar, URLhaus, ThreatFox

- 🔗 **[AlienVault OTX IOC Exporter](https://harshadodderi.github.io/multi-ioc-exporter/OTX-IOC-Exporter.html)** — fetch IOCs from your subscribed pulses

Or run locally: download `multi-source-ioc-exporter.html` from this repo and open in your browser.

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
- **CORS auto-detection** — on page load, the tool pings the OTX API to check if CORS is working from the current origin. If it is (e.g., GitHub Pages, localhost), the CORS extension warning is replaced with a green "✓ CORS is working" banner. If not (e.g., `file://`), the full extension setup guide is shown
- **Pre-fetch security modal** — before the first fetch in each session, a checklist modal prompts the user to close other tabs, verify the CORS extension, and use a dedicated browser profile. Shows once per session, then does not reappear for subsequent fetches
- **Post-export security modal** — after the first CSV or JSON export, a modal reminds the user to disable the CORS extension immediately, close the tab, and verify the downloaded file. Shows once per session

### abuse.ch Tab (PowerShell Generator)

- **One-click command generation** — paste your Auth-Key, select providers, click Generate, copy the command
- **Provider selection** — fetch from all three (MalwareBazaar + URLhaus + ThreatFox) or individually
- **Exports CSV + TXT** — CSV with full metadata (Date, IOC, Type, Provider, Family, Tags), TXT with raw IOC values only (one per line, ready for SIEM blocklists)
- **Timestamped output** — files saved to `.\abuse_ch_iocs\` with `iocs_YYYY-MM-DD_HH-mm-ss` naming
- **No CORS issues** — PowerShell talks directly to the APIs, bypassing all browser restrictions
- **Runs offline** — the HTML page itself is just a command generator; your Auth-Key never leaves your machine
- **Auto history cleanup** — every generated command ends with `Clear-History` and `[Microsoft.PowerShell.PSConsoleReadLine]::ClearHistory()` so your Auth-Key does not persist in PowerShell terminal history or the PSReadLine history file
- **Security reminder on copy** — after clicking "Copy Command", a warning appears reminding users about the PSReadLine history file location (`%APPDATA%\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`) for shared environments like jump boxes

---

## ⚠ Security & Threat Model — Read Before Using

**This tool trades convenience for attack surface.** You are making a conscious security decision by using it. Understand the risks before proceeding.

### Why This Trade-Off?

**Benefit:** Instant, zero-friction IOC fetching. No install, no backend, no proxy.

**Cost:** Your API keys live in browser memory. A compromised page, malicious CORS extension, or intercepted font CDN can steal them.

---

### Attack Surface & Risk Assessment

#### 1. API Keys in Browser Memory (HIGH RISK)

You paste your **OTX API key** or **abuse.ch Auth-Key** directly into the webpage. Malicious JavaScript can read it from the DOM and exfiltrate it in seconds.

**Attack vectors:**
- **Compromised font CDN** — fonts loaded from `googleapis.com`. If intercepted or compromised, attacker injects script that steals your key
- **GitHub Pages compromise** — unlikely but possible; attacker gains code execution, injects payload
- **Malicious CORS extension** — you install an extension to unblock CORS. If the extension is compromised downstream (supply chain attack) or is intentionally trojanized, it steals keys globally across all pages

**What an attacker can do with your key:**
- **OTX:** Enumerate all your subscribed pulses and private threat intel; pollute your API quota; perform reconnaissance on your threat landscape
- **abuse.ch:** Silently fetch IOCs and exfiltrate them; impersonate your organization's threat intelligence operations

**Mitigation:**
- ✅ Use **read-only or throwaway keys only** — never your production/primary credentials
- ✅ Generate a dedicated OTX key just for this tool; consider deleting it after use
- ✅ For abuse.ch: use a separate Auth-Key per team/tool so you can revoke individually if compromised
- ✅ Never reuse keys across tools or environments

---

#### 2. CORS Extensions Weaken Global Browser Security (MEDIUM RISK)

The AlienVault tab may require you to install a CORS-unblocking extension. These extensions inject `Access-Control-Allow-Origin: *` into **every HTTP response, globally, across all tabs and websites.**

**What this means:**
- While the extension is enabled, any website you visit can silently make requests to:
  - External APIs (GitHub, AWS, GCP, Azure endpoints, etc.)
  - Internal network services (`http://192.168.1.1/admin`, `http://jenkins:8080/`, etc.)
  - Third-party services relying on CORS as a security boundary

**Real attack scenarios:**
- You enable the CORS extension, then check Slack or Gmail
- Malicious actor posts a link in Slack that contains `<img src="http://internal-grafana:3000/api/datasources">`
- Browser fetches that endpoint (normally blocked by CORS) and exfiltrates your internal monitoring stack
- Similarly, attacker can enumerate internal services, call authenticated endpoints using your session cookies, or trigger actions on your behalf

**On corporate networks:** If you're on a VPN or internal network and have the CORS extension enabled, attacker can probe `*.company.local`, `10.0.0.0/8`, or dedicated internal IPs to map your infrastructure.

**Mitigation:**
- ✅ Enable the extension **only while actively using this tool**
- ✅ Close all other browser tabs before enabling
- ✅ Disable immediately after exporting IOCs (don't leave it on)
- ✅ **Recommended:** Use a **separate, dedicated browser profile** exclusively for this tool with the extension pre-installed. This fully isolates the risk from your primary browsing
- ❌ Do not use on machines connected to corporate networks or internal infrastructure
- ❌ Do not use on shared jump boxes or multi-user systems

---

#### 3. External Resources (MEDIUM RISK)

Fonts are loaded from `https://fonts.googleapis.com`. If this CDN is compromised, experiences a supply-chain attack, or your network is MITM'd, attacker injects malicious script.

**Who can intercept:**
- ISP or network provider (corporate proxy, public WiFi)
- Nation-state or advanced attacker with BGP hijacking / DNS poisoning capability
- Compromised CDN infrastructure

**Mitigation:**
- ✅ **Run locally instead of on GitHub Pages** — eliminates external font dependency:
  ```bash
  python -m http.server 8000
  # Open http://localhost:8000/multi-source-ioc-exporter.html
  ```
- ✅ Self-host fonts (download `.woff2` files, embed in repo)
- ✅ Set Content Security Policy (CSP) headers if hosting on your own domain (not possible on GitHub Pages)

---

#### 4. Public Source Code (LOW RISK)

The entire tool is open-source. Anyone can audit the HTML/JS, find bugs, or design flaws. No obfuscation, no secrets embedded in code.

**Mitigation:**
- Audit the code yourself if you're risk-averse
- Fork and self-host if you don't trust GitHub's infrastructure
- Trust is transitive: if you trust the code is safe, public hosting is fine

---

#### 5. API Key Persistence in PowerShell (abuse.ch tab only) (LOW RISK)

Your Auth-Key is embedded in the generated PowerShell command. The command auto-clears history, but cleanup can fail silently on restricted environments.

**What the cleanup covers:**
- ✅ In-session terminal history (`Get-History`)
- ✅ PSReadLine history file (`%APPDATA%\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`)

**What it does NOT cover:**
- ❌ Event Log transcripts (if PS Logging is enabled)
- ❌ Reverse-shell logs (if you use RDP or jump box)
- ❌ Manual history file edits (on shared machines, admin can restore)
- ❌ If `ClearHistory()` fails silently due to Group Policy or execution policy restrictions

**Mitigation:**
- ✅ Use **personal machines only**, not shared infrastructure
- ✅ If on a shared machine: run the command once, then **manually verify and delete**:
  ```
  %APPDATA%\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
  ```
- ✅ Prefer running locally over RDP or jump boxes
- ✅ If key is compromised, regenerate immediately at [auth.abuse.ch](https://auth.abuse.ch/) — old key is invalidated instantly

---

### Verdict: Who Should Use This?

| Scenario | Risk | Recommendation |
|---|---|---|
| **Personal SOC lab, weekend research, homelab detection engineering** | Low | ✅ **Safe.** Use throwaway keys. Close other tabs. Disable extension after export. |
| **Pull IOCs for SIEM rule development (personal laptop)** | Low | ✅ **Safe.** Use read-only keys. Local hosting (run on localhost) is safer than GitHub Pages. |
| **Shared jump box / team infrastructure** | High | ❌ **Avoid.** Keys will persist in shared history. Use a backend proxy or managed SIEM instead. |
| **Corporate network / VPN** | High | ❌ **Avoid.** CORS extension can probe internal endpoints. Use backend proxy or official threat intel platform. |
| **Production / primary API credentials** | Critical | ❌ **Never.** Use read-only or throwaway keys only. |
| **Untrusted network (public WiFi, shared office)** | Medium | ❌ **Avoid.** MITM can intercept fonts, inject script, steal keys. Use VPN + local hosting if you must. |

---

### Alternatives (Lower Risk)

If the threat model above concerns you, consider these alternatives:

#### 1. **Run Locally (Recommended for security-conscious users)**

```bash
# Download the HTML file, then serve it locally:
cd /path/to/multi-source-ioc-exporter
python -m http.server 8000

# Open http://localhost:8000/multi-source-ioc-exporter.html
```

**Why safer:**
- Eliminates GitHub Pages infrastructure risk
- No external font fetches (localhost = no CDN)
- You control the server (localhost = no MITM)
- Can set CSP headers with a proper web server (Flask, Node, etc.)

#### 2. **Backend Proxy (Production approach)**

Wrap the API calls in a backend service that holds the key:
```
Browser → Your Backend Service → OTX/abuse.ch APIs
                ↑
          (key stored here, not in browser)
```

Your backend authenticates to OTX/abuse.ch; the browser never sees the key. Build this with Flask, Node, Django, or Go.

#### 3. **Use the Official CLIs**

- **OTX:** `pip install python-otx && python -c "from OTXv2 import OTXv2; ..."`
- **abuse.ch:** `curl` / `wget` one-liners or Python `requests` library
- **Other:** Elasticsearch, Splunk, or cloud-native SIEM connectors (AWS Security Hub, Azure Sentinel, Google Chronicle)

#### 4. **Managed Threat Intelligence Platforms**

If you're running a production SOC, invest in:
- **Splunk SOAR / Phantom** with threat intel integrations
- **Elastic Security** (built-in abuse.ch connector)
- **Microsoft Sentinel** (built-in threat intel sources)
- **Google Chronicle** (managed threat intel pipeline)

---

### Acknowledgment

This tool prioritizes **speed and accessibility** over fortress-level security. If you need a production-grade threat intel pipeline with audit logging, key management, and compliance controls, invest in a managed platform or build a backend service.

For personal research, weekend labs, or low-stakes IOC enrichment: this tool is effective and reasonably safe if you follow the mitigations above.

---

## How to Use

### AlienVault OTX Tab

#### 1. Handle CORS

The tool **auto-detects** whether CORS is working on page load by pinging the OTX API. If you're hosting on GitHub Pages or localhost and OTX allows the origin, the CORS extension warning is automatically replaced with a green "✓ CORS is working" banner — no extension needed.

If CORS is blocked (e.g., opening as `file://`), the full extension setup guide is shown. Install one of the following:

| Browser | Extension |
|---|---|
| Chrome | [CORS Unblock](https://chrome.google.com/webstore/detail/cors-unblock/lfhmikememgdcahcdlaciloancbhjino) |
| Chrome | [Allow CORS: Access-Control-Allow-Origin](https://chrome.google.com/webstore/detail/allow-cors-access-control/lhobafahddgcelffkeicbaginigeejlf) |
| Firefox | [CORS Everywhere](https://addons.mozilla.org/en-US/firefox/addon/cors-everywhere/) |

**After installing:**
- Click the extension icon in your browser toolbar
- Toggle it **ON**
- Return to this page and click **Fetch IOCs**

#### 2. Get Your OTX API Key

1. Go to [https://otx.alienvault.com/api](https://otx.alienvault.com/api)
2. Copy your personal API key from the **"Your API Key"** section

#### 3. Fetch IOCs

1. Open `multi-source-ioc-exporter.html` in Chrome or Firefox
2. If you need the extension, enable it (see Step 1 above)
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

#### 7. Disable the Extension (Important!)

After you're done fetching and exporting:
1. Click the CORS extension icon in your toolbar
2. Toggle it **OFF**
3. Close this tab or browser window

---

### abuse.ch Tab (PowerShell)

#### 1. Get Your Auth-Key

1. Go to [https://auth.abuse.ch/](https://auth.abuse.ch/)
2. Sign in with Google, GitHub, X, or LinkedIn
3. Add a second login method and click **Save profile**
4. Generate your Auth-Key under **Your API Keys**
5. One key works for all three providers (MalwareBazaar, URLhaus, ThreatFox)

#### 2. Generate the Command

1. Open `multi-source-ioc-exporter.html` and click the **abuse.ch** tab
2. Paste your Auth-Key
3. Select providers:
   - **All Three** — MalwareBazaar + URLhaus + ThreatFox in one command
   - **MalwareBazaar Only** — SHA256 hashes only
   - **URLhaus Only** — Recent malicious URLs
   - **ThreatFox Only** — Last 7 days of IOCs
4. Click **Generate PowerShell Command**

#### 3. Run It

1. Click **Copy Command**
2. Open PowerShell (Win+X → Windows PowerShell or Windows Terminal)
3. Right-click to paste, hit **Enter**
4. Wait for completion — IOCs are exported to `.\abuse_ch_iocs\` as CSV + TXT

**Output files:**
- `iocs_YYYY-MM-DD_HH-mm-ss.csv` — full metadata (Date, IOC, Type, Provider, Family, Tags)
- `iocs_YYYY-MM-DD_HH-mm-ss.txt` — raw IOC values only (one per line, ready for SIEM blocklists)

#### 4. Manual Verification (on shared machines)

After the command completes, if you ran it on a shared machine:

1. Open File Explorer and navigate to: `%APPDATA%\Microsoft\Windows\PowerShell\PSReadLine\`
2. Right-click `ConsoleHost_history.txt` → **Delete** or **Edit** to remove any visible Auth-Key

The auto-cleanup command should handle this, but manual verification is recommended on jump boxes or multi-user systems.

---

### Quick One-Liner (abuse.ch, without the HTML tool)

If you prefer to run the command directly without the HTML generator:

```powershell
$k="YOUR-AUTH-KEY";$h=@{"Auth-Key"=$k};$d=".\abuse_ch_iocs";md $d -Force|Out-Null;$t=Get-Date -F "yyyy-MM-dd_HH-mm-ss";Write-Host "`n=== MalwareBazaar ===" -Fore Cyan;$mb=(irm "https://mb-api.abuse.ch/api/v1/" -Method Post -Headers $h -Body "query=get_recent&selector=time&limit=1000").data;Write-Host "  $($mb.Count) samples" -Fore Green;Write-Host "`n=== URLhaus ===" -Fore Cyan;$uh=(irm "https://urlhaus-api.abuse.ch/v1/urls/recent/" -Method Get -Headers $h).urls;Write-Host "  $($uh.Count) URLs" -Fore Green;Write-Host "`n=== ThreatFox ===" -Fore Cyan;$tf=(irm "https://threatfox-api.abuse.ch/api/v1/" -Method Post -Headers $h -Body '{"query":"get_iocs","days":7}' -ContentType "application/json").data;Write-Host "  $($tf.Count) IOCs" -Fore Green;$all=@();$mb|%{$all+=[pscustomobject]@{Date=$_.first_seen;IOC=$_.sha256_hash;Type="SHA256";Provider="MalwareBazaar";Family=$_.signature;Tags=($_.tags-join", ")}};$uh|%{$all+=[pscustomobject]@{Date=$_.date_added;IOC=$_.url;Type="URL";Provider="URLhaus";Family=$_.threat;Tags=($_.tags-join", ")}};$tf|%{$all+=[pscustomobject]@{Date=$_.first_seen;IOC=$_.ioc;Type=$_.ioc_type;Provider="ThreatFox";Family=$_.malware;Tags=($_.tags-join", ")}};$all|Export-Csv "$d\iocs_$t.csv" -NoType;$all|select -Expand IOC|Out-File "$d\iocs_$t.txt";$fullPath=(Resolve-Path $d).FullName;Write-Host "`n  📁 Files saved to: $fullPath" -Fore Green;Write-Host "  └─ iocs_$t.csv" -Fore Gray;Write-Host "  └─ iocs_$t.txt" -Fore Gray;Write-Host "`n  Total: $($all.Count) IOCs exported" -Fore Cyan;Clear-History;try{[Microsoft.PowerShell.PSConsoleReadLine]::ClearHistory()}catch{};Write-Host "`n  [SECURITY] PowerShell history cleared — Auth-Key removed from terminal." -Fore DarkGray
```

Replace `YOUR-AUTH-KEY` with your actual key.

---

## abuse.ch API Details

| Provider | Method | Endpoint | Auth | Body/Query |
|---|---|---|---|---|
| MalwareBazaar | POST | `https://mb-api.abuse.ch/api/v1/` | Header: `Auth-Key` | `query=get_recent&selector=time&limit=1000` |
| URLhaus | GET | `https://urlhaus-api.abuse.ch/v1/urls/recent/` | Header: `Auth-Key` | none |
| ThreatFox | POST | `https://threatfox-api.abuse.ch/api/v1/` | Header: `Auth-Key` | JSON: `{"query":"get_iocs","days":7}` |

**Why PowerShell instead of browser fetch?**

abuse.ch APIs require a custom `Auth-Key` header, which triggers CORS preflight requests. abuse.ch servers do not respond to preflight — meaning no browser-based approach (CORS extension, proxy, or otherwise) works reliably from a local HTML file. PowerShell talks directly to the APIs without any browser restrictions.

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
│   ├── CORS auto-detection (pings OTX on load → green/amber banner swap)
│   ├── CORS setup guide (collapsible, shown only if auto-detect fails)
│   ├── API key input
│   ├── Pre-fetch security modal (once per session)
│   ├── Progress bar + status line
│   ├── Stats bar (IOC count, pulse count, types, authors)
│   ├── Pulse page nav (1–10, on-demand fetch)
│   ├── Filter controls (type, TLP, search)
│   ├── Resizable IOC table with pagination
│   ├── Post-export security modal (once per session)
│   └── JS: fetchPulsePage() → extractIOCs() → renderTable() → exportCSV/JSON()
│
└── Tab 2: abuse.ch (PowerShell Generator)
    ├── Auth-Key input
    ├── Provider selection (All / MalwareBazaar / URLhaus / ThreatFox)
    ├── Generate button → builds PowerShell one-liner + auto history cleanup
    ├── Copy-to-clipboard output box
    ├── Security reminder (shown after copy — PSReadLine file path warning)
    └── Step-by-step instructions
```

**No frameworks. No npm. No server.** Pure HTML, CSS, and vanilla JavaScript in a single file.

---

## File Naming

Suggested repository name: **`multi-source-ioc-exporter`**

Suggested file name: **`multi-source-ioc-exporter.html`**

---

## Use Cases

- **SOC analysts** pulling fresh IOCs from subscribed AlienVault threat feeds for daily triage
- **Detection engineers** exporting abuse.ch IOCs (hashes, URLs, C2 IPs) for SIEM detection rules in Elastic Security, Splunk, or Microsoft Sentinel
- **Threat hunters** building blocklists (IP, domain, hash) for EDR or firewall policies
- **Incident responders** cross-referencing MITRE ATT&CK-tagged pulses during investigations
- **Researchers** quickly pivoting to source pulse context via the OTX Pulse link during analysis
- **Analysts** bulk-fetching ThreatFox IOCs with malware family classification for threat hunting

---

## Limitations

- **AlienVault OTX tab** may require a CORS browser extension when opened as `file://`. When hosted on GitHub Pages or localhost, CORS auto-detection checks on page load and may work without an extension
- **abuse.ch tab** requires PowerShell (Windows) — the HTML page generates the command, PowerShell executes it
- AlienVault fetches up to 500 pulses per session (10 pages × 50 pulses). For larger OTX subscriptions, use the OTX Python SDK or DirectConnect API
- abuse.ch APIs require a free Auth-Key from [auth.abuse.ch](https://auth.abuse.ch/)
- No authentication storage — API keys are not saved between sessions
- Internet connection required (both tabs call APIs live)
- PowerShell history cleanup (`ClearHistory()`) may fail silently on restricted environments — always verify on shared machines

---

## License

This is free and unencumbered software released into the public domain.

Anyone is free to copy, modify, publish, use, compile, sell, or distribute this software, for any purpose, commercial or non-commercial, and by any means. No attribution required.

See [https://unlicense.org](https://unlicense.org) for details.