# macOS Phishing Forensics Toolkit

Automated Evidence Collection + IOC Analysis for macOS Incident Response

This toolkit provides a two-phase macOS investigation workflow designed for IR analysts, SOC teams, DFIR students, and threat hunters who need rapid, repeatable host-based forensics during phishing investigations.

It automates evidence collection, IOC correlation, execution forensics, and timeline generation across Safari, Chrome, Firefox, QuarantineEvents, Unified Logs, persistence artifacts, and system metadata.

# Overview

The toolkit consists of two scripts:

### Phase 1 — Evidence Collection (mac_extract_phase1v2.sh)

Collects forensic artifacts from a live macOS host, including:

* Browser history (Chrome, Safari, Firefox)
* Download metadata + SHA-256 hashes
* QuarantineEventsV2 (download origin & attribution)
* Unified Logs (DNS, TLS, process execution, Gatekeeper) — sudo only
* DNS resolver configuration
* Persistence artifacts (LaunchAgents, LaunchDaemons)
* File metadata (xattrs, Spotlight data)
* Host metadata (OS, user, hostname)

Evidence is saved to:
* /tmp/mac_extract_<timestamp>/

A manifest.json with SHA-256 hashes is also generated.


### Phase 2 — IOC Search & Correlation (mac_extract_phase2.sh)

* Searches all Phase 1 artifacts for your IOC (domain, filename, keyword, etc.) and produces:
* Per-artifact IOC hit files
* A unified forensic timeline
* An IOC summary
* A human-readable Markdown report

Phase 2 creates:
* /tmp/mac_extract_<timestamp>/phase2_results_<timestamp>/




## Running Phase 1

Phase 1 supports two modes:


### Option A — Full Collection (Recommended)
Requires sudo and Terminal with Full Disk Access.
* sudo ./mac_extract_phase1v2.sh 7d


This collects:
* Unified Logs (log collect)
* System-level Quarantine DB
* Protected browser data
* More complete persistence artifacts

You will see output like:
* Folder : /tmp/mac_extract_20251205_210718



### Option B — Limited Collection (No sudo)
If sudo isn't available or you just want to test:
* ./mac_extract_phase1v2.sh --no-sudo 7d


This mode skips:

* Unified Logs
* System-level Quarantine DB
* system LaunchAgents / LaunchDaemons

Everything else is still collected.

You will see:

* [*] NOTE: Running in NO-SUDO mode.
* [*] Unified logs archive skipped (no-sudo mode).

## Running Phase 2

Phase 2 needs:
* A Phase 1 output folder
* An IOC string

You can run Phase 2 two ways:

### Option A — Specify Phase 1 folder explicitly
* /bin/bash ./mac_extract_phase2.sh /tmp/mac_extract_20251205_122702 "chatgpt.com"

### Option B — Auto-detect latest Phase 1 folder in /tmp
* /bin/bash ./mac_extract_phase2.sh "chatgpt.com"

## Phase 2 Outputs

Inside:
* /tmp/mac_extract_<timestamp>/phase2_results_<timestamp>/

you will find:

**hits/**

IOC matches from individual sources:
* chrome_urls_hits.txt
* chrome_downloads_hits.txt
* quarantine_hits.txt
* downloads_metadata_hits.txt
* safari_hits.txt
* firefox_hits.txt
* unified_dns.txt
* unified_tls.txt
* unified_general.txt
* unified_exec.txt

### timeline.tsv

* Chronologically sorted forensic timeline containing:
* Browser visits
* Downloads
* Quarantine events
* File metadata
* DNS/TLS events (sudo mode only)
* Execution events (sudo mode only)

### summary.txt
IOC hit counts by category.

### report.md
Markdown report suitable for IR documentation or ticketing systems.

# What You Can Detect
### Did the user visit a malicious URL?

From:
* Chrome/Safari/Firefox URL tables
* Unified Logs DNS lookups
* Unified Logs TLS connections

### Did they download a malicious file?

From:
* Chrome Downloads table
* Safari Downloads plist
* File metadata + hashes
* QuarantineEventsV2

### Did they open or execute the file?
Execution module correlates:
* Kernel exec() events
* Gatekeeper trustd signature checks
* UI app launches (launchservicesd)
* Process names containing the IOC

### Did persistence get created?

Checks:
* User LaunchAgents
* System LaunchAgents (sudo)
* LaunchDaemons (sudo)
* StartupItems

### Example Workflow

**1. Collect evidence
**
* sudo ./mac_extract_phase1v2.sh 7d

_**Suppose Phase 1 prints:**_
* Folder : /tmp/mac_extract_20251205_210718

**2. Run IOC correlation
**
* /bin/bash ./mac_extract_phase2.sh /tmp/mac_extract_20251205_210718 "invoice.pdf"

**3. View results
**
* open /tmp/mac_extract_20251205_210718/phase2_results_20251205_213000

# Permissions Required

### For Full Functionality (Recommended):
* Terminal must be granted Full Disk Access
* Script must be run with sudo

Without sudo:
* Unified Logs are skipped
* Some Safari and system artifacts may be incomplete
* Execution forensics cannot run

The script will warn you when:
* sudo is missing
* Full Disk Access appears to be missing
* It must fall back into no-sudo mode

# Notes for EDR / DFIR Deployment
This toolkit can be executed via:
* CrowdStrike RTR
* Velociraptor
* SSH
* Local Terminal

Phase 1 requires local shell access.
Phase 2 can run **offline**, using the evidence bundle.
