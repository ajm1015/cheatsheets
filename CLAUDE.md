# CLAUDE.md — Field Guides Site Expansion

## What Exists

A GitHub Pages site with 6 HTML files, zero dependencies, zero build step:

- `index.html` — Landing page listing all guides (SUN/NeXTSTEP aesthetic)
- `git.html` — Git cheatsheet (hacker dark, 60 commands, 8 tabs)
- `macos-shortcuts.html` — macOS keyboard shortcuts (hacker dark, 210+ entries)
- `macos-field-guide.html` — macOS admin commands (SUN/NeXTSTEP, 100+ commands)
- `windows-shortcuts.html` — Windows keyboard shortcuts (SUN/NeXTSTEP, 285 entries)
- `windows-field-cheatsheet.html` — Windows IT field reference (hacker dark, 33 entries)
- `LICENSE` — GPL v2

Each file is standalone HTML with inline CSS and JS. No shared stylesheets, no frameworks. Two visual themes: **hacker dark** (green-on-black CRT) and **SUN/NeXTSTEP workstation** (warm beige, beveled panels, purple accents). Every file has search (/ to focus, Esc to clear) and category tab navigation.

---

## PHASE 1: Desktop Environment (index.html rewrite)

### The Concept

index.html becomes a **windowed desktop environment**. Guides no longer open as separate pages — they load as draggable, resizable, stackable windows inside the landing page via iframes. The user can tile 1-4 guides side by side, rearrange them, minimize to a taskbar, and snap them to quadrants. Think: CDE/Motif desktop meets browser.

The individual guide HTML files remain untouched and standalone. They still work as direct URLs. index.html simply embeds them.

### Window Manager Behavior

**Opening a guide:**
- Click a guide card on the desktop -> guide opens in a new window (iframe) inside the page
- Card remains on the desktop but dims to show it's open
- Window appears centered with a slight random offset (so stacking is visible)
- Default window size: 65vw x 70vh. Minimum: 400px x 300px.

**Window chrome (per window):**
- Title bar matching the file's theme:
  - Hacker dark files (`macos-shortcuts`, `windows-field-cheatsheet`, `git`): dark title bar, green text, green-tinted buttons
  - Workstation files (`windows-shortcuts`, `macos-field-guide`): purple gradient title bar, beveled buttons
- Title bar buttons: Close (X), Minimize (--), Maximize ([])
- Title bar is the drag handle
- Double-click title bar: toggle maximize

**Dragging:**
- Drag by title bar only
- Use CSS `transform: translate3d()` for all movement — forces GPU compositing
- Constrain to viewport bounds (leave at least 50px visible)
- While dragging: opacity 0.85, subtle drop shadow expansion
- On drop: snap to full opacity, 120ms ease-out transition

**Resizing:**
- 8px resize handles on all edges and corners (custom DOM elements, not CSS resize)
- Minimum dimensions enforced: 400x300
- Use `will-change: width, height` during resize, remove after

**Z-order / Focus:**
- Click window or title bar to bring to front
- Focused window: brighter border or subtle glow (theme-appropriate)
- z-index via incrementing counter on focus

**Snap Tiling (keyboard + drag):**
- Drag to left edge (within 20px): snap left 50%
- Drag to right edge: snap right 50%
- Drag to top edge: maximize
- Drag to corner: snap to quadrant (25% viewport)
- Keyboard (when window focused):
  - `Ctrl+ArrowLeft` — snap left half
  - `Ctrl+ArrowRight` — snap right half
  - `Ctrl+ArrowUp` — maximize
  - `Ctrl+ArrowDown` — restore/minimize
  - `Ctrl+Shift+ArrowLeft` — top-left quadrant
  - `Ctrl+Shift+ArrowRight` — top-right quadrant
  - `Ctrl+Alt+ArrowLeft` — bottom-left quadrant
  - `Ctrl+Alt+ArrowRight` — bottom-right quadrant
- Snap zones: translucent purple highlight overlay while dragging near edges
- Snap transitions: `transition: all 200ms cubic-bezier(0.22, 1, 0.36, 1)`

**Minimize & Taskbar:**
- Fixed taskbar at viewport bottom, 40px height, themed to desktop
- Minimized windows become taskbar buttons (icon + short name)
- Click button to restore. Right-click for "Close".
- Clock on right side: `HH:MM`, updates every minute (aesthetic only)

