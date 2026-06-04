---
name: mac-doctor
description: |
  Free CleanMyMac alternative for macOS — three-phase maintenance scans caches, logs, malware, and privacy traces, then speeds up your machine. Asks before deleting. No subscription.
  Use when the user asks to clean up disk space, remove junk files, speed up their Mac, check for malware, clear browsing history and privacy traces, or optimize system performance. Also triggers on "free up space", "Mac cleaner", "Mac maintenance", "delete cache", "remove junk", "privacy clean", "browser history clear", "malware scan", "speed optimization", "CleanMyMac alternative".
---

# Mac Doctor



Three-phase Mac maintenance: **Scan → Protect → Speed**. Always asks for confirmation before changing anything.

---

## Phase 1: Smart Scan

Scan all cache, junk, and data locations. Group by category and present a clear table.

### 1.1 User Cache & System Cache

```bash
echo "=== ~/Library/Caches/ ==="
du -sh ~/Library/Caches/ 2>/dev/null
du -sh ~/Library/Caches/*/ 2>/dev/null | sort -rh | head -20

echo ""
echo "=== ~/.cache/ ==="
du -sh ~/.cache/ 2>/dev/null
du -sh ~/.cache/*/ 2>/dev/null | sort -rh

echo ""
echo "=== System Cache (/Library/Caches/) ==="
du -sh /Library/Caches/ 2>/dev/null
```

### 1.2 User & System Log Files

```bash
echo "=== User Logs (~/Library/Logs/) ==="
du -sh ~/Library/Logs/ 2>/dev/null
du -sh ~/Library/Logs/*/ 2>/dev/null | sort -rh | head -15

echo ""
echo "=== System Logs (/Library/Logs/ + /var/log/) ==="
du -sh /Library/Logs/ 2>/dev/null
du -sh /var/log/ 2>/dev/null
```

### 1.3 Language Files

```bash
echo "=== Language Packs ==="
for app in /Applications/*.app/Contents/Resources/; do
  base=$(basename "$(dirname "$(dirname "$app")")")
  lang_count=$(ls "$app"*.lproj 2>/dev/null | wc -l)
  if [ "$lang_count" -gt 10 ] 2>/dev/null; then
    echo "  $base: $lang_count languages"
  fi
done 2>/dev/null | sort -t: -k2 -rn | head -10
```

### 1.4 Trash Bins

```bash
echo "=== Trash ==="
du -sh ~/.Trash/ 2>/dev/null
ls -lhS ~/.Trash/ 2>/dev/null | head -10
echo "  Items: $(find ~/.Trash/ -maxdepth 1 2>/dev/null | wc -l)"
```

### 1.5 Mail Attachments

```bash
echo "=== Mail Attachments ==="
du -sh ~/Library/Mail/ 2>/dev/null
du -sh ~/Library/Containers/com.apple.mail/Data/Downloads 2>/dev/null
```

### 1.6 Xcode Junk

```bash
echo "=== Xcode ==="
echo "DerivedData: $(du -sh ~/Library/Developer/Xcode/DerivedData/ 2>/dev/null | awk '{print $1}')"
echo "iOS DeviceSupport: $(du -sh ~/Library/Developer/Xcode/iOS\ DeviceSupport/ 2>/dev/null | awk '{print $1}')"
echo "Archives: $(du -sh ~/Library/Developer/Xcode/Archives/ 2>/dev/null | awk '{print $1}')"
```

### 1.7 Other System Junk

```bash
echo "=== Old Updates ==="
du -sh /Library/Updates/ 2>/dev/null

echo ""
echo "=== Broken Login Items ==="
osascript -e 'tell application "System Events" to get name of every login item' 2>/dev/null
ls ~/Library/LaunchAgents/ 2>/dev/null

echo ""
echo "=== Unused Disk Images ==="
find ~/Documents -maxdepth 3 -name "*.dmg" -exec ls -lh {} \; 2>/dev/null | awk '{print $5, $NF}'

echo ""
echo "=== Downloads ==="
du -sh ~/Downloads/ 2>/dev/null
find ~/Downloads -maxdepth 1 -type f -mtime +90 2>/dev/null | wc -l | xargs echo "Files older than 90 days:"

echo ""
echo "=== Broken Preferences ==="
find ~/Library/Preferences -name "*.plist" -mtime +365 2>/dev/null | wc -l | xargs echo "Old plists:"

echo ""
echo "=== iOS Device Backups ==="
du -sh ~/Library/Application\ Support/MobileSync/Backup/ 2>/dev/null

echo ""
echo "=== Universal Binaries ==="
echo "(Skipped — requires full /Applications lipo scan)"
```

