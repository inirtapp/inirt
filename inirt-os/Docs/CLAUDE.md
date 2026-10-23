# inirt-os

inirt is a Linux-kernel-based operating system. The AI writes all of the code. Goals, in order of importance: **speed, usability, security**.

Items marked **(DEFAULT)** are the starting values the user has accepted. Follow them, and change them only when the user says so.

---

## 1. Targets (every change must work on all three)
1. **UTM** (emulated/virtualized) on the developer's Mac.
2. **Bare metal on a MacBook Pro** (MacBookPro14,2), installed in its own partition.
3. **Any Surface Pro or other Intel-only machine from 2020 onward.**

If something is hardware-specific, isolate it, document it in `Docs/`, and never let it break the other targets.

## 2. Development machine (keep everything lightweight)
- MacBook Pro, Intel Core i5 (2 cores), **8 GB RAM**, x86_64. macOS tops out at Ventura.
- Prefer incremental builds. Never run full rebuilds, heavy emulation, or large downloads unless asked.
- Build for x86_64 only.
- Because the hardware is modest, **speed beats visual effects**: no effect may be on by default if it costs noticeable performance.

## 3. Folder layout (DEFAULT purposes)
- `Work/` - source code, one subfolder per component
- `Tools/` - build scripts and helper tooling
- `Docs/` - documentation, decisions, test checklists
- `Reference/` - upstream docs, specs, notes (read-only reference)
- `Releases/` - finished ISOs (see hard rules)
- `PROGRESS.md` - running log of what is done, in progress, and next (project root; see section 13)

## 4. Hard rules (never break these)
1. **Hardened malloc is always on**, system-wide. Never disable it, add opt-outs, or make it conditional.
2. **Every ISO is numbered and never replaced**: `inirt-0001.iso`, `inirt-0002.iso`, and so on, each with a checksum and a signature file, written to `Releases/`. Never overwrite or delete anything in `Releases/`.
3. **Surface builds ship a Secure Boot signing package** (shim, mokutil, sbctl) plus a guided step so the user can enroll their own key and sign after install.
4. **Vetted crypto only** (libsodium, Argon2, standard system libraries). Never invent encryption or hashing.
5. **Make the smallest change that solves the task.** No unrelated refactors. No new dependencies without asking.
6. **Ask before** deleting files, installing packages, or running destructive commands. If requirements are unclear, ask before writing code.
7. After every change, say what changed and how to verify it (in UTM first).
8. **Use only the names in the rename map** (section 11). For any other name, brand text, or slogan, ask the user instead of inventing one.
9. **Record progress in `PROGRESS.md`** after every task, and read it at the start of every session (see section 13).

## 5. System design
- **Kernel/drivers:** Linux kernel with all drivers included. Prefer upstream/mainline drivers and never remove existing ones. Known extras: Broadcom Wi-Fi, keyboard/trackpad, and Touch Bar on the MacBook Pro; linux-surface patches for Surface touch, pen, and cameras.
- **SIP-style protection:** read-only root filesystem, integrity-verified (dm-verity or an immutable/atomic design), with a protected area that even admin cannot change without an explicit unlock step.
- **Terminal:** a fully functional Linux terminal with standard GNU tools and bash/zsh/fish, regardless of how the desktop looks.
- **Disk encryption:** LUKS.
- **Update system:** keep it simple. An update is a special update zip/package that targets only the compositor and everything above it (`inirt-comp`, `inirt-shell`, `inirt-settings`, and the inirt apps). It never touches the kernel, drivers, or the read-only base.
  - Applying an update **requires the admin password**.
  - The package must be signed and verified before anything is applied.
  - Applied atomically: all or nothing, never a half-applied state.
  - The previous version is always kept.
  - **Rollback must work even if the OS won't start**, through a recovery/boot-menu option that switches back to the previous version without needing the desktop.

## 6. Accounts and passwords
- Only ever **one user account**, but **two separate passwords**: a user password and an admin password.
- Admin tasks need the admin password (sudo with `rootpw` and polkit rules).
- The user password can only be changed if the admin password is entered, **unless no admin password is set**.
- The installer must strongly encourage setting an admin password, since without one this protection does not exist.
- Every admin prompt uses the same dialog (see section 9.8).

