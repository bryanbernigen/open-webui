# Deployment Guide: Vercel (Frontend) + Railway (Backend)

This app is split into two parts:
- **Frontend**: SvelteKit SPA deployed to Vercel (static hosting)
- **Backend**: Python FastAPI deployed to Railway (persistent process)

## Prerequisites

1. **Neon Postgres** (free) — you already have this for your other project
2. **Railway account** — [railway.app](https://railway.app) (free tier, GitHub login)
3. **Vercel account** — you already have this

---

## Step 1: Deploy Backend to Railway

1. Go to [railway.app](https://railway.app) → **New Project** → **Deploy from GitHub repo** → select your fork of open-webui
2. Railway auto-detects the `Dockerfile`
3. Add environment variables in Railway Dashboard → **Variables**:

```
DATABASE_TYPE=postgresql
DATABASE_HOST=<your-neon-host.neon.tech>
DATABASE_PORT=5432
DATABASE_USER=<your-neon-username>
DATABASE_PASSWORD=<your-neon-password>
DATABASE_NAME=openwebui
WEBUI_SECRET_KEY=<generate with: node -e "console.log(require('node:crypto').randomBytes(32).toString('hex'))">
OPENAI_API_KEY=<your-api-key>
ENV=prod
PORT=8080
WEBUI_AUTH=true
ENABLE_INITIAL_ADMIN_SIGNUP=true
WEBUI_ADMIN_EMAIL=admin@example.com
WEBUI_ADMIN_PASSWORD=ChangeMe123!
WEBUI_ADMIN_NAME=Admin
CORS_ALLOW_ORIGIN=https://open-webui.vercel.app
FORWARDED_ALLOW_IPS=*
USE_SLIM=true
SCARF_NO_ANALYTICS=true
DO_NOT_TRACK=true
ANONYMIZED_TELEMETRY=false
ENABLE_VERSION_UPDATE_CHECK=false
```

4. Under **Settings** → **Networking** → **Generate Domain** (this gives you `*.up.railway.app`)
5. Note the Railway URL (e.g. `https://open-webui.up.railway.app`)

---

## Step 2: Update Vercel Rewrites

Once Railway gives you a domain, update `vercel.json` rewrites with your Railway URL:

```json
"destination": "https://open-webui.up.railway.app/api/:path*"
```

Push the change to GitHub.

---

## Step 3: Deploy Frontend to Vercel

1. Go to [vercel.com](https://vercel.com) → **Add New** → **Project** → import your fork
2. Set environment variable in Vercel:
   - `OLLAMA_BASE_URL` = `https://open-webui.up.railway.app`
3. **Deploy**
4. Note your Vercel URL (e.g. `https://open-webui.vercel.app`)

---

## Step 4: Update CORS (final step)

After getting both URLs:
1. In **Railway** dashboard → `CORS_ALLOW_ORIGIN` → set to your Vercel URL
2. In **Vercel** dashboard → `OLLAMA_BASE_URL` → verify it points to Railway
3. Redeploy if needed

---

## Step 5: First Login

1. Visit your Vercel URL
2. Sign up with the admin credentials you set:
   - Email: `admin@example.com`
   - Password: `ChangeMe123!`

Then go to **Settings** → **Connections** → add your OpenAI/Anthropic API key.

---

## Troubleshooting

**Railway cold starts**: First request after inactivity takes ~30-60s (free tier). `USE_SLIM=true` skips ML model downloads to minimize this.

**WebSocket issues**: If chat doesn't update in real-time, Railway may not support WebSocket upgrades on free tier. The app falls back to HTTP polling automatically.

**Database connection error**: Double-check your Neon credentials. Make sure your Neon project allows connections from Railway (check Neon dashboard → Connection → Allowed IPs).

**Build fails on Vercel**: Make sure `USE_SLIM=true` is set in Railway so it doesn't try to download 500MB+ of ML models on startup.