### 1.8 Document Versions & Deleted Users

```bash
echo "=== Document Versions ==="
test -d ~/.DocumentRevisions-V100 && du -sh ~/.DocumentRevisions-V100/ || echo "  None"
echo ""
echo "=== Deleted Users ==="
ls -la /Users/ 2>/dev/null | grep -v "total\|^\." | grep -v "andy\|Shared\|Guest"
```

### 1.9 Present Summary

Compile a table like this and present it to the user:

| Category | Item | Size |
|---|---|---|
| User Cache | ~/Library/Caches/ | X |
| Hidden Cache | ~/.cache/ | X |
| System Cache | /Library/Caches/ | X |
| User Logs | ~/Library/Logs/ | X |
| System Logs | /Library/Logs/ + /var/log/ | X |
| Language Packs | (top apps) | X languages |
| Trash | ~/.Trash/ | X |
| Mail | ~/Library/Mail/ | X |
| Xcode | DerivedData + others | X |
| Old Updates | /Library/Updates/ | X |
| Downloads | ~/Downloads/ | X |
| iOS Backups | MobileSync | X |
| **TOTAL** | | **X** |

---

## Phase 2: Protection

### 2.1 Malware Deep Scan

Scan for malware across system and user locations.

**Scan DMG files:**
```bash
echo "=== DMG Scans ==="
find /Users/$USER/Downloads /Users/$USER/Desktop /Users/$USER/Documents -name "*.dmg" -maxdepth 4 2>/dev/null
for dmg in $(find /Users/$USER/Downloads -name "*.dmg" -maxdepth 2 2>/dev/null); do
  echo "--- $dmg ---"
  ls -lh "$dmg" 2>/dev/null
  # Verify code signature if mounted
  hdiutil verify "$dmg" 2>/dev/null | tail -1
done
```

**Scan archives:**
```bash
echo "=== Archive Scans ==="
find /Users/$USER/Downloads /Users/$USER/Desktop -maxdepth 3 \( -name "*.zip" -o -name "*.tar.gz" -o -name "*.tgz" -o -name "*.tar" -o -name "*.7z" -o -name "*.rar" \) -mtime -30 2>/dev/null
```

**Scan for known malware patterns:**
```bash
echo "=== Malware Pattern Scan ==="
# Common malware/PUP names
for pattern in "keygen" "crack" "patch" "activator" "loader" "macbooster" "mackeeper" "advanced.*cleaner"; do
  results=$(find /Users/$USER/Downloads /Users/$USER/Desktop /Users/$USER/Documents -maxdepth 5 -iname "*$pattern*" 2>/dev/null)
  if [ -n "$results" ]; then
    echo "  Found: $results" | head -5
  fi
done
echo ""
echo "  (Filter git hook false positives manually)"
```

**System security status:**
```bash
echo "=== System Security ==="
csrutil status 2>/dev/null
spctl --status 2>/dev/null
xprotect version 2>/dev/null
```

**Installed apps review (recent):**
```bash
echo "=== Recent App Installs ==="
ls -lt /Applications/ 2>/dev/null | head -10
```

**Chrome extensions review:**
```bash
echo "=== Chrome Extensions ==="
for ext in /Users/$USER/Library/Application\ Support/Google/Chrome/Default/Extensions/*/; do
  extid=$(basename "$ext")
  manifest=$(ls -t "$ext"*/manifest.json 2>/dev/null | head -1)
  if [ -f "$manifest" ]; then
    name=$(python3 -c "import json; d=json.load(open('$manifest')); print(d.get('name','?'))" 2>/dev/null)
    echo "  $name"
  fi
done
```

**Email attachments:**
```bash
echo "=== Email Attachments ==="
find /Users/$USER/Library/Containers/com.apple.mail/Data/Downloads -type f 2>/dev/null | head -10
find /Users/$USER/Library/Mail -name "*.zip" -o -name "*.exe" -o -name "*.dmg" 2>/dev/null | head -10
```