**Maximize:**
- Fills viewport minus taskbar height
- Title bar stays. Double-click or button to restore.
- Remembers pre-maximize position/size

**Close:**
- Removes iframe, removes taskbar entry, un-dims desktop card

### Desktop Surface

**Cards:**
- Existing cards become launchers (click opens window, not navigates)
- Drag-and-drop to reorder. Persists in `localStorage` (`fieldguides-card-order`)
- While reordering: other cards animate apart to show drop zone
- Open guides get a "running" indicator dot on their card icon

**Right-click context menu:**
- Right-click empty desktop:
  - "Tile All Open" — even grid arrangement
  - "Cascade" — stacked with 30px offset
  - "Minimize All"
  - "Close All"
  - "Reset Layout" — clears localStorage, default card order
- SUN/NeXTSTEP styled (beveled panel, shadow)
- Dismiss on click outside or Esc

### Hardware-Accelerated Effects

All animations use properties that trigger GPU compositing. Target: 60fps.

**GPU-composited properties only:**
- `transform: translate3d()` for window positioning during drag
- `will-change: transform` during interaction (remove 500ms after interaction ends)
- `opacity` for focus/blur transitions
- `filter: blur()` sparingly and gated

**Animations:**

1. **Window open:** scale 0.92->1.0 + opacity 0->1, 180ms ease-out. Transform origin: launching card's center.
2. **Window close:** scale 1.0->0.95 + opacity 1->0, 120ms ease-in. Remove iframe after animation.
3. **Minimize:** shrink toward taskbar button position via `transform: scale(0.15) translate3d(targetX, targetY, 0)`, 250ms. Fade in last 80ms.
4. **Restore from minimize:** reverse of minimize animation.
5. **Desktop blur (optional, default OFF):** focused window causes desktop cards to get `filter: blur(2px)` + `opacity: 0.7`. Toggle via context menu "Enable backdrop blur". Stored in `localStorage` (`fieldguides-blur-enabled`).
6. **Snap zone preview:** overlay div, `background: rgba(123,97,255,0.15)`, `border: 2px solid rgba(123,97,255,0.4)`, 100ms fade-in.
7. **Focus ring:** theme-appropriate `box-shadow`, 150ms transition. Workstation: `0 0 0 2px rgba(123,97,255,0.5), 0 8px 32px rgba(0,0,0,0.3)`. Dark: `0 0 0 1px rgba(60,255,168,0.4), 0 8px 32px rgba(0,0,0,0.5)`.
8. **Taskbar hover:** scale 1.05, 100ms transition.
9. **Card reorder drag:** `transform: scale(1.03)`, shadow expansion, opacity 0.9. Others slide with translateY.

**Performance rules:**
- Never animate `width`, `height`, `top`, `left`, `margin`, `padding`
- `transform` and `opacity` only for animations
- `will-change` applied only during interaction, removed 500ms after
- `requestAnimationFrame` for drag updates (not raw mousemove)
- Iframes are self-contained rendering contexts -- parent transforms don't affect them

### Layout Persistence

`localStorage` keys (all prefixed `fieldguides-`):
- `fieldguides-card-order` — array of filenames in display order
- `fieldguides-open-windows` — array: `{ file, x, y, w, h, minimized, maximized }`
- `fieldguides-blur-enabled` — boolean

On load: restore card order and reopen saved windows. Clamp to visible area if viewport changed.

### Responsive Behavior

- **Below 768px:** windowing disabled entirely. Cards link directly to guide files (current behavior). No taskbar.
- **768px-1200px:** max 2 windows. Snap supports halves only, no quadrants.
- **Above 1200px:** full WM, up to 6 windows.

---

## PHASE 2: Command Palette (all guide files)

A global command palette (Ctrl+K / Cmd+K) in every guide file. Not in index.html (desktop has its context menu).

- Fuzzy search across all commands/shortcuts in current guide
- Results: command text, description, category badge
- Enter: scroll to result + 500ms highlight flash
- Input starting with `$` or `>`: "copy to clipboard" action instead of scroll
- Mode toggle pills: `ALL | PS | BASH | ZSH` (filter by shell type where tags exist)
- Matches host file's visual theme
- Dismiss: Esc or click outside
- Max 15 visible results, scrollable
- Pure vanilla JS per file

### Fuzzy Search Scoring

```
score(query, text):
  exact substring: 100
  all chars in order (fuzzy): 50 + (consecutive_bonus * 5)
  word-boundary matches (+10 each)
  shorter text bonus: +5
  minimum threshold: 20
```

