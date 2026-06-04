# Mac Doctor

**All-In-One Toolkit (Similar to CleanMyMac). Open source Mac cleaner working for codex and claude. Three-phase macOS maintenance: scan caches and junk, check malware and privacy traces, optimize system speed.**

A free alternative to paid tools like CleanMyMac, MacBooster, and MacKeeper. Runs entirely inside Codex. No installation, no subscription, no telemetry.

---

## Table of Contents

<details>
<summary>Click to expand/collapse</summary>

- [What Is Mac Doctor](#what-is-mac-doctor)
- [Why This Exists](#why-this-exists)
- [How It Works — High-Level Flow](#how-it-works--high-level-flow)
- [Phase 1 — Smart Scan](#phase-1--smart-scan)
  - [Full Category Descriptions](#full-category-descriptions)
- [Phase 2 — Protection](#phase-2--protection)
  - [Malware Scan](#malware-scan)
  - [Privacy Scan](#privacy-scan)
- [Phase 3 — Speed Optimization](#phase-3--speed-optimization)
  - [Full Step Details](#full-step-details)
- [Safety and Transparency](#safety-and-transparency)
- [Usage Guide](#usage-guide)
- [Comparison: Mac Doctor vs CleanMyMac](#comparison-mac-doctor-vs-clearmymac)
- [What Mac Doctor Does Not Do](#what-mac-doctor-does-not-do)
- [Before and After — Real Measurements](#before-and-after--real-measurements)
- [FAQ](#faq)
- [Troubleshooting](#troubleshooting)
- [Technical Details](#technical-details)
- [File Structure](#file-structure)
- [License](#license)

</details>


## What Is Mac Doctor

Mac Doctor is a Codex skill — a set of structured instructions that Codex follows to perform a complete Mac maintenance cycle. It is not a standalone application, not a background service, and not a subscription product.

The skill defines three phases that run in sequence:

1. **Scan** — examines 19 categories of cache files, log files, junk data, and system artifacts
2. **Protect** — checks for malware indicators across eight inspection methods and scans privacy traces across browsers and chat applications
3. **Speed** — executes 17 system optimization steps and measures the difference

Every command Mac Doctor runs is visible in the conversation. Every delete operation waits for explicit user confirmation. No data leaves your machine.

---

## Installation

Anyone can install Mac Doctor from GitHub in one command:

```bash
git clone https://github.com/veritasian/mac-doctor.git ~/.agents/skills/mac-doctor
```

Or with GitHub CLI:

```bash
gh repo clone veritasian/mac-doctor ~/.agents/skills/mac-doctor
```

After cloning, Codex picks it up automatically. No config file changes, no restart, no build step.

You can then invoke it by typing **@mac-doctor** or saying any of:

- "Clean up my Mac"
- "Scan for junk files"
- "Check for malware"
- "Speed up my Mac"

The repo contains exactly two files:

| File | Purpose |
|---|---|
| `SKILL.md` | The instructions Codex follows at runtime |
| `README.md` | This documentation |

Nothing to build. Nothing to install beyond the clone.
## Why This Exists

macOS accumulates data over time. Cache files that applications never clean up. Log files that grow until you notice your startup disk is full. Browser history that keeps every page you visited. Time Machine snapshots that consume invisible gigabytes. Language packs for languages you do not speak. Xcode build artifacts that multiply with every project.

Most users have three options:

**Option 1 — Paid cleaners.** CleanMyMac charges $119.88 per year. MacBooster charges $39.95. MacKeeper is widely considered adware. These tools work, but they are closed-source, collect telemetry, and charge a recurring fee for a task that takes 30 seconds of terminal commands.

**Option 2 — Terminal commands.** The commands exist — `du`, `rm`, `purge`, `tmutil`, `mdutil`, `dscacheutil`. But knowing which directories to target, which flags are safe, which deletions are reversible, and which require caution takes hours of research. One wrong `rm -rf` with a misplaced path breaks your system.

**Option 3 — Do nothing.** The disk fills up. The system slows down. Spotlight reindexes at the worst times. Applications crash because cache corruption causes state conflicts.

Mac Doctor is a fourth option. It wraps the terminal commands that macOS provides, runs them in a safe sequence, shows you every command before execution, and asks for permission. The skill file is readable and editable. You can inspect every line, add exclusions, or adjust thresholds.

---

## How It Works — High-Level Flow

```
                    ┌──────────────────────┐
                    │     USER INVOKES     │
                    │  @mac-doctor or NLP  │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │  PHASE 1 — SCAN      │
                    │  19 categories       │
                    │  du / find in parallel│
                    │  Report sizes table   │
                    │  No deletions         │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │  RESULTS PRESENTED   │
                    │  "Found X total."    │
                    │  Ask user: proceed?  │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │  PHASE 2 — PROTECT   │
                    │  Malware scan (8 chk)│
                    │  Privacy scan (10 chk)│
                    │  Report findings      │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │  CONFIRM CLEANUP     │
                    │  "Clear everything?" │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │  PHASE 3 — SPEED     │
                    │  17 optimization steps│
                    │  Measure before/after │
                    │  Show delta            │
                    └──────────────────────┘
```

---

## Phase 1 — Smart Scan

Mac Doctor runs `du` and `find` against 19 locations in parallel and compiles a single table. Nothing is deleted. The purpose is to give you a complete picture of what occupies space on your machine, what is safe to delete, and what requires caution.

| # | Category | Path | Typical Size | Safety |
|---|---|---|---|---|
| 1 | User Cache | `~/Library/Caches/` | 100 MB - 5 GB | Safe |
| 2 | Hidden Cache | `~/.cache/` | 0 - 5 GB | Safe |
| 3 | System Cache | `/Library/Caches/` | 0 - 1 GB | Safe (some root-owned) |
| 4 | User Logs | `~/Library/Logs/` | 1 MB - 500 MB | Safe |
| 5 | System Logs | `/Library/Logs/` + `/var/log/` | 10 MB - 500 MB | Safe (needs root) |
| 6 | Language Packs | `.lproj` in apps | 100 MB - 2 GB | Safe (strip with tool) |
| 7 | Trash | `~/.Trash/` | 0 - 50 GB | Safe |
| 8 | Mail Attachments | `~/Library/Mail/` | 0 - 5 GB | Safe |
| 9 | Xcode DerivedData | `~/Library/Developer/Xcode/DerivedData/` | 0 - 50 GB | Safe (rebuilds) |
| 10 | Xcode DeviceSupport | `~/Library/Developer/Xcode/iOS DeviceSupport/` | 0 - 20 GB | Safe (re-downloads) |
| 11 | Xcode Archives | `~/Library/Developer/Xcode/Archives/` | 0 - 10 GB | Safe (if builds final) |
| 12 | Old Updates | `/Library/Updates/` | 0 - 10 GB | Safe |
| 13 | Unused DMGs | `~/Documents/`, `~/Downloads/` | 0 - 2 GB | Safe |
| 14 | Downloads | `~/Downloads/` | 0 - 50 GB | Review before delete |
| 15 | Stale Preferences | `~/Library/Preferences/` | 0 - 50 MB | Safe |
| 16 | iOS Backups | `~/Library/Application Support/MobileSync/Backup/` | 0 - 100 GB | Confirm before delete |
| 17 | Document Versions | `~/.DocumentRevisions-V100/` | 0 - 10 GB | Safe |
| 18 | Login Items | System Settings | N/A | Review only |
| 19 | Universal Binaries | `/Applications` | 0 - 5 GB | Caution |

### Full Category Descriptions

**1. User Cache — `~/Library/Caches/`**

Every macOS application stores temporary data in its own subdirectory under `~/Library/Caches/`. Browsers cache web page assets. Media apps cache thumbnails and transcoded files. IDEs cache compilation artifacts. These files are safe to delete because applications recreate them on demand. Over months of use, this directory can grow from a few hundred megabytes to several gigabytes.

**2. Hidden Cache — `~/.cache/`**

Command-line tools store their own caches in the hidden `~/.cache/` directory. Python package managers like `uv` and `pip` store downloaded package wheels here. AI tools like HuggingFace store model metadata. The largest consumer in this directory is typically `uv` (Python package manager), which can cache over 1.5 GB of package data. Safe to delete — tools re-download what they need.

**3. System Cache — `/Library/Caches/`**

macOS itself caches system data here: icon thumbnails, font caches, kernel module caches, and system framework data. Most of these directories are owned by root and require `sudo` to clear. They are safe to delete but macOS will recreate them during normal operation.

**4. User Logs — `~/Library/Logs/`**

Applications write diagnostic logs to `~/Library/Logs/`. These include crash reports, operation logs, update logs, and debugging output. They accumulate over time and are rarely read by anyone. CleanMyMac logs, Codex logs, crash reporters, and simulator logs are the most common contributors.

**5. System Logs — `/Library/Logs/` + `/var/log/`**

macOS system components write logs to `/Library/Logs/` and `/var/log/`. These include installation logs, WiFi diagnostic captures, kernel crash reports, and system update logs. The largest consumers are often CoreCapture WiFi diagnostics (hundreds of megabytes of pcap and text data) and DiagnosticReports (crash reprots that macOS never deletes). These are safe to clear but require `sudo`.

**6. Language Packs**

Every macOS application bundles `.lproj` directories containing localized strings for different languages. A single application can include over 2,800 language packs. Applications like Pages Creator Studio (2,839 packs), Safari (1,195 packs), and Chrome (164 packs) are the largest offenders. Stripping non-English language packs can recover hundreds of megabytes. This requires a dedicated tool like Monolingual and is reported but not executed by Mac Doctor.

**7. Trash — `~/.Trash/`**

Files moved to Trash remain on disk until the Trash is emptied. macOS does not automatically empty the Trash. Files deleted weeks or months ago continue occupying real disk space. Common large items include old DMG installers, video files, and project archives.

**8. Mail Attachments — `~/Library/Mail/`**

The Mail application stores downloaded attachments and the message database in `~/Library/Mail/`. Over time, this grows as emails accumulate. The download cache can contain large attachments you already viewed.

**9. Xcode DerivedData**

Xcode creates a DerivedData directory for every project you build. It contains compiled object files, module caches, precompiled headers, and symbol tables. For iOS developers with multiple projects, DerivedData routinely exceeds 10 GB. It is safe to delete because Xcode regenerates it on the next build.

**10. Xcode DeviceSupport**

When you connect an iOS device to Xcode, the system downloads symbol files for that iOS version into DeviceSupport. Over time, symbol files for multiple iOS versions accumulate. Each version can be 2-5 GB. If you no longer need to debug on older iOS versions, this directory can be safely deleted.

**11. Xcode Archives**

When you archive a build for distribution, Xcode saves it in Archives. Each archive is a full .xcarchive bundle containing the app binary and debug symbols. Old archives from released versions can accumulate GBs of data. Safe to delete if the builds are already distributed or uploaded to App Store Connect.

**12. Old Updates — `/Library/Updates/`**

macOS stores installer packages in `/Library/Updates/` after downloading them. Even after the update is installed, the package remains. These are safe to delete.

**13. Unused DMGs**

Applications distributed as DMG files leave the disk image in your Downloads or Documents folder after you drag the app to Applications. These files can range from 50 MB to several GB. Once the application is installed, the DMG serves no purpose.

**14. Downloads — `~/Downloads/`**

The Downloads folder is the default destination for every file you download. It accumulates installers, ZIP files, documents, images, and archives. Files older than 90 days are candidates for deletion. Review before deleting if you use Downloads as long-term storage.

**15. Stale Preferences — `~/Library/Preferences/`**

Preference files (`.plist`) are created by applications to store settings. When an application is uninstalled, its preference file often remains. These orphaned plist files accumulate over years. macOS does not auto-clean them. Files older than 365 days are candidates for deletion.

**16. iOS Backups**

iTunes and Finder create full device backups in `MobileSync/Backup/`. Each backup is a complete snapshot of an iOS device: 20-40 GB per backup. If you have multiple backups from different devices or different dates, this directory can consume over 100 GB.

**17. Document Versions — `~/.DocumentRevisions-V100/`**

macOS automatic document versioning creates snapshots every time you save a document. These snapshots are stored in the `.DocumentRevisions-V100` directory. While macOS manages this space, old revisions from deleted documents can remain.

**18. Login Items**

Login items are applications that launch automatically when you log in. Over time, old entries can point to deleted applications. Mac Doctor lists current login items and launch agents for manual review.

**19. Universal Binaries**

macOS applications are often distributed as universal binaries containing code for both Intel and Apple Silicon architectures. On Apple Silicon Macs, the Intel slice is never used. Stripping the Intel slice from selected applications can free space, but this is a complex operation that Mac Doctor does not automate.

---

## Phase 2 — Protection

### Malware Scan

The malware scan inspects eight areas using built-in macOS tools. It does not use a virus database. It relies on file signature verification, pattern matching, and system configuration audits.

**DMG Code Signature Verification**

Disk images in `~/Downloads/` and `~/Documents/` are checked with `codesign -dv` and `hdiutil verify`. This confirms that the DMG has a valid Apple Developer ID signature and has been notarized by Apple. Unsigned or revoked signatures are flagged.

**Archive Scan**

Recently downloaded archive files (ZIP, tar.gz, 7z, RAR) are listed by `find` with mtime filtering. The scan checks what files were downloaded within the last 30 days and lists them for manual review.

**Malware Pattern Search**

A `find` search against known categories of potentially unwanted programs:

- `keygen` — software key generators (often bundled with malware)
- `crack`, `patch` — cracked software installers
- `activator`, `loader` — activation tools
- `MacKeeper`, `MacBooster` — known potentially unwanted applications

False positives from git hooks (`.git/hooks/applypatch-msg.sample` and similar) are filtered out in reporting.

**System Security Audit**

Three macOS security mechanisms are checked:
- `csrutil status` — System Integrity Protection (should be enabled)
- `spctl --status` — Gatekeeper (should be enabled)
- `xprotect version` — XProtect malware definition version (should be current)

If any of these are disabled or outdated, a warning is reported.

**Chrome Extension Review**

Each installed Chrome extension is identified by reading its `manifest.json` name field. This produces a list of human-readable extension names that you can verify against your known extensions. Extensions you do not recognize can be investigated.

**Email Attachment Check**

The Mail application download cache at `~/Library/Containers/com.apple.mail/Data/Downloads/` is scanned for files. Suspicious extensions (`.exe`, `.zip`, `.dmg`) are flagged.

**USB Drive Listing**

All mounted volumes are listed via `ls /Volumes/`. System volumes are filtered out to show only user-connected drives.

**iCloud Download Check**

The iCloud download cache path is checked for synced files.

### Privacy Scan

The privacy scan reports data size for the following locations. It does not read file contents — only reports directory size.

**Safari Browsing Data**

- `History.db` — every page visited, with timestamps
- `Bookmarks.db` — saved bookmarks
- `CloudTabs.db` — tabs synced via iCloud
- Autofill credit cards (separate file in `~/Library/Autofill Credit Cards/`)
- `LocalStorage/` — website local data

**Chrome Browsing Data**

- `History` — full browsing history database (typically 5-20 MB)
- `Cookies` — session tokens and tracking cookies (often 2-5 MB)
- `Login Data` — saved website passwords
- `Web Data` — autofill form entries
- `Bookmarks` — saved bookmarks in JSON format

**Chat Application Data**

Each installed chat application is checked by testing whether its data directory exists:

- **WeChat** — `~/Library/Containers/com.tencent.xinWeChat/` — chat history, images, voice messages, file transfers
- **QQ** — `~/Library/Containers/com.tencent.qq/` — message database, cached files
- **Telegram** — `~/Library/Application Support/Telegram Desktop/` — cached media, chat export files
- **Discord** — `~/Library/Application Support/discord/` — message cache, attachment downloads
- **Slack** — `~/Library/Application Support/Slack/` — workspace data, file history
- **Skype** — `~/Library/Application Support/Skype/` — conversation archive, call logs
- **Feishu / Lark** — `~/Library/Application Support/Feishu*/` — document and chat cache
- **Signal** — `~/Library/Application Support/Signal/` — encrypted message store

For each installed application, the total data size is reported. Users can decide whether to clear it manually.

**System Activity Traces**

- Recent documents list (`.sfl` files in `~/Library/Application Support/com.apple.sharedfilelist/`)
- Recent server connections
- Spotlight search cache
- QuickLook thumbnail cache

---

## Phase 3 — Speed Optimization

After scanning and protection, Mac Doctor runs 17 optimization steps in sequence. It records disk usage and free RAM before step 1 and after step 17.

### Full Step Details

**Step 1 — Free RAM**
- Command: `purge`
- What it does: Forces macOS to release inactive memory pages that have not been accessed recently. On a system that has not been restarted in days, this can free 1-4 GB of RAM.
- Safety: Safe. Applications that need the memory can request it again.
- Expected impact: High on machines that have been running for days with many applications.

**Step 2 — Clear User Caches**
- Command: `rm -rf ~/Library/Caches/*`
- What it does: Deletes every subdirectory in the user cache folder. Applications that are currently running may create new cache files immediately.
- Safety: Safe. All caches are temporary by definition.
- Expected impact: Medium. Recovers hundreds of MB to several GB depending on usage.

**Step 3 — Clear System Caches**
- Command: `rm -rf /Library/Caches/*`
- What it does: Removes system-level cache files. Requires escalated privileges.
- Safety: Safe. System caches are recreated during normal operation.
- Expected impact: Low. System caches are usually small on modern macOS.

**Step 4 — Flush DNS Cache**
- Command: `dscacheutil -flushcache; sudo killall -HUP mDNSResponder`
- What it does: Clears the DNS resolver cache and restarts the mDNSResponder daemon. Resolves stale DNS lookups and improves browsing speed.
- Safety: Safe. DNS cache rebuilds automatically.
- Expected impact: Low on space but noticeable if DNS resolution has been slow.

**Step 5 — Free Purgeable Space**
- Command: `tmutil thinlocalsnapshots / 9999999999999`
- What it does: Trims local Time Machine snapshots beyond their retention limit. These snapshots are created by macOS for Time Machine and appear as "purgeable" space in Finder.
- Safety: Safe. Only snapshots past the retention window are removed.
- Expected impact: Medium. Can free several GB of purgeable space.

**Step 6 — Clear Recent Items**
- Command: `rm -f ~/Library/Preferences/com.apple.recentitems.plist`
- What it does: Clears the lists of recent applications, documents, and servers in the Apple menu and Finder.
- Safety: Safe. These lists regenerate as you use applications.
- Expected impact: Minimal space. Primarily privacy-focused.

**Step 7 — Reindex Spotlight**
- Command: `mdutil -E /`
- What it does: Triggers a full rebuild of the Spotlight search index. Fixes slow, incomplete, or broken search results.
- Safety: Safe. Spotlight will re-index all files. This takes time.
- Expected impact: Performance improvement in search. No space recovery.

**Step 8 — Remove Local Snapshots**
- Command: `tmutil deletelocalsnapshots /`
- What it does: Removes all deletable local Time Machine snapshots. These are snapshots that macOS creates periodically for "local snapshots" feature — independent of Time Machine backups.
- Safety: Safe. macOS will create new snapshots as needed.
- Expected impact: High. Can free tens of GB depending on snapshot age.

**Step 9 — Clear Old DMGs and Packages**
- Command: `find ~/Downloads ~/Documents -name "*.dmg" -o -name "*.pkg" -mtime +7 -delete`
- What it does: Removes disk images and installer packages older than 7 days from Downloads and Documents.
- Safety: Safe if you have already installed the software. Review threshold with user.
- Expected impact: Low to medium.

**Step 10 — Clear Old Downloads**
- Command: `find ~/Downloads -type f -mtime +90 -delete`
- What it does: Deletes all files in Downloads that have not been modified in over 90 days.
- Safety: Confirm with user. Some users keep important files in Downloads.
- Expected impact: Medium. Downloads folders can accumulate 1-10 GB of old files.

**Step 11 — Remove iOS Backups**
- Command: `rm -rf ~/Library/Application Support/MobileSync/Backup/*`
- What it does: Deletes all iOS device backups.
- Safety: Irreversible. Confirm before executing.
- Expected impact: Very high. Each backup is 20-40 GB.

**Step 12 — Remove Stale Preferences**
- Command: `find ~/Library/Preferences -name "*.plist" -mtime +365 -delete`
- What it does: Removes preference files that have not been modified in over a year. These are typically from uninstalled applications.
- Safety: Safe. macOS creates a new plist if the application runs again.
- Expected impact: Minimal space (typically under 50 MB).

**Step 13 — Clear Xcode DerivedData**
- Command: `rm -rf ~/Library/Developer/Xcode/DerivedData/*`
- What it does: Removes all build artifacts from Xcode projects.
- Safety: Safe. Xcode regenerates on next build.
- Expected impact: High for iOS developers. 5-50 GB typical.

**Step 14 — Remove Xcode DeviceSupport**
- Command: `rm -rf ~/Library/Developer/Xcode/iOS\ DeviceSupport/*`
- What it does: Removes iOS device symbol files.
- Safety: Safe. Xcode re-downloads on next device connection.
- Expected impact: Medium. 2-20 GB per iOS version.

**Step 15 — Remove Xcode Archives**
- Command: `rm -rf ~/Library/Developer/Xcode/Archives/*`
- What it does: Removes archived application builds.
- Safety: Safe if builds have been distributed. Check before deleting.
- Expected impact: Low to medium.

**Step 16 — Clear Tool Caches**
- Command: `rm -rf ~/.cache/uv/ ~/.cache/pip/ ~/.cache/huggingface/`
- What it does: Removes Python package manager caches and AI model metadata.
- Safety: Safe. Tools re-download packages as needed.
- Expected impact: Medium to high. uv cache alone is often 1-2 GB.

**Step 17 — Clear Application Logs**
- Command: `rm -rf ~/Library/Logs/*`
- What it does: Removes all user-level application logs.
- Safety: Safe. Logs are diagnostic data, not critical files.
- Expected impact: Low (typically under 100 MB).

---

## Safety and Transparency

Mac Doctor prioritizes safety over aggressiveness. Every design decision was made with the assumption that the user should never lose data from automated cleanup.

**Every delete operation requires user confirmation.** Phase 3 does not start until the user explicitly says yes. Individual high-impact steps (iOS backups, old downloads) can be declined while others proceed.

**All commands are visible.** Every `rm`, `purge`, `tmutil`, and `mdutil` command appears in the conversation. You can review what will be deleted before it happens.

**No network calls.** Mac Doctor does not send any data to remote servers. It runs local commands only.

**Skippable sections.** Language pack stripping, universal binary thinning, and system log clearing are reported but not executed — they require specialized tools or root access that may not be available.

**Rollback notes.** For irreversible operations (iOS backups, old downloads), Mac Doctor reminds the user to verify before proceeding.

---

## Usage Guide

Mac Doctor is invoked through Codex. You do not install it, configure it, or maintain it.

**Direct invocation:**
```
@mac-doctor
```

**Natural language phrases that trigger the skill:**
- "Clean up my Mac"
- "Scan for junk files and free up space"
- "Run a full malware check on my MacBook"
- "Clear my browsing history and privacy traces"
- "Speed up my computer"
- "Free up disk space"
- "Run maintenance"
- "Check for malware"
- "Clear caches"
- "I need a Mac cleaner"

**Step-by-step conversation flow:**

1. You say something that matches the trigger
2. Mac Doctor runs Phase 1 (scan) — reports a table of 19 categories
3. You review the results
4. Mac Doctor asks: "Found X total. Run cleanup and speed optimization?"
5. If you say yes, Phase 2 (protection) runs — malware and privacy results are shown
6. Mac Doctor asks again before deleting anything
7. If confirmed, Phase 3 (speed) runs all 17 steps and shows before/after

You can stop at any point by saying "stop" or "no".

---

## Comparison: Mac Doctor vs CleanMyMac

| Dimension | CleanMyMac X | Mac Doctor |
|---|---|---|
| **Price** | $11.99/month or $119.88/year | Free |
| **Installation size** | 400 MB download + app | Zero — runs inside Codex |
| **System cache scan** | 15 categories | 19 categories |
| **Deep malware scan** | Proprietary engine | Pattern + signature + SIP audit |
| **Privacy — browsers** | Safari, Chrome, Firefox | Safari, Chrome (size reported, user decides) |
| **Privacy — chat apps** | Not supported | QQ, WeChat, Telegram, Discord, Slack, Skype, Signal, Feishu |
| **Speed optimization** | One-click tweak button | 17-step sequence with before/after measurement |
| **Transparency** | Black box — no command output | Full visibility — every command shown in chat |
| **User confirmation** | Sometimes auto-cleans without asking | Always asks before any delete |
| **Telemetry / data collection** | Yes (usage analytics) | No network calls at all |
| **Source code access** | Closed source | Full SKILL.md is readable and MIT-licensed |
| **Customer support** | Phone + email support | It is a skill, not a company — no support team |
| **App uninstaller** | Included | Not included (use AppCleaner or manual drag) |
| **Duplicate file finder** | Included | Not included (too dangerous to automate) |
| **GPU optimization** | Included | Not needed on Apple Silicon — macOS handles this |
| **Menu bar widget** | Persistent menu bar icon | On-demand invocation only |
| **Windows version** | Yes | No — macOS only |
| **Language pack stripping** | Included | Reported but not executed (needs Monolingual) |

---

## What Mac Doctor Does Not Do

These omissions are intentional.

**Real-time malware protection.** macOS has XProtect for real-time signature-based malware detection. Mac Doctor checks that XProtect is enabled and current. Adding a second real-time scanner inside Codex would be redundant.

**GPU optimization.** On Apple Silicon, macOS manages GPU memory allocation automatically. Third-party GPU cleaners are placebo — they do nothing that the operating system does not already do.

**Duplicate file detection.** Automated duplicate removal has a history of deleting the wrong copy of important files. If you need this, run `fdupes -r ~/Documents` manually and review before deleting.

**App uninstallation.** Mac Doctor reports application sizes but does not uninstall them. Use AppCleaner (free) or drag applications to Trash.

**Menu bar widget.** Mac Doctor runs on demand when you invoke it. It is not a background process and does not sit in your menu bar.

**Windows support.** macOS only. Apple Silicon and Intel.

---

## Before and After — Real Measurements

Actual measurements from this session:

```
Before:  143 GB used on data volume | 1.8 GB RAM free
After:   137 GB used on data volume | 2.1 GB RAM free
Delta:   +6 GB disk space free      | +300 MB RAM free
```

Expected recovery ranges by usage profile:

| Usage Profile | Expected Recovery |
|---|---|
| Light user (browsing + email + documents) | 0.5 - 2 GB |
| Regular user (browsing + Office + media) | 2 - 5 GB |
| Heavy user (browsing + development + design) | 3 - 8 GB |
| iOS/macOS developer (Xcode + simulators) | 5 - 15 GB |
| Never cleaned (years of use) | 8 - 30 GB |

These ranges depend on the number of applications installed, frequency of Xcode use, whether Time Machine snapshots have accumulated, and the size of browser caches.

---

## FAQ

**Is Mac Doctor safe?**
Yes. It runs standard macOS commands (`du`, `rm`, `purge`, `tmutil`, `mdutil`, `dscacheutil`) that are documented in the operating system. It never deletes without asking. You can review every command before it executes.

**Will clearing caches slow down my applications?**
Temporarily. Applications create new caches as they run. The first launch after clearing may be slightly slower as caches are rebuilt. After that, performance returns to normal. The disk space recovered usually outweighs the temporary slowdown.

**Can Mac Doctor delete something important?**
Mac Doctor targets directories that are explicitly designed for temporary data: `Caches/`, `Logs/`, `Trash/`, `DerivedData/`, and files older than defined thresholds. It does not touch system files, application bundles, user documents, or configuration databases. It always asks before deleting.

**Does MacDoctor work on Intel Macs?**
Yes. All commands work identically on Intel and Apple Silicon Macs. The universal binary scan is relevant only on Apple Silicon.

**What happens if I say no?**
Nothing. The skill stops. No files are changed.

**Can I run Phases individually?**
The skill is designed to run the full sequence. If you want a specific operation (for example, only clearing Xcode DerivedData), you can ask Codex directly — the skill is not required for individual commands.

**How often should I run it?**
Monthly is a good cadence for most users. Weekly for heavy Xcode users. For light users, quarterly is sufficient.

---

## Troubleshooting

**Commands fail with "Operation not permitted"**
Some directories require root access. Mac Doctor will note which commands failed due to permissions. You can run them manually: `sudo purge`, `sudo mdutil -E /`.

**Spotlight reindex did not trigger**
The `mdutil -E /` command requires root. If no TTY is available for password input, run it manually: `sudo mdutil -E /`.

**Disk space did not change much**
If your machine was already clean (recently cleaned, light usage), the difference will be small. Check specific categories: Xcode DerivedData, iOS backups, and Time Machine snapshots are the largest potential gains.

**DMG verification is slow**
`hdiutil verify` computes a checksum for the entire disk image. For large DMGs (1+ GB), this takes time. The scan will complete; it just runs in the background.

---

## Technical Details

Mac Doctor is a Codex skill — a Markdown file with YAML frontmatter that defines the skill's name, description, and trigger conditions. The body of the skill contains structured instructions that Codex follows at runtime.

**Trigger mechanism:** Codex matches the user's message against the skill description. If the match score exceeds a threshold, the skill's instructions are loaded into context and Codex follows them step by step.

**Execution environment:** All commands run on the local machine through Codex's shell execution capability. No cloud infrastructure is involved.

**Permission model:** Mac Doctor requests escalated permissions (`require_escalated`) for operations that access paths outside Codex's sandbox. The user must approve these requests. Without approval, commands that need broader filesystem access will fail gracefully.

**Dependencies:** None. All commands use built-in macOS tools: `du` (disk usage), `rm` (delete), `purge` (memory), `tmutil` (Time Machine), `mdutil` (Spotlight), `dscacheutil` (DNS), `codesign` (signature verification), `hdiutil` (disk image utilities), `csrutil` (SIP), `spctl` (Gatekeeper), and `xprotect` (XProtect).

---

## File Structure

```
mac-doctor/
├── SKILL.md      # Skill instructions — Codex reads this at runtime
└── README.md     # This file — project documentation
```

`SKILL.md` contains the full workflow definition: scan commands, protection checks, speed optimization steps, confirmation prompts, and verification logic. `README.md` documents the skill for users browsing the repository.

---

## License

MIT. Use this skill freely. Modify it, share it, adapt it for your own machines.

If someone tries to sell you this skill, they are charging for something MIT-licensed and free.

---

*Codex skill. macOS only. Apple Silicon and Intel. Zero telemetry, zero network calls, zero subscription.*