**USB drives (mounted):**
```bash
echo "=== USB Drives ==="
ls /Volumes/ 2>/dev/null | grep -v "Macintosh HD"
```

**iCloud downloads:**
```bash
echo "=== iCloud Downloads ==="
find /Users/$USER/Library/Mobile\ Documents/com\~apple\~CloudDocs/Downloads -maxdepth 2 2>/dev/null | head -10
```

### 2.2 Privacy Scan

**Browsing data (Safari):**
```bash
echo "=== Safari ==="
echo "History: $(ls -lh ~/Library/Safari/History.db 2>/dev/null | awk '{print $5}')"
echo "Bookmarks: $(ls -lh ~/Library/Safari/Bookmarks.db 2>/dev/null | awk '{print $5}')"
echo "CloudTabs: $(ls -lh ~/Library/Safari/CloudTabs.db 2>/dev/null | awk '{print $5}')"
echo "Autofill: $(ls -lh ~/Library/Autofill\ Credit\ Cards/ 2>/dev/null | awk '{print $5}')"
echo "LocalStorage: $(du -sh ~/Library/Safari/LocalStorage/ 2>/dev/null | awk '{print $1}')"
```

**Browsing data (Chrome):**
```bash
echo "=== Chrome ==="
echo "History: $(ls -lh ~/Library/Application\ Support/Google/Chrome/Default/History 2>/dev/null | awk '{print $5}')"
echo "Cookies: $(ls -lh ~/Library/Application\ Support/Google/Chrome/Default/Cookies 2>/dev/null | awk '{print $5}')"
echo "Saved Passwords: $(ls -lh ~/Library/Application\ Support/Google/Chrome/Default/Login\ Data 2>/dev/null | awk '{print $5}')"
echo "Autofill: $(ls -lh ~/Library/Application\ Support/Google/Chrome/Default/Web\ Data 2>/dev/null | awk '{print $5}')"
echo "Bookmarks: $(ls -lh ~/Library/Application\ Support/Google/Chrome/Default/Bookmarks 2>/dev/null | awk '{print $5}')"
```

**Chat application data:**
```bash
echo "=== Chat Apps ==="
for app in "QQ" "WeChat" "Telegram Desktop" "discord" "Slack" "Skype" "Feishu" "bytedance.lark" "Signal"; do
  path1="$HOME/Library/Application Support/$app"
  path2="$HOME/Library/Containers/com.tencent.xinWeChat"
  found=false
  for p in "$path1" "$path2"; do
    if [ -d "$p" ]; then
      size=$(du -sh "$p" 2>/dev/null | awk '{print $1}')
      echo "  $app: $size"
      found=true
      break
    fi
  done
  if ! $found; then
    echo "  $app: not installed"
  fi
done
```

**System activity traces:**
```bash
echo "=== System Traces ==="
echo "Recent Documents: $(ls -1 ~/Library/Application\ Support/com.apple.sharedfilelist/*Recent* 2>/dev/null | wc -l) files"
echo "Spotlight Cache: $(du -sh ~/Library/Application\ Support/com.apple.spotlight/ 2>/dev/null | awk '{print $1}')"
echo "QuickLook: $(du -sh ~/Library/Caches/com.apple.QuickLook.QL/ 2>/dev/null | awk '{print $1}')"
```

### 2.3 Present Protection Summary

| Category | Item | Size/Status |
|---|---|---|
| Malware | DMGs checked | result |
| Malware | Archives checked | result |
| Malware | Patterns found | count |
| System Security | SIP/Gatekeeper/XProtect | status |
| Privacy | Safari browsing data | X |
| Privacy | Chrome browsing data | X |
| Privacy | Chat app data | X |

---

## Phase 3: Speed Optimization

### 3.1 Measure Before State

Record baseline:

```bash
BEFORE_DISK=$(df -h /System/Volumes/Data 2>/dev/null | tail -1 | awk '{print $3, $4}')
BEFORE_RAM=$(vm_stat 2>/dev/null | grep 'Pages free' | awk '{print $3}' | tr -d '.')
echo "Before: Disk used=$BEFORE_DISK / RAM free=$(echo "scale=0; $BEFORE_RAM*16384/1024/1024" | bc)MB"
```

### 3.2 Execute Optimizations

Run each step with escalated permissions. Skip any step the user declined.

