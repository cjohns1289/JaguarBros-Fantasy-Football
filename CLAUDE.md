# JaguarBros Fantasy Football — Project Context for Claude

## What this project is
A full-stack fantasy football league website for a 12-team PPR league called the **JaguarBros Fantasy Football League**. Built in React (Vite), hosted on Netlify, connected to GitHub for auto-deploy, using Sleeper API for live data and Firebase Realtime Database for pick storage and Google Auth.

**Live URL:** https://jaguarbros.netlify.app  
**GitHub:** https://github.com/cjohns1289/JaguarBros-Fantasy-Football  
**Netlify:** https://app.netlify.com (project: jaguarbros)

---

## Critical configurations (already hardcoded in src/App.jsx)

```javascript
const SLEEPER_USERNAME = "CommishChris";  // Case-sensitive — capital C's
const NFL_SEASON = "2025";               // Update to "2026" each September
const LEAGUE_START_YEAR = 2020;          // Year league moved to Sleeper
const ESPN_START_YEAR = 2014;            // Year league started on ESPN
const GOOGLE_CLIENT_ID = "367228013896-saon70qkr5e5s2mqnrdjchi422uci7jt.apps.googleusercontent.com";
```

**Firebase config** is hardcoded in FIREBASE_CONFIG block — project ID: `jaguarbros-fantasy-football`, databaseURL: `https://jaguarbros-fantasy-football-default-rtdb.firebaseio.com`

---

## Tech stack
- **Frontend:** React + Vite (JSX, no TypeScript, inline styles only — no Tailwind, no CSS modules)
- **Hosting:** Netlify (free tier, 300 build credits/month — conserve carefully)
- **Auth:** Google Identity Services (OAuth 2.0, no Firebase Auth SDK)
- **Database:** Firebase Realtime Database (REST API only, no Firebase SDK)
- **Data:** Sleeper API (public, no auth required, uses CORS proxy fallback)
- **Build:** `npm run build` → `dist/` folder → Netlify reads `netlify.toml`

---

## File structure
```
JaguarBros-Fantasy-Football/
├── src/
│   ├── App.jsx        ← ENTIRE app is one file. All components, logic, styles here.
│   └── main.jsx       ← Entry point, do not modify
├── index.html
├── package.json
├── vite.config.js
├── netlify.toml       ← Build command: npm run build, publish: dist
└── .gitignore         ← node_modules and dist excluded
```

**Everything lives in `src/App.jsx`.** This is intentional — do not split into multiple files without discussing with the user first.

---

## Deployment workflow
1. Make changes to `src/App.jsx`
2. User tests locally with `npm run dev` (localhost:5173)
3. User copies updated `src/App.jsx` into local GitHub Desktop repo folder
4. GitHub Desktop: commit → Push origin
5. Netlify auto-deploys from GitHub main branch
6. **Always provide a zip** containing the full project (excluding node_modules and dist)

**CRITICAL:** Always run `npm run build` locally before packaging. A build failure on Netlify wastes credits. The most common build errors have been:
- Duplicate `const` declarations (especially `JAX_TEAM_ID`, `LOCKDOWN_IMG`, `ESPN_START_YEAR`)
- Missing constant declarations used in JSX
- Wrong variable names (`LOCK_SCREEN_IMG` vs `LOCKDOWN_IMG`)

---

## Tab order (current)
1. Standings
2. Weekly Picks
3. Pick Leaderboard
4. Weekly Incentives
5. Scoreboard
6. Teams
7. League History
8. Rules
9. Setup Guide

---

## Key features and their implementation

### Sleeper API
- Uses `fetchWithFallback()` which tries direct → corsproxy.io → allorigins.win
- Session cache (5 min TTL) + localStorage persistence for offline fallback
- Stale data banner + retry button if all fetches fail
- `getLeagueData()` walks `previous_league_id` chain for history

### Google Sign-In (Weekly Picks auth)
- Uses Google Identity Services (`accounts.google.com/gsi/client` script)
- No Firebase Auth SDK — pure OAuth token + userinfo endpoint
- First sign-in: user claims their team → saved to Firebase `usermap/{uid}`
- Teams can only be claimed once — claimed teams hidden from list
- Race condition protection: re-fetches usermap before saving claim
- To reset a user: delete their entry from Firebase `usermap/` node

### Weekly Picks window
- Opens: Tuesday 3:00am ET
- Locks: Thursday 8:15pm ET
- Uses `America/New_York` timezone via Intl API (auto EDT/EST)
- Rechecks every 30 seconds
- Locked screen shows "Lock Down The Bank" image + submission checklist with timestamps

