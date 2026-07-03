# SquaredUp Diagnostics Analyzer

A single-page, client-side tool for quickly making sense of a SquaredUp on-premises diagnostics package (`.zip`). Drop in a diagnostics zip and get an instant, readable breakdown of errors, warnings, database health, HA status, and reference packages — no extraction, no server, no upload.

**Live tool:** hosted via GitHub Pages from this repo (`index.html`).

## Why this exists

SquaredUp diagnostics packages bundle raw log files, config dumps, and reference package archives. Digging through them by hand to find the actual problem is slow — logs can span multiple files, the same error repeats hundreds of times with slightly different payloads, and easy-to-miss issues (duplicate reference packages, DB lock contention) don't jump out from a text editor.

This tool parses the package entirely in the browser and turns it into a structured, collapsible report so you can go from "customer sent me a zip" to "here's the likely root cause" in seconds.

## What it does

Drag a diagnostics `.zip` onto the page (or browse to select one) and it will:

- **Unzip and decompress in-browser** — no files are extracted to disk or uploaded anywhere; everything happens client-side using the native `DecompressionStream` API.
- **Parse all log files** under `Log/*.log` and merge them into a single chronological timeline.
- **Group errors and warnings** by normalising each message (stripping GUIDs, URLs, timestamps, and variable numbers) so repeated occurrences of the same underlying issue collapse into one group with an occurrence count, first/last seen timestamps, and full context/stack trace on demand.
- **Detect recurring patterns** in grouped events (e.g. "~15min interval") to help spot scheduled jobs or polling loops that are misbehaving.
- **Filter out noise** — known non-actionable errors (e.g. AJAX exceptions) are excluded from counts and groups so they don't drown out real issues.
- **Analyse database health**, flagging "database is locked" events and index connection threshold breaches, including peak/average hold times.
- **Detect High Availability role**, identifying whether the package came from a primary or secondary node (and flagging conflicting/split-brain signals) based on log evidence.
- **Check reference packages** for duplicates — leftover copies from a failed upgrade or manual re-import that can indicate configuration drift.
- **Surface system information** (version, server, and other environment details) pulled from `Overview.txt` / `SquaredUpDiags.txt`.

## Usage

1. Open `index.html` (locally or via the hosted GitHub Pages link).
2. Drag and drop a SquaredUp diagnostics `.zip` onto the page, or click to browse for one.
3. Review the summary strip and expand the sections (Errors, Warnings, Reference Packages, Database Analysis, System Information, High Availability) as needed.
4. Click into any group to see individual occurrences and their full log context.

## Notes

- Everything runs locally in the browser — no data leaves your machine, which makes it safe to use with customer diagnostics packages.
- No build step, dependencies, or install required — it's a single static HTML file.
