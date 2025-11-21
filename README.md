# macOS Phishing Forensics Toolkit
Automated Evidence Collection + IOC Analysis for macOS Incident Response

This project provides a two-phase macOS investigation toolkit built for IR analysts, SOC teams, and threat hunters who need to quickly triage phishing incidents on macOS endpoints.

It automates:

Browser history extraction

Download activity & metadata collection

Quarantine event extraction

Unified Logs acquisition (DNS, TLS, exec events, etc.)

Persistence discovery

IOC searching & correlation

Timeline/report generation

The result is a repeatable, host-based forensic workflow you can run manually or deploy via EDR tools (CrowdStrike RTR, Velociraptor, etc.).



Overview

The toolkit consists of:

Phase 1 — Evidence Collection (mac_extract_phase1.sh)

Creates a complete evidence package from a live macOS host, including:

Browser history (Chrome, Safari, Firefox)

Download metadata (~/Downloads)

QuarantineEventsV2 (download origin, app used, timestamps)

Unified Logs (log show) for a specified time window

DNS lookups

TLS handshakes

Process events (via Phase 2)

Cookies, cache, preferences (browser artifacts)

Persistence artifacts (LaunchAgents, LaunchDaemons, StartupItems)

Network configuration (DNS servers, hardware ports)

Host metadata (OS version, hostname, user)

Output is saved to:

/tmp/mac_extract_<timestamp>/


The script also generates a manifest.json documenting all artifacts, complete with SHA-256 hashes.

Phase 2 — IOC Search & Correlation (mac_extract_phase2.sh)

Accepts a Phase 1 folder (or the newest one in /tmp) + your IOC string (domain, filename, hash, keyword), and performs:

Full recursive IOC search across all Phase 1 artifacts

Browser history correlation (URLs + download tables)

Download metadata correlation

QUARANTINE correlation (file origin, timestamp, app used)

Unified Logs searches

DNS lookups

TLS trustd validations

General process messages



Phase 2 now extracts process execution events tied to your IOC using Unified Logs:

exec() calls logged by the kernel

trustd code signing checks (Gatekeeper validations)

launchservicesd UI-launch events

Any process whose name matches/contains the IOC

Any unified log event mentioning the downloaded file

This detects scenarios like:

A malicious PDF that was opened

A payload inside a ZIP that was executed

An unsigned binary that triggered Gatekeeper

All execution findings are appended to the master timeline.

📊 Phase 2 Outputs

Inside the Phase 1 folder, Phase 2 generates a results directory:

/tmp/mac_extract_<timestamp>/phase2_results_<timestamp>/


Contents include:

✔ hits/

Raw grep hits for your IOC:

chrome_urls_hits.txt

chrome_downloads_hits.txt

quarantine_hits.txt

unified_dns.txt

unified_tls.txt

unified_general.txt

unified_exec.txt ← NEW

downloads_metadata_hits.txt

firefox_hits.txt

safari_hits.txt

✔ timeline.tsv

A sorted forensic timeline combining:

Browser history

Download activity

Quarantine events

DNS/TLS unified logs

Process execution events

File metadata events

✔ summary.txt

Count of IOC hits by category.

✔ report.md

Human-readable report — suitable for tickets, IR writeups, etc.

Running the Toolkit
Run Phase 1 (Evidence Collection)
chmod +x mac_extract_phase1.sh
sudo ./mac_extract_phase1.sh 7d


The 7d argument controls how far back Unified Logs are collected.
You can use:

24h

48h

30d

1h

Example output:

Folder : /tmp/mac_extract_20251120_210718
Tarball: /tmp/mac_extract_20251120_210718.tgz




Run Phase 2 (IOC Search & Correlation)
Option A: Specify Phase 1 folder explicitly
sudo /bin/bash ./mac_extract_phase2.sh /tmp/mac_extract_20251120_210718 "chatgpt.com"

Option B: Auto-detect latest Phase 1 folder
sudo /bin/bash ./mac_extract_phase2.sh "chatgpt.com"

What You Can Detect With This Toolkit
✔ Did the user visit a malicious URL?

Chrome / Safari / Firefox history, Unified Logs DNS/TLS

✔ Did they download a malicious file?

Chrome Downloads.csv, Safari downloads, Quarantine DB, file metadata

✔ Did they open/run the file?

NEW execution module detects:

kernel exec events

trustd signer checks

launchservices app launches

Gatekeeper validations

executable names containing the IOC

✔ Was persistence created?

LaunchAgents, Daemons, StartupItems scanned

✔ What happened when?

Master timeline aligns browser activity, downloads, execution, DNS lookups, and system events in chronological order.
