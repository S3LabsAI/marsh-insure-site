# Marsh.insure — website

Local agency site for Greg Marsh's agency in Westminster, MD. Sister site of IHateMyRates.com (`S3LabsAI/Ihate-my-rates`).

## Structure

- `index.html` — the whole site. It is a self-unpacking bundle exported from a design tool (page HTML, JS runtime, React 18.3.1 and fonts are embedded as data islands). See `CLAUDE.md` for how it works and how to edit it safely.
- `vercel.json` — static hosting config (clean URLs, security headers, no-cache on the HTML).

## Local preview

Open `index.html` in a current Chrome, Safari or Firefox, or serve the folder:

```bash
python3 -m http.server 8000
```

## Deployment (Vercel)

Static site, no build step and no environment variables yet. When importing the repo in Vercel:

- Framework Preset: **Other**
- Build Command: leave empty
- Output Directory: leave empty (the repo root is served)
- Install Command: leave empty

Pushes to `main` deploy to production; other branches and PRs get preview deployments.
