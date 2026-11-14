# PROVENANCE — Fox's hands (CuaDriver)

This repository is a fork. It is not our original work. This file records exactly
what it came from, so that any code review can tell at a glance what is upstream's
and what is ours, and so that a future reviewer can decide whether something new
upstream is worth taking.

Filed under WINGSCRUM-598 (feature "Fox's hands under our own control"), story WINGSCRUM-599.
Registered in the fork register, WINGSCRUM-322.

## What this is

| | |
|---|---|
| **Upstream repo** | https://github.com/trycua/cua |
| **Upstream project** | Cua Driver (`cua-driver-rs`), in `libs/cua-driver/` |
| **Pinned commit** | `acf84b91609d021497deaf771c781fad2b89ced5` |
| **Pinned tag** | `cua-driver-rs-v0.5.8` |
| **Upstream version** | CuaDriver **0.5.8** |
| **Commit date** | 2026-06-21 01:32:27 UTC |
| **Commit subject** | `Bump cua-driver-rs to v0.5.8` |
| **Our pinned branch** | `fox/pin-0.5.8` |
| **Licence** | MIT — see `LICENSE.md`, unmodified |
| **Copyright** | Copyright (c) 2025 Cua AI, Inc. — **retained, not removed, not reassigned** |
| **Date forked** | 2026-11-14 |
| **Forked by** | Raven, for Kuchu Software Solutions Inc. |

## Why 0.5.8 and not something newer

0.5.8 is the version **we actually run**. It is not a guess:

- `/Applications/CuaDriver.app/Contents/Info.plist` reports
  `CFBundleShortVersionString = 0.5.8`, `CFBundleIdentifier = com.trycua.driver`.
- `/Applications/CuaDriver.app/Contents/MacOS/cua-driver --version` reports
  `cua-driver 0.5.8`.
- That binary is dated **Jun 20 18:35 local = Jun 21 01:35 UTC**, three minutes after
  the pinned commit was authored. The installed binary and this commit are the same build.

## Why we forked at all

We need our hands under our own control.

CuaDriver is how Fox moves the mouse, presses keys and reads the screen on the Mac
Mini. It is the single component between Fox's intent and a real click on a real
machine. Three reasons that has to be pinned and forked rather than tracked:

1. **Upstream drift must not change Fox's actuation behaviour without us choosing it.**
   Upstream moves fast — the driver went from 0.5.8 (June 2026) to 0.49.x (November
   2026) in five months, and the driver ships an **auto-updater**
   (`crates/cua-driver/src/updater.rs`, `version_check.rs`). A behaviour change that
   arrives on its own is a change to how Fox touches a live machine. That decision is ours.
2. **The behaviour we rely on is measured, not documented.** Fox's working method depends
   on quirks we found by measurement, not from the README — most importantly that
   **every modifier hotkey silently fails while CuaDriver's own agent cursor is enabled**,
   returning a success string while doing nothing. Those measurements are version-specific.
   An upgrade invalidates them until they are re-measured.
3. **Fork provenance is a standing definition of done** on this project: anything forked
   from a Git repo must say so in the code itself.

## What we changed

**Nothing in upstream's code.** As of the date above, this fork adds only this file.
`LICENSE.md` and the upstream copyright notice are untouched.

The pinned branch `fox/pin-0.5.8` exists so the commit we depend on cannot drift even
if upstream rewrites history or retags.

## Upstream state at fork time — recorded for later, NOT adopted

Recorded so a future review starts from facts instead of re-deriving them. **None of
this is adopted. Taking any of it is a decision, with a re-measurement cost.**

- Upstream's driver line at fork time is **0.49.x** (0.49.11 on 2026-11-14) against our
  0.5.8. Roughly 44 minor versions of drift.
- **Correction worth recording: Windows support is NOT new.** It is often described as
  something upstream added since. It was already present **at 0.5.8**: this very commit
  contains the `crates/cua-driver-uia/` crate (Windows UI Automation),
  `Skills/cua-driver/WINDOWS.md`, `cua-driver.manifest`, `cua-driver.rc` and
  `tests/e2e_windows_bg_input_test.rs`, and `rust/PARITY.md` at this commit already
  describes the `page` tool as "cross-platform: Apple-Events on macOS, UIA+CDP on
  Windows, AT-SPI+CDP on Linux". What upstream has *hardened* since is a different
  claim from what it *added*.
- What genuinely did land after 0.5.8, from upstream's own release notes: Linux capture
  paths (GNOME Mutter logical frame, rootless Xwayland, a Hyprland plugin), Windows
  consent-matcher and virtual-desktop fixes, sigstore-signed release assets, a nightly
  channel, and macOS fixes including background-click refusal under an app's own popup
  and screenshot memory release.
- **Packaging changed shape.** 0.5.8 installs as `/Applications/CuaDriver.app`, which is
  what holds our TCC grants. The current line installs via `install.sh` to a plain binary
  path. An upgrade therefore lands a new binary at a **new path with zero TCC grants**,
  so it is not a drop-in replacement — it is a re-permissioning exercise.

### What triggers a re-evaluation

- A CVE or a security advisory against the driver.
- A capability Fox actually needs that 0.5.8 cannot do.
- Needing Fox's hands on a non-macOS machine in earnest.
- Upstream deleting or retagging this commit.

A re-evaluation means reading the diff and proposing what is worth taking — not
upgrading because a newer number exists.

## Licence and security intake

- **Licence: MIT.** Permissive. Commercial use, modification and distribution allowed.
  The one obligation is to keep the copyright and permission notice in all copies or
  substantial portions — `LICENSE.md` is unmodified and stays that way.
- **What the driver can reach on the Mac**, which is the honest reason this is gated:
  - **Screen capture** of the whole display, and per-window image capture.
  - **Synthetic keyboard and mouse events** — real clicks, real keystrokes, delivered
    to whatever is in front of them.
  - **The accessibility tree** of other applications: reading UI element values, and
    writing them (`type_text` via `kAXSelectedText`, `set_value` via `AXValue`).
  - **Process control** — it can kill an application.
  - **Network and self-update**: it embeds an HTTP/MCP server
    (`crates/cua-driver/src/mcp_http.rs`, `serve.rs`, `proxy.rs`), an updater
    (`updater.rs`, `version_check.rs`), autostart (`autostart.rs`) and
    **telemetry** (`crates/cua-driver/src/telemetry.rs`; a telemetry id exists on the
    machine at `~/.cua-driver/.telemetry_id`).
- **Permissions it actually holds**, read from the macOS TCC databases: under its own
  identity `com.trycua.driver` it holds **Accessibility** and **Apple Events**, and
  **not** Screen Recording or Input Monitoring. Screen capture arrives through the
  host application's own grant, not the driver's.
- **What it sits behind:** it is reachable only on **loopback** on the Mac Mini, it is
  never exposed through the Fox gateway, and any action that would hit an Allow/consent/
  install prompt goes through the approval loop in WINGSCRUM-493 — Fox asks first and
  only clicks on an explicit yes. A locked screen silently disables all input and all
  accessibility reads while still returning success, so lock state is checked before any
  actuation run is believed.

## How to see what upstream has done since

```bash
git remote add upstream https://github.com/trycua/cua
git fetch upstream
git log --oneline acf84b91609d021497deaf771c781fad2b89ced5..upstream/main -- libs/cua-driver/
```