Sort descending by score.

---

## PHASE 3: Deep Poweruser Content Additions

### `windows-field-cheatsheet.html`

**New section: "Endpoint Forensics & Deep Diagnostics"**
- `Get-WmiObject Win32_StartupCommand` — all startup entries
- `wevtutil qe Security /c:25 /f:text /rd:true` — last 25 security events
- `Get-CimInstance MSFT_NetAdapter -Namespace root/StandardCimv2 | select Name, InterfaceDescription, DriverVersion, DriverDate` — NIC driver audit
- `Get-Process | Sort-Object WorkingSet64 -Descending | Select -First 15 Name, @{N='MB';E={[math]::Round($_.WorkingSet64/1MB)}}` — top 15 memory hogs
- `cipher /w:C:\` — secure wipe free space
- `sfc /scannow` + `DISM /Online /Cleanup-Image /RestoreHealth` — system file repair chain
- `Get-NetTCPConnection -State Established | Select LocalAddress, LocalPort, RemoteAddress, RemotePort, OwningProcess | Sort RemoteAddress` — active TCP connections
- `Get-ScheduledTask | Where-Object {$_.Principal.RunLevel -eq 'Highest'} | Select TaskName, TaskPath, State` — elevated scheduled tasks
- `bcdedit /enum all` — boot configuration dump
- `reagentc /info` — WinRE status
- `vssadmin list shadows` — Volume Shadow Copy inventory
- `wmic path Win32_PnPSignedDriver get DeviceName, DriverVersion, Manufacturer /format:table` — signed driver list

**New section: "Remote & Fleet"**
- `Invoke-Command -ComputerName $hosts -ScriptBlock { Get-HotFix | Sort InstalledOn -Desc | Select -First 5 }` — fleet patch check
- `Test-NetConnection -ComputerName $target -Port 5985 -InformationLevel Detailed` — WinRM test
- `Get-ADComputer -Filter * -Properties LastLogonDate | Where { $_.LastLogonDate -lt (Get-Date).AddDays(-90) } | Select Name, LastLogonDate` — stale AD computers (RSAT required)
- `Invoke-Command -ComputerName $target -ScriptBlock { gpresult /r }` — remote GP result

### `macos-field-guide.html`

**New section: "Deep System Internals"**
- `sudo fs_usage -f network` — real-time FS/network tracing
- `sudo dtrace -n 'syscall::open*:entry { printf("%s %s", execname, copyinstr(arg0)); }'` — trace file opens (SIP off)
- `sudo powermetrics --samplers cpu_power,gpu_power -i 5000 -n 1` — CPU/GPU power snapshot
- `sudo log collect --last 1h --output ~/Desktop/syslog.logarchive` — compressed log archive
- `sudo sysctl -a | grep -i kern.securelevel` — kernel security level
- `sudo kextstat | grep -v com.apple` — third-party kexts
- `ioreg -r -c IOPlatformExpertDevice -d 2 | grep -i board-id` — board-id
- `sudo /usr/libexec/mdmclient dep nag` — force DEP nag
- `sudo profiles validate -type enrollment` — validate enrollment profile
- `networkQuality -v` — network quality test (Monterey+)
- `sudo pmset -g assertions` — power management assertions
- `sudo openssl s_client -connect host:443 -servername host 2>/dev/null | openssl x509 -noout -dates -subject` — remote cert check
- `sudo lsof -i -P -n | grep LISTEN` — listening ports with PIDs
- `sudo dsconfigad -show` — AD binding status
- `sudo fdesetup validaterecovery` — test FileVault recovery key
- `sudo nvram -xp` — dump NVRAM (XML plist)

### `macos-shortcuts.html`

**New category: "HIDDEN SYSTEM GESTURES"**
- Option+click Wi-Fi: detailed info (BSSID, RSSI, channel, PHY)
- Option+click Bluetooth: BT details
- Option+click Sound: direct device selection
- Ctrl+Shift+Eject/Power: sleep displays
- Cmd+Option+Eject/Power: sleep Mac

**New category: "TERMINAL POWER MOVES"**
- Cmd+Shift+. in Open/Save dialog: toggle hidden files
- Drag file into Terminal: paste full path
- `open .`: Finder at cwd
- `pbcopy`/`pbpaste`: clipboard pipe
- `caffeinate -t 3600`: prevent sleep 1hr
- `mdfind -onlyin ~/Documents "query"`: scoped Spotlight
- `textutil -convert html document.docx`: format conversion

### `windows-shortcuts.html`

**New category: "HIDDEN / UNDOCUMENTED"**
- Win+Alt+K: mic mute toggle (Win 11 22H2+)
- Shift+right-click file: extended context / "Copy as path"
- Shift+right-click folder bg: "Open Terminal here"
- Ctrl+Shift+N in Save dialog: new folder
- Alt+Space then S: keyboard resize
- Ctrl+Shift+Click taskbar: launch as admin

### `git.html`

**New section: "Advanced / Esoteric"**
- `git shortlog -sn --no-merges` — contributor leaderboard
- `git diff --word-diff` — word-level diff
- `git stash branch <name> stash@{0}` — branch from stash
- `git log -p -S "function_name" --all` — code archeology
- `git rev-list --count HEAD` — total commit count
- `git for-each-ref --sort=-committerdate refs/heads/ --format='%(refname:short) %(committerdate:relative)'` — branches by recency
- `git commit --fixup=<sha>` + `git rebase -i --autosquash` — fixup workflow
- `git rerere` — reuse recorded conflict resolution
- `git maintenance start` — background maintenance
- `git sparse-checkout set <dir>` — partial checkout
- `git range-diff main~3..main feat~3..feat` — compare commit ranges

---

## PHASE 4: index.html Sync

**Every time a guide file is modified**, update index.html:

- Exact entry/shortcut/command counts in card `<span class="mv">` elements
- Section counts per card
- Terminal `ls -la` block: filenames and actual file sizes (check after edit, round to K)
- Footer total file count

---

## Constraints — What Claude Would Get Wrong

### Architecture
- **No shared CSS/JS.** Every HTML file is standalone. Do not refactor.
- **No build tools, npm, or frameworks.** Pure HTML/CSS/JS.
- **No external JS libraries** for the WM. No interact.js, jQuery UI, etc. Vanilla only.
- **iframes, not DOM injection.** `<iframe src="git.html">`. Do not parse and inject guide HTML -- breaks their self-contained JS.

### Visual
- **Do not change existing guide themes.** Dark stays dark, workstation stays workstation.
- **ASCII only in code blocks.** No em dashes, smart quotes, multi-byte chars. `--` not `--`. Straight quotes.
- **Window chrome matches iframe theme.** Dark guides get dark chrome, workstation guides get beveled chrome. Map by filename.

### Performance
- **Never animate layout properties.** `transform` and `opacity` only.
- **`will-change` is temporary.** Apply during interaction, remove 500ms after.
- **rAF for drag.** Not raw mousemove.
- **Max 6 iframes.** Below 768px: zero iframes.

### Functionality
- **Existing / search in guides is untouched.** Palette is Ctrl+K -- separate.
- **`data-keywords` on every new entry.** Required for search.
- **Tags on destructive/elevated commands.** DESTRUCTIVE, SUDO, ADMIN, WIN11 etc.
- **localStorage keys prefixed `fieldguides-`.** No collisions.
- **Below 768px: no WM.** Cards link directly. Non-negotiable.

---

## Verification

1. Each guide opens standalone -- search, tabs, copy buttons work
2. index.html: cards open windows as iframes
3. Drag, resize, minimize, maximize, close all work
4. Snap to halves/quadrants via drag and keyboard
5. 4 windows tiled simultaneously, no frame drops
6. Taskbar: minimize/restore cycle works
7. Context menu: Tile All, Cascade, Minimize All, Close All, Reset
8. Card drag reorder persists across reload
9. Command palette (Ctrl+K) works in iframes and standalone
10. Animations at 60fps (open, close, minimize, restore)
11. Below 768px: direct links, no windowing
12. index.html counts match actual guide content
13. No console errors
14. New `<pre>` blocks use existing syntax classes (`.cmd`, `.flag`, `.string`/`.str`, `.comment`)

---

## Execution Order

1. Rewrite `index.html` — full desktop environment (WM, taskbar, snap, drag, resize, context menu, card reorder, persistence, animations)
2. Add command palette to each of the 5 guide files
3. Add new sections/entries to `windows-field-cheatsheet.html`
4. Add new sections/entries to `macos-field-guide.html`
5. Add new entries to `macos-shortcuts.html`
6. Add new entries to `windows-shortcuts.html`
7. Add new section/entries to `git.html`
8. Final pass: update `index.html` counts, file sizes, verify
