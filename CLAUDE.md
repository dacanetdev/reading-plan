# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A personal reading and learning tracker — a single-file static web app (`tracker.html`) with no build step, no dependencies, and no backend. All state is persisted in browser `localStorage`.

The companion file `reading_plan.md` is the master planning document (38-week, 7-phase curriculum across 28 books and 3 Udemy courses).

## Running the App

Open `tracker.html` directly in any browser, or serve it locally:

```
python -m http.server 8080
```

The `.vscode/launch.json` is configured to open Edge at `http://localhost:8080`.

## Deployment (GitHub Pages)

The repo is intended to be hosted on GitHub Pages. Push to the `main` branch; GitHub Pages serves `tracker.html` from the repo root. No CI/CD or build configuration is needed.

## Architecture

Everything lives in `tracker.html` — HTML structure, embedded `<style>`, and a single `<script>` block.

### Data model (in-memory + localStorage)

- `PLAN_START` — ISO date string for the first day of the plan (`'2026-04-06'`)
- `PHASES[]` — 7 phase objects: `{ name, start, end, books[], courses[], targets: { read, speechify, reference, udemy } }`
- `BOOKS{}` — keyed by book ID: `{ title, author, track, pages }`
- `COURSES{}` — keyed by course ID: `{ title, totalMinutes, startMinutes }`
- `state` — loaded/saved to `localStorage` under `STORAGE_KEY = 'rp_state_v1'`:
  ```js
  {
    progress:       { [bookId]:   { pagesRead, started, finished } },
    courseProgress: { [courseId]: { minutesWatched } },
    dailyLog:       { [date]:     { books: {}, courses: {} } },
    streak:         { lastLogDate, count }
  }
  ```

### Four reading tracks

| Track | Color | Key |
|-------|-------|-----|
| Read (desk) | blue `#5aa3d8` | `read` |
| Speechify (audio) | orange `#e07c44` | `speechify` |
| Reference | green `#68b87e` | `reference` |
| Udemy | purple `#a87ed4` | `udemy` |

### Key functions

- `getCurrentPhase()` — returns the active phase object by comparing today's date against phase date ranges
- `addPages(bookId, pages)` / `addMinutes(courseId, minutes)` — update progress and daily log, then re-render
- `undoBook()` / `undoCourse()` — revert only today's entries for a given item
- `updateStreak()` — maintains consecutive-day activity count; checks both books and courses
- `saveState()` — writes to localStorage immediately, then schedules a debounced `pushToGitHub()` (1.5s)
- `loadLocalState()` / `mergeState(remote)` — local cache load and remote-state merge with default fill-in

### GitHub sync layer

State is persisted as `state.json` in the repo root via the GitHub Contents API. Each save becomes a commit.

- `ghConfig` — `{ token, owner, repo }` loaded from `localStorage['rp_github_config']`
- `fileSha` — in-memory SHA of the current `state.json`; required for updates (GitHub rejects PUT without correct SHA)
- `fetchFromGitHub()` — GET `state.json`, updates `fileSha`, returns parsed state (null on 404)
- `pushToGitHub()` — PUT `state.json`; on 409 (SHA conflict) refetches SHA and retries via `schedulePush()`
- `initApp()` — async startup: loads config, fetches remote state (falls back to localStorage on error), renders
- Sync status badge in header: grey = idle, amber pulse = saving, green = saved, red = error (click to retry)

### Settings modal

The ⚙ button opens the GitHub sync settings. First-time users are auto-redirected here on load.
Requires a GitHub Personal Access Token with **Contents: Read & Write** on the repo (fine-grained) or **repo** scope (classic). The token is stored only in browser localStorage.

### Rendering

There is no virtual DOM or framework. Each user action calls `render()` at startup or targeted `renderBookCard()` / `renderCourseCard()` on individual updates, rebuilding DOM via `innerHTML`. Animations are triggered by toggling CSS classes after the DOM write.

## .gitignore

PDFs and EPUBs (the actual book files) are excluded from the repository — see `.gitignore`.
