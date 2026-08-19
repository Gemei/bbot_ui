# BBOT TUI

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![Platform](https://img.shields.io/badge/platform-linux%20%7C%20macos%20%7C%20windows-lightgrey.svg)](https://www.python.org/)
[![Textual TUI](https://img.shields.io/badge/TUI-Textual-blueviolet)](https://textual.textualize.io/)

A self-contained terminal UI for browsing and analyzing [BBOT](https://www.blacklanternsecurity.com/bbot/) scan results.

Supports **BBOT 2.x and 3.x** NDJSON (`output.json`), including 3.0 breaking changes (`data_json`, FINDING-with-severity instead of `VULNERABILITY`, dict-backed URLs, `uuid` / `host_metadata`). See the [3.0 migration guide](https://www.blacklanternsecurity.com/bbot/Stable/migration/3.0_breaking_changes/).

## Features

- **Zero setup** — single self-installing file; creates its own venv on first run
- **BBOT 2.x + 3.x** — auto-detects event schema (`data` vs `data_json`)
- **Scan browser** — live scans and archives in one place, with name filter
- **Accurate status** — RUNNING / FINISHED / INTERRUPTED via process detection (psutil)
- **Workspace views** — issues, assets, events, stats, and preset in a three-pane layout
- **Intel menus** — IPs, emails, social profiles, and subdomains
- **Annotations** — triage vulns/findings with status, priority, and notes
- **Archives** — ZIP compress / restore with integrity checks
- **Copy** — selected row (`c`), full table as markdown (`C` / `Shift+C`), JSON (`y`), or mouse selection
- **Live refresh** — updates while a scan is running
- **In-app help** — press `?`

## BBOT version notes

| Topic | BBOT 2.x | BBOT 3.x (how the UI maps it) |
|-------|----------|--------------------------------|
| Vulns | `type: VULNERABILITY` | `type: FINDING` with severity `CRITICAL`/`HIGH`/`MEDIUM`/`LOW` |
| Findings | `type: FINDING` | `FINDING` with severity `INFO` (or empty) |
| Event payload | `data` (string or dict) | Dict events use `data_json`; strings still use `data` |
| URLs | `data: "https://…"` | `data_json: { "url": "https://…", … }` |
| Severity aliases | `INFORMATIONAL`, `MODERATE` | Normalized to `INFO`, `MEDIUM` |
| Confidence | n/a | Shown when present (`UNKNOWN`…`CONFIRMED`) |
| Modules | e.g. `httpx` | e.g. `http` (displayed as recorded in the scan) |

## Quick start

```bash
chmod +x bbot-ui
./bbot-ui                    # default: ~/.bbot/scans
./bbot-ui /path/to/scans     # browse a scans directory
./bbot-ui /path/to/one-scan  # open a single scan (has output.json)
```

First run creates `~/.bbot_ui_venv/` and installs dependencies. Later runs start immediately.

## Usage

```bash
./bbot-ui --help
./bbot-ui --scan-interval 5 --list-interval 10
```

| Option | Description | Default |
|--------|-------------|---------|
| `path` | Scans directory or single scan folder | `~/.bbot/scans` |
| `--scan-interval SECONDS` | Live refresh inside a scan | `2.0` |
| `--list-interval SECONDS` | Browser list refresh | `3.0` |

Settings are stored in `~/.bbot_ui_config.json`.

## Interface

### Browser (home)

| Key | Action |
|-----|--------|
| `1` / `2` | Scans / Archives |
| `/` | Filter by name |
| `Enter` | Open scan |
| `a` | Archive scan |
| `u` | Unarchive |
| `d` | Delete |
| `c` / `C` or `Shift+C` | Copy row / whole table (markdown) |
| `r` | Refresh |
| `?` | Help |
| `q` | Quit |

Status values: **● RUNNING** · **✓ FINISHED** · **⚠ INTERRUPTED** · **▣ ARCHIVE**

### Workspace (scan open)

Three panes: **views** (left) · **table + filters** (center) · **details** (right).

| Key | View | Group |
|-----|------|--------|
| `1` | Vulns | Issues |
| `2` | Findings | Issues |
| `3` | IPs | Assets |
| `4` | Emails | Assets |
| `5` | Social | Assets |
| `6` | Subdomains | Assets (if `subdomains.txt` exists) |
| `7` | Events | Raw feed |
| `8` | Stats | Meta |
| `9` | Preset | Meta |

| Key | Action |
|-----|--------|
| `Tab` | Next view |
| `/` or `f` | Focus search |
| `c` | Copy selected table row (markdown) |
| `C` / `Shift+C` | Copy entire visible table (markdown) |
| `y` | Copy JSON for current item |
| `t` | Annotate (vulns/findings) |
| `x` | Mark false positive |
| `i` | Mark accepted risk |
| `r` | Refresh |
| `?` | Help |
| `q` / `Esc` | Back to browser |

Filters:

- **Vulns / Findings** — status dropdown (default: Actionable)
- **Events** — event type dropdown
- **Social** — platform dropdown
- **Search** — space-separated terms (AND)

## Archives

1. In the browser, select a non-running scan → `a` → confirm  
2. Switch to Archives with `2` → select archive → `u` to restore  

Safety: RUNNING scans cannot be archived/deleted; ZIP integrity is checked before removing sources; operations are rolled back on failure.

## Annotations

Stored in `.bbot_ui_annotations.json` next to each scan (never modifies BBOT `output.json`). Included in archives.

| Status | Meaning |
|--------|---------|
| New | Default |
| Investigating | In progress |
| Confirmed | Real issue |
| False Positive | Not real |
| Reported | Sent to team |
| Fixed | Remediated |
| Accepted Risk | Known / accepted |

Optional priorities: Critical, High, Medium, Low.

## Configuration

| Path | Purpose |
|------|---------|
| `~/.bbot_ui_venv/` | App virtualenv |
| `~/.bbot_ui_config.json` | Theme and refresh intervals |
| `<scan>/.bbot_ui_annotations.json` | Per-scan annotations |

Force a clean dependency reinstall:

```bash
rm -rf ~/.bbot_ui_venv && ./bbot-ui
```

## Requirements

- Python 3.8+
- Auto-installed: `textual>=0.47.0`, `rich>=13.0.0`, `psutil>=5.9.0`

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Bad/old venv | `rm -rf ~/.bbot_ui_venv && ./bbot-ui` |
| No scans listed | Need folders containing `output.json` under the path |
| Missing Python | Install `python3` and `python3-venv` (or equivalent) |
| Status stuck on RUNNING | Ensure `psutil` is in the venv (reinstall venv above) |

## Project layout

```
bbot-ui     # self-installing executable (single file)
README.md
LICENSE
```

## License

MIT — see [LICENSE](LICENSE).
