# TrainSense — Web Demo

A live, browser-runnable build of the [TrainSense](https://github.com/pradeepg78/TrainSense)
NYC subway app, packaged for deployment on **Vercel** (frontend + backend in one project).

It's the real React Native (Expo) app compiled for the web, with the Flask API
running as a serverless function — so the whole thing lives at a single URL on
Vercel's free tier.

## Layout

```
public/          Prebuilt Expo web export (the app). Served as the static site.
  └─ showcase/   Phone-frame page that embeds the app (for portfolios).
backend/         Flask app (code + prebuilt SQLite DB). Loaded by the function.
api/index.py     Serverless entry — exposes the Flask WSGI app at /api/*.
requirements.txt Trimmed Python deps for the function.
vercel.json      Serves public/, routes /api/* to the function.
```

## Deploy

### Option A — Vercel Dashboard (no CLI)

1. Create a new GitHub repo and push this folder to it.
2. [vercel.com](https://vercel.com) → **Add New → Project → Import** the repo.
3. Framework preset: **Other** (`vercel.json` handles build/routing).
4. **Environment Variables:** add `MAPBOX_ACCESS_TOKEN` = your Mapbox public
   (`pk....`) token. The build injects it into the bundle at deploy time, so the
   token is never committed to git. (MTA feeds are public — no other secrets.)
5. **Deploy.**

### Option B — Vercel CLI

```bash
npm i -g vercel
cd trainsense-web
vercel --prod
```

## URLs after deploy

| URL | What |
|-----|------|
| `/`            | the live app |
| `/showcase`    | the app inside an iPhone frame (drop this in a portfolio) |
| `/api/health`  | backend health check (verify the function works) |

## Embedding in a portfolio

```html
<iframe
  src="https://<your-project>.vercel.app/showcase"
  style="width: 520px; height: 920px; border: 0;"
  allow="geolocation"
  title="TrainSense demo">
</iframe>
```

## Updating the app build

This repo ships a **prebuilt** `public/`. To refresh it after changing the app
in the main repo:

```bash
cd /path/to/TrainSense/mobile
npx expo export --platform web
cp -r showcase dist/showcase
rm -rf /path/to/trainsense-web/public && cp -r dist /path/to/trainsense-web/public
```

## Notes

- **Cold starts:** the first station-tap after idle may take ~1–3s while the
  function wakes; the map is instant (data is bundled).
- **SQLite is read-only** on Vercel (copied to `/tmp` at startup); the
  `/api/data/load` admin endpoint is not available there — not needed for the demo.