### Firebase data structure
```
picks/
  usermap/
    {googleUid}: { owner, team, rosterId, email, claimedAt }
  week1/
    {googleUid}: { picks..., submittedAt, displayName, teamName }
  week2/
    ...
```

### League History
- Walks Sleeper's `previous_league_id` chain back to 2020
- ESPN era (2014-2019) hardcoded in `ESPN_HISTORY` constant
- `ESPN_START_YEAR = 2014` must be declared — absence caused white screen crash
- All-time W/L records aggregate from Sleeper seasons only
- All-time points include current in-progress season

### Weekly Incentives (15 weeks)
- Auto-calculates winners from Sleeper API after each week completes
- Bulls-Eye & Overachiever use `m.projected_points` from matchup object (team-level, same as Sleeper scoreboard)
- Last In First Out: uses `roster.settings.rank === 6` (real-time, updates when commissioner re-seeds)
- Week 14 LIFO fetches fresh rosters via API call for real-time accuracy
- Weeks needing player data (3,4,7,8,9,10,11) fetch full NFL players list

### Jaguars game tile
- On Standings tab, above standings table
- Uses `sleepercdn.com/images/team_logos/nfl/{abbr}.jpg` for real logos
- `onError` fallback hides broken images gracefully
- Schedule from `https://api.sleeper.app/v1/schedule/nfl/regular/{season}`

---

## Known issues & history

### Issues that were fixed
- `ESPN_START_YEAR is not defined` → white screen on League History — fixed by ensuring constant is declared
- `LOCK_SCREEN_IMG` vs `LOCKDOWN_IMG` variable name mismatch
- Duplicate `const JAX_TEAM_ID` declaration — build error
- Duplicate `const LOCKDOWN_IMG` declaration — build error
- `commishchris` (lowercase) vs `CommishChris` (correct) — Sleeper username is case-sensitive
- NFL_SEASON set to "2024" instead of "2025" — caused no league found
- Files uploaded flat to GitHub root instead of preserving `src/` folder structure — broke build
- `FIREBASE_CONFIG.databaseURL` containing "PASTE_" placeholder — Firebase not working

### Current known limitations
- Jaguars schedule tile may show "unavailable" if Sleeper's NFL schedule endpoint returns no data for that week
- Bulls-Eye/Overachiever projected_points field may be 0 if Sleeper hasn't populated it yet for the week
- Weekly Incentives loads slowly (fetches player data for multiple weeks sequentially)
- Google Sign-In requires OAuth consent screen to be configured in Google Cloud Console

---

## League details
- **12 teams**, PPR scoring
- **6 playoff spots** (Weeks 15-17, championship Week 17)
- **Sacko** = worst record (shown with 💩 emoji)
- **Standings badges:** Gold (#1), Silver (#2), Bronze (#3)
- **ESPN history:** 2014-2019 (manual, hardcoded)
- **Sleeper history:** 2020-present (auto via API)
- **Weekly picks incentives:** $10 each, 15 weeks
- **Season resets:** Update `NFL_SEASON` constant each September

---

## ESPN historical data (hardcoded, do not delete)
```javascript
const ESPN_HISTORY = [
  { year: 2014, champion: "TXJR",            sacko: null },
  { year: 2015, champion: "TXJR",            sacko: null },
  { year: 2016, champion: "CommishChris",     sacko: "JohnnyLiar" },
  { year: 2017, champion: "spamas",           sacko: "quarensheed" },
  { year: 2018, champion: "CommishChris",     sacko: "TXJR" },
  { year: 2019, champion: "joethebestjagbro", sacko: null },
];
```

---

## Netlify credit conservation
- Free tier: 300 build credits/month (resets ~21st of each month)
- Each GitHub push = 1 deploy = credits used
- **Always test locally first** with `npm run dev` before pushing
- **Batch changes** — collect multiple fixes before deploying
- Build failures also cost credits — always verify `npm run build` succeeds locally first
- Current OAuth Google Client ID and Firebase config are hardcoded — no need to edit for normal changes

---

## How to package for deployment
```bash
# In the project root:
npm run build
echo "/*  /index.html  200" > dist/_redirects
zip -r deploy.zip . --exclude "node_modules/*" --exclude "dist/*" --exclude ".git/*"
```
User then extracts zip, copies files into GitHub Desktop repo, commits, pushes.

---

## What NOT to do
- Do not split App.jsx into multiple files
- Do not add npm packages without discussing with user first
- Do not change NFL_SEASON without user confirmation
- Do not use localStorage in artifacts (use React state)
- Do not guess at Sleeper API field names — check against known working fields
- Do not provide a zip without running `npm run build` first
- Do not deploy directly — user controls all GitHub pushes
- Do not use browser storage APIs that don't work in Claude artifacts
