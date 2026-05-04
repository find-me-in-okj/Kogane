# Kogane  --  Certificate Transparency Threat Hunter

> Kogane (黄金, "gold") draws its name from the shikigami in Jujutsu Kaisen:
> messengers that serve as your personal interface to a game you don't fully control --
> announcing rules, tracking points, watching the field on your behalf.
> This tool does the same: your Kogane watches the global certificate stream so you don't have to.

---

## What It Does

Every time any website anywhere in the world obtains an SSL/TLS certificate, that event is logged
publicly in Certificate Transparency logs. Phishing kits, typosquatters, lookalike domains, and
brand impersonators all have to get a cert too. Kogane watches that global stream in real time and
alerts you the moment something suspicious appears that matches your watchlist.

**Designed locally. Scans globally.**

No server. No account. No API key. One HTML file.

---

## Quick Start

No install. No build step. No dependencies.

    git clone https://github.com/YOUR_USERNAME/kogane.git
    cd kogane
    open index.html        # macOS
    start index.html       # Windows
    xdg-open index.html    # Linux

Or host it on GitHub Pages (see below) for HTTPS access and live CertStream connectivity.

---

## How It Works

Kogane connects to CertStream (certstream.calidog.io), a free public WebSocket feed maintained
by Calidog Security that aggregates Certificate Transparency logs from all major CT servers globally.
When that connection is available, Kogane processes the live stream in real time -- typically
10-50 new certificates per second.

If CertStream is unreachable (corporate proxy, browser security policy, offline), Kogane
automatically falls back to simulation mode, which runs the identical matching engine with
generated certificates seeded to produce realistic hits. Simulation mode works on any local
file:// open; live mode requires an HTTPS context.

---

## Matching Engine

When a certificate arrives, every domain in it is checked against your watchlist using five layers:

### Domain patterns  (type: domain)

  Exact clone      --  domain registered as exact match (catches cross-TLD shadow registrations)  Score: 100
  Homoglyph swap   --  paypa1.com, rnicosoft.com (character substitution)                         Score:  91
  Subdomain abuse  --  login.paypal.com.attacker.net                                              Score:  88
  TLD swap         --  paypal.net, paypal.io                                                      Score:  82
  Typosquat        --  Levenshtein distance <= 2: paypall.com, paupal.com                        Score: 72-86
  Brand embedded   --  paypal-secure-login.com                                                    Score:  72

### Keyword patterns  (type: keyword)

  Exact TLD        --  keyword registered as keyword.com / .net / .org / .io                     Score:  90
  Keyword present  --  keyword appears anywhere in the domain                                     Score:  72
  Fuzzy per-segment--  each domain segment checked individually with fuzzy match                  Score: 43-68

### Threat levels

  CRITICAL  90-100   |   HIGH  75-89   |   MEDIUM  55-74   |   LOW  0-54

---

## Features

  Capture controls
    - Wireshark-style START / STOP capture with live duration clock and per-session alert count
    - Feed, alert list, and watchlist each have individual CLEAR buttons
    - Nothing is recorded until a capture is started; stopping freezes the snapshot for review

  Detection
    - Live CT feed via CertStream WebSocket (wss://certstream.calidog.io/)
    - Auto-fallback to simulation mode if stream is unavailable
    - Five-layer matching engine with scored threat levels
    - Hit notifications: Kogane buddy ASCII art, DING-DONG DING-DONG DING-DONG!, domain + score

  Watchlist
    - Persistent watchlist stored in browser localStorage (survives page refresh)
    - Keyword and domain pattern types
    - Adding a new pattern immediately re-scores the existing feed retroactively

  Feed
    - All certs / Hits only filter toggle
    - Live domain search filter
    - Detail inspector: full cert info, all SANs, per-match breakdown with scores

  Active Alerts panel
    - Sort by: time newest, time oldest, score high, score low
    - Filter by: watchlist pattern, threat level band (Critical / High / Medium / Low)
    - Filter by time range: from any year down to the exact second
    - Count bar showing visible vs total alerts

  Report export
    - Three formats: plain TEXT, JSON, CSV
    - Live preview before download
    - TEXT: human-readable with header, stats, pattern hit counts, full alert list
    - JSON: structured, nested, importable into other tools
    - CSV: one row per match, ready for Excel or command-line analysis
    - Auto-timestamped filenames

  General
    - In-tool feature guide (GUIDE button in header)
    - Live stats: certs/sec rate, total scanned, match count, session uptime
    - Zero dependencies -- single HTML file, no build step, no install

---

## Project Structure

    kogane/
    +-- index.html    The entire application. Open this.
    +-- README.md     This file.
    +-- GUIDE.md      Quick-reference guide for all features.

Kogane is intentionally a single HTML file. Fork it, embed it, host it on GitHub Pages,
drop it on a USB drive. It works anywhere a browser runs.

---

## Hosting on GitHub Pages

1. Push this repo to GitHub
2. Settings -> Pages -> Source: main branch, / (root)
3. Visit https://YOUR_USERNAME.github.io/kogane/

Note: GitHub Pages serves HTTPS, which is required for the CertStream WebSocket connection.
Running index.html directly from disk (file://) will fall back to simulation mode in most browsers
due to mixed-content restrictions on WebSocket connections.

---

## Customization

All logic lives in index.html. Key sections:

  Default watchlist    --  find loadWL(), edit the fallback array
  Matching thresholds  --  find analyze(), adjust Levenshtein distance limits and score values
  Simulation speed     --  find simLoop(), change the setTimeout delay
  Notification timing  --  find NOTIF_DURATION, change the auto-dismiss delay in milliseconds
  Theme colors         --  CSS custom properties in :root at the top of the <style> block
  CertStream URL       --  find connect(), change the WebSocket URL to your own aggregator

---

## CertStream

Kogane uses CertStream by Calidog Security, a free public WebSocket feed.

  Feed URL:       wss://certstream.calidog.io/
  No key needed.  Read-only. One-way stream.

To run your own CT log aggregator instead of relying on CertStream:
https://github.com/CaliDog/certstream-server

---

## Privacy

Kogane runs entirely in your browser. Your watchlist patterns are stored only in your browser's
localStorage. No data is sent to any server except for the outbound WebSocket connection to
certstream.calidog.io to receive the public CT feed. That feed is read-only and one-directional.
Exported reports are generated and downloaded entirely client-side.

---

## Contributing

Issues and pull requests welcome. Areas with room to grow:

  - Persistent alert log across sessions (IndexedDB)
  - Desktop notifications via the Notifications API
  - Configurable scoring weights per pattern
  - WHOIS / DNS enrichment on hits
  - Multiple simultaneous watchlists (profiles)

---

## License

MIT. Use it, fork it, build on it.

---

Kogane (黄金, gold). The thing you are hunting in a field of noise.

Developed by Max Malz.
