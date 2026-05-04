# Kogane  --  Feature Guide

Quick reference for all features. See README.md for full documentation.

---

## Startup

On first load, the splash screen shows the full Kogane logo art.
Click anywhere to dismiss and enter the tool.
Click the mini logo in the top-left of the header at any time to bring the splash back.

---

## Capture Bar

The capture bar sits directly below the header and controls all recording.
Nothing is processed or recorded until you press START. This mirrors Wireshark's capture model.

  [ START ]          --  Begin a capture session. Starts the duration clock, records all
                         incoming certs and alerts to the current session. The connection to
                         CertStream (or simulation) runs continuously in the background;
                         START/STOP controls what gets recorded, not the connection itself.

  [ STOP ]           --  Freeze the capture. Feed and alerts hold their current state.
                         Use this to study a snapshot without new data overwriting it.

  [ EXPORT REPORT ]  --  Open the report modal at any time (during or after capture).

  Duration clock     --  Time elapsed since the current capture started (HH:MM:SS).
  Session alerts     --  Count of alerts recorded during the current capture only.

---

## Watchlist  (left panel)

Your watchlist is the set of patterns Kogane checks every incoming certificate against.
Patterns are saved to localStorage and persist across sessions.

  Adding a pattern:
    1. Type a domain or keyword in the input at the bottom of the sidebar
    2. Select the type: "keyword" or "domain"
    3. Press Enter or click + ADD

  Pattern types:
    keyword  --  Matches if the term appears in, or fuzzy-matches a segment of, any domain.
                 Best for brand names, product names, internal codenames.
                 Example: "paypal" catches paypal.com, secure-paypal.net, paypa1.io

    domain   --  Checks full domains for exact clone, homoglyph, TLD swap,
                 subdomain abuse, typosquat, and brand-embedded detection.
                 Best for specific domains you own or monitor.
                 Example: "google.com" catches g00gle.com, google.net, login.google.com.phish.io

  Removing a pattern:
    Click [x] on any watchlist entry.

  CLEAR button:
    Removes all watchlist patterns and clears localStorage. Use with caution.

  Adding a new pattern immediately re-scores the existing feed --
  any recent cert matching the new pattern is retroactively flagged.

---

## Certificate Feed  (center panel)

Live stream of incoming TLS certificates, newest at top.
Nothing appears here until a capture is started.

  Normal certs  --  dimmed text, no flag
  Hit certs     --  bold text, [!] flag, match label + score pills, green background

  Click any hit row to open the detail inspector at the bottom.

  Filter bar:
    ALL        --  show every cert processed in this capture
    HITS ONLY  --  show only certs that matched your watchlist
    Search box --  live filter by domain text (as you type)
    CLEAR      --  wipe the feed display and reset the scanned counter

---

## Detail Inspector  (bottom strip of center panel)

Click any hit in the feed or alerts panel to inspect it.

  Shows:
    - Domain name and threat level badge with score
    - Certificate issuer
    - Detection timestamp
    - All Subject Alternative Names (SANs) in the certificate
    - Total match count
    - Per-match row: matched domain / vs watched pattern / detection method / score

---

## Active Alerts  (right panel)

Chronological log of every hit detected, with full filter and sort controls.

  Sort:
    Time (newest)  --  most recent hit at top (default)
    Time (oldest)  --  oldest hit at top
    Score (high)   --  highest threat score at top
    Score (low)    --  lowest threat score at top

  Filter by watchlist pattern:
    Dropdown auto-populated from every pattern that has fired.
    Select one to show only alerts triggered by that specific watchlist entry.

  Filter by threat level:
    All levels / Critical (90+) / High (75+) / Medium (55+) / Low (<55)

  Filter by time range:
    FROM and TO datetime inputs with per-second precision (YYYY-MM-DD HH:MM:SS).
    Use FROM only, TO only, or both together to isolate any time window.

  CLEAR (filter bar):
    Resets all sort and filter controls back to defaults. Does not remove alerts.

  CLEAR (panel header):
    Removes all alerts from the list entirely and resets the match counter.

  Count bar:
    Shows "N of M alerts" -- how many are visible given current filters vs total collected.

  Each alert row shows:
    - Domain name
    - Which watchlist pattern(s) triggered it
    - Threat level badge with score
    - Date and time of detection

  Click any row to open it in the detail inspector.
  Selected alert is highlighted with a left border.

---

## Hit Notifications

Every hit triggers a notification in the bottom-right corner.

  Contains:
    - Kogane buddy ASCII art (white on black)
    - DING-DONG DING-DONG DING-DONG!
    - Matched domain
    - Threat level badge
    - Watched pattern, detection method, score

  Auto-dismisses after 6 seconds with a progress bar.
  Click [x] to dismiss early.
  Multiple hits queue and display one at a time.

---

## Report Export

Click [ EXPORT REPORT ] in the capture bar to open the report modal.

  Formats:
    TEXT  --  Human-readable plain text. Includes session metadata, watchlist, summary
              stats, per-pattern hit counts, and every alert with full match breakdown.
              Good for sharing, archiving, or pasting into tickets.

    JSON  --  Structured, machine-readable. Includes all metadata and nested match arrays.
              Good for importing into other tools, SIEM pipelines, or scripting.

    CSV   --  One row per match (flattened). Columns: timestamp, domain, all SANs,
              issuer, score, threat level, matched pattern, match label, match score.
              Good for Excel, Python, or command-line analysis.

  Preview pane shows the full report before download.
  Footer shows alert count, line count, and file size.
  [ DOWNLOAD ] saves a timestamped file: kogane-report-YYYY-MM-DDTHH-MM-SS.{ext}
  All generation and download happens entirely client-side.

---

## Header Controls

  SIM MODE  --  Toggle simulation mode on/off manually.
                In sim mode, Kogane generates realistic fake certificates with seeded hits
                using the identical matching engine and scoring as live mode.
                Kogane auto-enables sim mode if CertStream is unreachable.
                Useful for testing watchlist patterns before going live.

  GUIDE     --  Opens this guide as an in-tool overlay.

  Connection indicator (dot + label):
    Connecting   --  attempting CertStream WebSocket connection
    Live Stream  --  receiving real global CT data
    Sim Mode     --  running local simulation
    Disconnected --  CertStream unreachable, simulation active

---

## Stats Bar  (top-right of header)

  SCANNED  --  total certificates processed since page load
  MATCHES  --  total hits found since page load
  RATE     --  average certificates per second (5-second rolling window)
  UPTIME   --  time elapsed since page load

---

## Sim Mode vs Live Mode

  Live mode:  Requires HTTPS context (GitHub Pages or any HTTPS web server).
              Processes real global certs from certstream.calidog.io.
              Typical rate: 10-50 certs/sec depending on time of day.

  Sim mode:   Works anywhere, including local file:// opens.
              Same matching engine, same scoring, same UI.
              Hits are seeded at a realistic interval (~every 10-20 certs).
              Use this for testing watchlist patterns or demoing the tool offline.

---

## Tips

  - Keep keywords at least 5 characters. Short terms produce false positives.
  - Add your own domain as type:domain to catch all six attack categories at once.
  - Scores above 85 on keyword patterns usually indicate genuine threats worth investigating.
  - Stop the capture before exporting -- the report uses session alerts when a capture
    has been run, and all alerts otherwise.
  - Use the time range filter in the alerts panel to isolate a specific burst of activity.
  - The feed buffer holds 200 entries internally. Use HITS ONLY or the search box in
    high-volume sessions to keep the relevant certs in view.
  - Watchlist patterns added mid-capture retroactively flag matching certs already in the feed.

---

Developed by Max Malz.