## 7. Components (all written by the AI)
| Component | Path | Tech |
|---|---|---|
| Compositor | `Work/inirt-comp` | Rust, Smithay |
| Shell (dock, launcher, lock screen, wallpapers) | `Work/inirt-shell` | Qt/QML, layer-shell client |
| Settings app | `Work/inirt-settings` | Qt/QML |
| Browser | `Work/inirt-browser` | WebKitGTK, custom UI |
| Other inirt-made apps | `Work/<app-name>` | Qt/QML unless stated |

inirt-made apps: installer, step-by-step helper, composition/app maker, password manager, system usage monitor, calculator, clock + weather, app store, general game app.

---

## 8. Desktop behavior
- **Super+Q quits the focused app, immediately.** Handled inside the compositor so it works before any app sees the key. It sends a graceful close; if the app has not exited in about 3 seconds, show a Force quit prompt and kill it.
- **Dark and light mode**, switchable from settings, applied system-wide and instantly (no restart).
- **Touch gestures** must work for Surface Pro.
- **Fractional scaling** from 1x to 2x for the MacBook Pro display and high-DPI Surface screens.

### Keyboard shortcuts (DEFAULT, except Super+Q)

All shortcuts can be changed in Settings > Keyboard Shortcuts (section 9.11). Super+Q is the required default for quitting the focused app.

| Shortcut | Action |
|---|---|
| Super+Q | Quit focused app (required) |
| Super (tap) | Open/close launcher |
| Super+Tab | Switch between open apps |
| Super+L | Lock screen |
| Super+Arrow keys | Snap window left/right, maximize, restore |
| Super+Enter | Open terminal |
| Super+, | Open settings |

---

## 9. UI design system (all values DEFAULT)

### 9.1 Design principles
- **Calm, fast, clear.** Minimal chrome, generous spacing, nothing that blinks or bounces.
- **Consistent.** Every inirt app and the shell use the same tokens below. Qt apps, GTK apps (the browser), and web content must all follow the system dark/light setting.
- **Keyboard-first and touch-friendly.** Everything reachable by keyboard; touch targets at least 44 px on touch devices.
- **Performance first.** No blur, heavy shadows, or long animations by default.

### 9.2 Color tokens
| Token | Dark | Light |
|---|---|---|
| background | `#0F1115` | `#F6F7FA` |
| surface | `#171A21` | `#FFFFFF` |
| surface-raised | `#1F232C` | `#FFFFFF` (with soft shadow) |
| border | `#2A2F3A` | `#DDE1E8` |
| text | `#E8EAF0` | `#14171C` |
| text-muted | `#9AA3B2` | `#5B6472` |
| accent | `#6C7CFF` | `#4A5BFF` |
| accent-hover | `#8592FF` | `#6171FF` |
| success | `#3DD68C` | `#1FA463` |
| warning | `#F5B84B` | `#C98A12` |
| danger | `#FF5C6C` | `#E0364A` |

All text must meet WCAG AA contrast. The accent color is user-changeable in settings (a short list of presets plus custom).

### 9.3 Typography
- UI font: **Inter** (open license, bundled with the OS). Monospace (terminal, code): **JetBrains Mono**.
- Size scale: 12 / 14 / 16 / 20 / 28 px at 1x. Body text 14 px. Text size is adjustable in settings.
- Weights: regular, medium, semibold only.

### 9.4 Shape, spacing, depth
- 4 px spacing grid (4, 8, 12, 16, 24, 32).
- Corner radius: windows 12 px, buttons and inputs 10 px, dock and launcher 16 px, small chips fully rounded.
- Borders: 1 px using the border token. Shadows: one soft, low-cost shadow for floating surfaces only.
- **Transparency without blur** is the default. Blur is an optional setting, off by default, with a note that it costs performance.

### 9.5 Windows
- The **compositor draws the title bar** (server-side decorations) so every app, including renamed ones, looks consistent.
- Title bar 36 px high: window title centered, window buttons (minimize, maximize, close) on the right, close button turns danger color on hover.
- Rounded corners, 1 px border, soft shadow. The focused window has a slightly stronger border (accent at low opacity).
- Fast, simple window animation: 120-160 ms ease-out, and a "reduce motion" setting that turns all animation off.

