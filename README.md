# automate_screenshots

Automates checking SynTempVision/VLC displays remotely, without needing to VNC in manually.

## Layer 1: Capture infrastructure

- **Phase 1 (done):** `Capture-Screen.ps1` — captures the full virtual screen (all
  monitors) to a timestamped PNG using .NET's `SystemInformation.VirtualScreen`.
  Tested locally; this machine only has one monitor, so multi-monitor spanning
  hasn't been verified against real hardware yet.
- **Phase 2:** validate on a real field server with multiple monitors — run the
  script there, compare the result against a manual VNC screenshot taken at the
  same moment.
- **Phase 3:** run it live on each server, viewable via a small svhelp admin page
  (same pattern as `server-stats.php` — PHP `shell_exec()` of a `.ps1` by file
  path) instead of needing VNC at all. Rides the existing `deploy_svhelp.ps1`
  pipeline that already pushes svhelp identically to every server.

## Layer 2: Analysis (bigger, more open-ended goal)

Five distinct detection targets, each a separate technical problem:

| Target | What it needs | Difficulty |
|---|---|---|
| **Fault + timer** (text in panel title bar) | OCR on screenshot — UI Automation doesn't work (confirmed 2026-10-20: SynTempVision renders its camera grid as one custom-drawn surface, zero accessible UI Automation elements found) | Low-medium |
| **Frozen frame** | Diff two consecutive captures of the same panel region | Low |
| **Cold/blank camera** | Low-variance/near-uniform region = likely dead feed | Low |
| **View moved** | Needs a saved "correct" reference image per camera to diff against | Medium |
| **Hotspot / POX heating-cooling-shutdown** | Needs to decode the actual IR color palette to temperature, track trends over time per region | High — closest to the real IR monitoring product itself |

### Open questions (unresolved as of 2026-10-20)

1. **Where does analysis run** — centrally (pull screenshots from every server
   to one place) or locally on each server?
2. **What happens on detection** — just logged, or feeds an alert (email,
   Service Log, a dashboard)?
3. **Per-camera reference images** for "view moved" — who captures/approves the
   "correct" baseline, and when does it get updated after a legitimate re-aim?
4. **Scope boundary with the real IR monitoring product** — hotspot/process-state
   detection is domain logic that may already partly exist in the actual
   product. Worth confirming this isn't duplicating that before building it.
5. **OCR reliability** untested against the actual busy thermal-palette
   backgrounds behind the title text — needs a real trial before trusting it.

### Reference material

348 manually-taken screenshots exist at
`\\FIC-NAS\Public\SynTempVision\SiteCameraChecks\screenshots` (June-Oct 2026),
but none are tagged with known-problem/known-good state — useful as general
reference for what normal looks like, not as calibration data for detection
thresholds without manual review first.
