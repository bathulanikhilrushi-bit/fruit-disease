# Orchard Loupe

Cloud-based fruit disease detector. Upload a photo of a fruit, and the app
sends it to Claude's vision API for a structured diagnostic report:
condition, confidence, severity, symptoms, treatment, and prevention.

```
orchard-loupe-app/
  backend/    Express API that calls the Anthropic API
  frontend/   Vite + React UI
```

## Prerequisites

- Node.js 18 or newer (check with `node -v`)
- An Anthropic API key: https://console.anthropic.com/settings/keys

## 1. Backend setup

```bash
cd backend
npm install
cp .env.example .env
```

Open `.env` and paste in your real key:

```
ANTHROPIC_API_KEY=sk-ant-...
```

Start the backend:

```bash
npm start
```

You should see:

```
Orchard Loupe backend listening on http://localhost:3001
Using model: claude-sonnet-5
```

Check it's alive: open http://localhost:3001/api/health in a browser — it
should return `{"status":"ok","model":"claude-sonnet-5"}`.

## 2. Frontend setup

In a second terminal:

```bash
cd frontend
npm install
npm run dev
```

Vite will print a local URL, typically http://localhost:5173. Open it in
your browser.

The dev server proxies any request to `/api/*` through to
`http://localhost:3001`, so the frontend and backend talk to each other
automatically — no CORS configuration needed in development.

## 3. Use it

1. Drop a fruit photo into the upload area, or click to browse.
2. Click "Run diagnostic."
3. The report panel fills in with the fruit type, health status, confidence,
   symptoms, treatment steps, and prevention tips.
4. Past scans this session appear as thumbnails you can click back into.

## Building for production

```bash
cd frontend
npm run build
```

This outputs static files to `frontend/dist/`. Serve them with any static
host (nginx, Vercel, Netlify, `serve dist`, etc). Since the production build
won't have Vite's dev proxy, set `VITE_API_BASE_URL` at build time to point
at wherever you deploy the backend:

```bash
VITE_API_BASE_URL=https://your-backend.example.com npm run build
```

The backend itself is a plain Express app — deploy it anywhere that runs
Node (Render, Fly.io, Railway, an EC2 box, etc), as long as `ANTHROPIC_API_KEY`
is set in that environment. Never ship the API key in the frontend build —
it must stay server-side, which is why this app has a backend at all.

## Notes

- The model is set via `CLAUDE_MODEL` in `backend/.env` (defaults to
  `claude-sonnet-5`). Swap in `claude-haiku-4-5-20251001` for a cheaper,
  faster option, or `claude-opus-4-8` for higher accuracy on tricky images.
- To use a dedicated agricultural vision API instead of Claude (e.g.
  Plant.id or Kindwise), the only file to change is `backend/server.js` —
  swap out the `anthropic.messages.create(...)` call for that API's request,
  keeping the same JSON response shape so the frontend doesn't need changes.
- Scan history is kept in memory in the browser tab and resets on refresh.