### 9.6 Dock
- **Bottom of the screen**, floating, horizontally centered, 16 px radius, **semi-transparent** (surface token at about 80% opacity, no blur by default).
- Layout, left to right: launcher button, pinned and running apps, a thin divider, then the status area with **wifi, battery, and time**.
- Icon size 48 px by default, adjustable from 32 to 72 px. Running apps show a small dot under the icon. Hover gives a subtle highlight, no bouncing.
- **Editable:** drag to reorder, drag out to remove, right-click an icon for Keep in Dock / Remove from Dock / Quit. Settings also has a Dock page to add and remove apps.
- **Unpinned apps appear in the dock while open and disappear when closed.** Pinned apps stay.
- Clicking time opens a small panel with date and a calendar. Clicking wifi or battery opens a small panel (network list; battery percentage and power mode).
- The dock never overlaps maximized windows (windows stop above it unless set to auto-hide).

### 9.7 Launcher (pop-out panel)
- Opens from the dock launcher button or by tapping Super. Anchored above the dock at the bottom-left, **search field focused immediately**.
- Search is fuzzy and covers apps and settings pages. Results update as you type. Arrow keys move, Enter launches, Esc closes.
- Below the search: a Recent row, then an **All apps** grid (alphabetical). Right-click an app to add it to the dock.
- On touch: swipe up from the bottom edge to open it.

### 9.8 Dialogs and the admin prompt
- Modal dialogs are centered, 12 px radius, with clear primary and secondary buttons (primary uses accent).
- The **admin password prompt** is one shared dialog: shows what is being changed, one password field, Cancel and Confirm. It dims the screen behind it. Never ask for the admin password any other way.

### 9.9 Lock screen and home screen
- **Lock screen:** wallpaper, large light-weight time and date, the user's name, and a password field. A clear, calm look; no extra widgets.
- **Home screen:** wallpaper only, with the dock.
- Lock and home wallpapers can be set separately or together in settings.

### 9.10 Wallpapers: 20 originals, each with light and dark variants
- Generated in code as original abstract art (gradients, soft shapes, light geometric patterns, subtle noise). Never download, copy, or imitate existing artwork.
- Render large enough for the biggest target screen, and scale/crop to fill any screen (16:10 Mac, 3:2 Surface). Store as WebP to save space.
- Names (DEFAULT): Aurora, Dusk, Tide, Ember, Moss, Slate, Orchid, Glacier, Sand, Nebula, Ink, Citrus, Coral, Midnight, Haze, Pine, Rose Quartz, Copper, Mist, Prism.
- The light/dark variant follows the system mode automatically.

### 9.11 Settings app
- Layout: left sidebar of sections, a search box at the top of the sidebar, and a clean page on the right. Changes apply instantly where possible.
- Sections: **Appearance** (dark/light, accent, text size), **Wallpapers**, **Dock**, **Display**, **Keyboard Shortcuts**, **Network**, **Sound**, **Power & Battery**, **Accounts**, **Security**, **Apps**, **About**.
- **Security page** (the most important page):
  - Passwords: change user password (needs admin password), set/change admin password
  - Disk encryption status
  - Secure Boot keys (enroll and sign)
  - App permissions
  - Firewall
  - System protection status (read-only root, with the explicit unlock step)
  - Hardened malloc: shown as **Always on**, read-only, cannot be toggled
- Anything that needs admin shows a small lock icon and uses the shared admin prompt.
- **Keyboard Shortcuts page:** lists every action with its current shortcut, grouped (Apps, Windows, System, Launcher), with a search box.
  - To change a shortcut: click it, press the new key combination, Esc cancels, Backspace clears.
  - To add a shortcut: pick an app or command and assign a key combination.
  - Conflict detection: if the combination is already used, warn and offer to swap or cancel.
  - Reset one shortcut or all shortcuts to defaults at any time.
  - Changes apply instantly, no restart.
  - Shortcuts are handled by the compositor and stored in one plain config file (`~/.config/inirt/keybinds.toml`), so they work even when an app is hung and are easy to back up.
  - **Safety:** the quit-focused-app action (default Super+Q) can be rebound but can never be left unassigned. A terminal command (`inirt-keybinds --reset`) always restores defaults, even if the desktop is misconfigured.

