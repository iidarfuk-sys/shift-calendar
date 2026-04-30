# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

シフト管理システム — a Japanese shift scheduling app consisting of two standalone HTML files with no build tools, frameworks, or package manager.

- `index.html` — スタッフ向け: staff enter their monthly shift preferences
- `viewer.html` — 管理者向け: admin views all staff preferences and enters confirmed shifts

## Architecture

### Backend
All data is stored and retrieved via a single Google Apps Script (GAS) endpoint:
```
GAS_URL = 'https://script.google.com/macros/s/AKfycbz.../exec'
```

- **POST** (no-cors): submit shift preferences (`index.html`) or save confirmed shifts (`viewer.html`)
- **GET `?type=kibou&yearMonth=YYYY-MM`**: fetch staff preference data
- **GET `?type=kakutei&yearMonth=YYYY-MM`**: fetch confirmed shift data

### Data flow
```
index.html  →  POST to GAS  →  Google Sheets
viewer.html ←  GET from GAS ←  Google Sheets
                ↑
        POST confirmed shifts
```

### Staff list
Defined as a hardcoded JavaScript array in both files:
```js
var STAFF = ['野口','吉田','磯村','音光寺','原'];
```
To add/remove staff, update this array in **both** files.

### Local storage (`index.html`)
Shift preferences are cached with keys:
- `sh_{name}_{year}_{month}` — day-by-day status and time data
- `sent_{name}_{year}_{month}` — flag marking a submission as sent

### Screens (`index.html`)
Single-page app with 5 screens toggled via CSS `active` class:
`screen-top` → `screen-name` → `screen-calendar` → `screen-done`
(and `screen-schedule` for viewing confirmed shifts)

### View modes (`viewer.html`)
Three modes toggled via `currentMode`:
- `kibou-all` — all staff preferences on one calendar
- `kibou-single` — one staff member's preferences
- `kakutei` — admin input mode for confirmed shifts (2-step modal: staff selection → per-person time setting)

## No build commands

These are plain HTML files. Open directly in a browser or serve with any static file server:
```
python3 -m http.server 8000
```
