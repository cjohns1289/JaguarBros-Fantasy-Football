# Local Testing Guide — JaguarBros Fantasy Football

## Prerequisites
You need Node.js installed. Check by running: `node --version`
If not installed, download from nodejs.org (LTS version).

## One-time setup
```bash
# Navigate to your local repo folder (the one GitHub Desktop cloned)
cd path/to/JaguarBros-Fantasy-Football

# Install dependencies (only needed once, or after package.json changes)
npm install
```

## Run locally every time
```bash
npm run dev
```
This starts a local server. Open your browser to: **http://localhost:5173**

The site hot-reloads — any change you save instantly appears in the browser.
No deploy needed. No Netlify credits used.

## Workflow going forward
1. Claude gives you updated zip → extract → copy src/App.jsx into your local repo
2. Run `npm run dev` → test in browser at localhost:5173
3. Only push to GitHub (triggering a Netlify deploy) when you're satisfied
4. This should reduce deploys from 10-20 per change to 1 per change

## Build for production (optional local check)
```bash
npm run build
```
Creates the dist/ folder. If this succeeds locally, Netlify will succeed too.