### 9.12 App icons and branding
- Every app gets a **custom logo**: SVG, rounded-square tile, one simple glyph, a subtle two-tone gradient from the accent family, readable at 24 px. One consistent style across all apps.
- Every app gets a **renamed, custom-styled window** (title, app name, and about text). Consistent with the title bar in section 9.5.
- Interface icons: one consistent line-icon style (2 px stroke at 24 px, rounded ends).

### 9.13 Motion and feedback
- Durations 120-180 ms, ease-out. No bounce, no parallax, no long fades.
- A global **Reduce motion** setting disables all animation. Respect it everywhere.

### 9.14 Accessibility
- WCAG AA contrast, scalable text, full keyboard navigation, visible focus rings (accent), and screen-reader labels on all controls.

---

## 10. Browser
- Custom browser on **WebKitGTK** with a fully custom UI. No Chromium-based engines anywhere.
- Address bar: direct URLs go straight to the site; other input searches Google (changeable in settings).
- Stays logged in on sites; plays video with all normal features (H.264, VP9, AV1, hardware decoding where possible).
- **Cookie files are encrypted**, with the key held in the system keyring, and the browser runs in a sandbox. Block third-party cookies.
- The browser UI uses the same design tokens (section 9) and follows the system dark/light setting, including a matching `prefers-color-scheme` for web pages.

## 11. Apps and branding
- Terminal: Konsole. Files: Dolphin. Editor: VSCodium. Office: Calligra. PDF: Okular. Video: mpv. Music: Elisa. Images: gThumb. Notes: QOwnNotes. Games: inirt game app plus Prism Launcher (needs Java). VPN: Tailscale. Developer tools: Git, Podman, Python, Java.
- **Every app gets a custom logo and a renamed, custom-styled window** (section 9.12).
- Deliberately not included: email app, chat app, screenshot/recording tool, torrent/download apps.
- The password manager must read and write the KeePass (KDBX) format.

### Rename map
| Original | inirt name |
|---|---|
| Konsole | Prompt |
| Dolphin | Drift |
| VSCodium | Forge |
| Calligra | Draft (its parts: Draft Words, Draft Sheets, Draft Slides) |
| Okular | Leaf |
| mpv | Reel |
| Elisa | Tune |
| gThumb | Frame |
| QOwnNotes | Jot |
| Prism Launcher | Cubes |
| Tailscale | Mesh |

Rules for renaming:
- Only the **display name and logo** change: window title, launcher and dock label, about text. Internal package and binary names stay the same as upstream so dependencies and updates don't break.
- Keep each app's **upstream license notice and credit** in its About page (for example, "Prompt is based on Konsole").
- Use these names everywhere the user sees them: launcher, dock, settings, window titles, and messages.

## 12. How to work
- One small task at a time; plan in 2-3 lines before editing multiple files.
- Read relevant files before editing. Do not guess at code you have not seen.
- Keep responses short; avoid re-reading large files.
- Each task ends with a checklist the user can test in UTM.
- Build order: compositor, then dock and launcher, then lock/home screens and wallpapers, then settings and passwords.

## 13. Progress log (`PROGRESS.md`)
`PROGRESS.md` in the project root is the project's memory. The AI has no memory between sessions, so this file is how work carries over.

- **At the start of every session:** read `PROGRESS.md` first, then continue from "Next up".
- **After every completed task, and before ending a session:** update it. Never skip this.
- **Never delete history.** Add new log entries at the top; keep old ones.
- Keep entries short and factual. Do not paste large code.
- If something is unfinished or broken, say so plainly. Never mark it done.
- Record every ISO release (its number and what it contains) in the log.

Format:

```
# inirt-os Progress

## Status
- Current phase: [e.g., 1 - compositor]
- Last ISO: inirt-000N (or none)
- Works in UTM: yes/no | MacBook Pro: yes/no/untested | Surface: untested

## Next up
1. [next task]

## Blocked / open questions
- [anything waiting on the user]

## Log (newest first)
### YYYY-MM-DD - [task name]
- Done: [what changed, files touched]
- Tested: [how it was verified, and where: UTM / MacBook Pro / Surface]
- Problems: [anything broken or left undone]
- Next: [what should happen next]
```
