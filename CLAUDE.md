# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A single-file Pomodoro timer web app. There is no build system, package manager, or test suite — everything (HTML, CSS, JS) lives in `index.html`. `img/` holds two unused/legacy progress icons (the current UI renders progress indicators as inline SVG instead).

## Development

- No install/build step. Open `index.html` directly in a browser, or serve it locally, e.g.:
  ```
  python3 -m http.server 8000
  ```
- There is no linter or test runner configured. Verify changes by opening the page in a browser and exercising the timer manually (start/pause/skip/reset, settings save, keyboard shortcuts).

## Architecture

Everything is contained in `index.html`:

- **`<style>` block** — all CSS, using custom properties defined on `:root` (colors, radius, transitions). Responsive breakpoints at 968px and 640px collapse the two-column layout to one column.
- **`<script>` block** — a single IIFE containing all app logic. Key parts:
  - `STORAGE` keys namespace all `localStorage` usage: settings, current timer state, per-day focus totals (`pomodoro_today_<dateString>`), and all-time totals.
  - `state` (mode, `endTs`, `running`, `pomodoroCount`) is the single source of truth for the timer and is persisted to `localStorage` on every change plus a 2s interval, so a reload or tab close mid-session resumes correctly by comparing `endTs` to `Date.now()` (see the `window.addEventListener('load', ...)` handler).
  - Timing is wall-clock based (`state.endTs = Date.now() + duration*1000`), not a decrementing counter — this is what makes resume-after-reload and pause/resume correctness work; don't switch to a naive `setInterval` countdown without preserving this.
  - `STATES` cycles WORK → SHORT_BREAK → WORK → ... → LONG_BREAK based on `settings.cyclesBeforeLong`, driven by `completePeriod()` (natural completion) and `skipPeriod()` (manual skip, mirrors the same transition logic).
  - `render()` is the single UI sync function — call it after any state mutation rather than updating DOM elements piecemeal.
  - `els` is a flat lookup of all DOM element references, populated once at the top of the IIFE.
  - `window.__pomodoro` exposes `state`/`settings`/a `reset()` helper for debugging in the browser console.
- Audio ding and favicon/PWA manifest icon are inlined as data URIs — there are no external asset files for those.
