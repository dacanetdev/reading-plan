# 📚 Reading Plan Tracker

A personal reading and learning tracker for a 38-week, 7-phase curriculum covering 28 books and 3 Udemy courses.

## 🔗 Open the App

**[→ Launch Reading Plan Tracker](https://dacanetdev.github.io/reading-plan/tracker.html)**

## Features

- **4 reading tracks** — desk reading, Speechify (audio), reference, and Udemy courses
- **Phase-based curriculum** — 7 phases with date-aware progress tracking
- **Daily log & streak counter** — records activity per day and maintains a consecutive-day streak
- **GitHub sync** — state is saved as `state.json` in this repo via the GitHub Contents API; every save becomes a commit
- **Undo support** — revert today's entries for any book or course
- **No build step** — single `tracker.html` file, no dependencies, no backend

## Setup

1. Open the app link above (or serve `tracker.html` locally with `python -m http.server 8080`).
2. Click the **⚙** button and enter your GitHub sync settings:
   - **Token** — a GitHub Personal Access Token with *Contents: Read & Write* (fine-grained) or *repo* scope (classic)
   - **Owner / Repo** — `dacanetdev` / `reading-plan`
3. The token is stored only in your browser's `localStorage` and is never sent anywhere other than the GitHub API.

## Curriculum

See [`reading_plan.md`](reading_plan.md) for the full 38-week plan with all books, courses, and phase targets.

## Tech

Plain HTML + CSS + vanilla JS in a single file. State lives in `localStorage` (local cache) and `state.json` (remote backup via GitHub API). Hosted on GitHub Pages — no CI/CD needed; push to `main` and it's live.