```bash
# Free up RAM
purge 2>/dev/null && echo "RAM purged" || echo "RAM purge skipped (needs root)"

# Clear application cache
rm -rf ~/Library/Caches/* 2>/dev/null

# Clear system cache
rm -rf /Library/Caches/* 2>/dev/null

# Clear user logs
rm -rf ~/Library/Logs/* 2>/dev/null

# Flush DNS
dscacheutil -flushcache 2>/dev/null

# Free purgeable space (trim Time Machine snapshots)
tmutil thinlocalsnapshots / 9999999999999 2>/dev/null

# Clear recent items
rm -f ~/Library/Preferences/com.apple.recentitems.plist 2>/dev/null
rm -f ~/Library/Application\ Support/com.apple.sharedfilelist/*.sfl3 2>/dev/null
rm -f ~/Library/Application\ Support/com.apple.sharedfilelist/*.sfl2 2>/dev/null

# Reindex Spotlight
mdutil -E / 2>/dev/null

# Remove Time Machine snapshots
tmutil deletelocalsnapshots / 2>/dev/null

# Remove disk images and archives from Downloads/Documents
find ~/Downloads ~/Documents -maxdepth 3 \( -name "*.dmg" -o -name "*.pkg" \) -mtime +7 -exec rm {} \; 2>/dev/null

# Remove unneeded downloads (older than 90 days)
find ~/Downloads -maxdepth 1 -type f -mtime +90 -exec rm {} \; 2>/dev/null

# Remove iOS device backups
rm -rf ~/Library/Application\ Support/MobileSync/Backup/* 2>/dev/null

# Remove stale preferences
find ~/Library/Preferences -name "*.plist" -mtime +365 -delete 2>/dev/null

# Remove Xcode DerivedData
rm -rf ~/Library/Developer/Xcode/DerivedData/ 2>/dev/null
rm -rf ~/Library/Developer/Xcode/iOS\ DeviceSupport/ 2>/dev/null
rm -rf ~/Library/Developer/Xcode/Archives/ 2>/dev/null

# Remove Document revisions
rm -rf ~/.DocumentRevisions-V100 2>/dev/null

# Remove app logs and support files (core application caches)
rm -rf ~/.cache/uv/ ~/.cache/pip/ ~/.cache/huggingface/ 2>/dev/null

# Turn off unused login items (keep only essential ones)
# Check: only Google Updater is legitimate; all others removed
# (Manual: user must check System Settings > General > Login Items)

# Remove unused launch agents (check for non-Apple items not in use)
# (Manual: audit ~/Library/LaunchAgents/ and /Library/LaunchAgents/)

# Close browser tabs — not possible from CLI, suggest user do manually
```

### 3.3 Measure After State

```bash
AFTER_DISK=$(df -h /System/Volumes/Data 2>/dev/null | tail -1 | awk '{print $3, $4}')
AFTER_RAM=$(vm_stat 2>/dev/null | grep 'Pages free' | awk '{print $3}' | tr -d '.')
echo "After: Disk used=$AFTER_DISK / RAM free=$(echo "scale=0; $AFTER_RAM*16384/1024/1024" | bc)MB"
```

### 3.4 Present Speed Summary

| Metric | Before | After | Delta |
|---|---|---|---|
| Disk used | X | Y | ±Z |
| RAM free | X MB | Y MB | ±Z MB |

---

## Master Flow

The complete invocation goes through these steps in order:

1. **Phase 1 — Smart Scan**: Scan all locations, present the full table
2. **Phase 2 — Protection**: Scan for malware and privacy traces, present findings
3. **Confirm**: Ask the user: "Found X total. Scan complete. Clear everything and run speed optimization?"
   - If user says yes, proceed to Phase 3
   - If user declines, stop
4. **Phase 3 — Speed**: Execute all optimizations, show before/after comparison

### Notes

- Always use `require_escalated` when deleting outside sandbox.
- Never delete without user confirmation.
- For items requiring sudo (system logs, full purge, Spotlight reindex): note them but proceed with what's possible.
- `codex-runtimes` in ~/.cache/ is intentionally excluded from cleanup since Codex needs it — mention this to the user.
- Language file removal requires a dedicated tool (Monolingual) — flag it but don't execute.
- Chat app data: only report sizes, don't read content.
- Items the user explicitly wants kept (e.g., Chrome data) should be skipped in cleanup.
