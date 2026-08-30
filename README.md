# SOU Song Database

## Local Development

### Materials Server
```bash
cd materials-server
npm install
npm start
```

### Frontend
```bash
npm install
npm start
```

Visit http://localhost:3000

## Deployment

### Option 1: Vercel (Recommended)

**Frontend:**
1. Push code to GitHub
2. Import repository in Vercel
3. Set root directory to `/`
4. Add environment variable: `REACT_APP_API_URL=<your-backend-url>`
5. Deploy

**Backend:**
1. Create new Vercel project for `materials-server/`
2. Set root directory to `materials-server`
3. Add environment variables:
   - `NODE_ENV=production`
   - `FRONTEND_URL=<your-frontend-url>`
4. Deploy

### Option 2: Railway (Backend) + Vercel (Frontend)

**Backend on Railway:**
1. Create new project from GitHub
2. Select `materials-server` directory
3. Add environment variables
4. Deploy
5. Copy the Railway URL

**Frontend on Vercel:**
1. Same as Option 1 frontend steps
2. Use Railway URL as `REACT_APP_API_URL`

### Option 3: Render

**Backend:**
1. Create Web Service from repository
2. Root directory: `materials-server`
3. Build command: `npm install`
4. Start command: `npm start`
5. Add environment variables

**Frontend:**
1. Create Static Site from repository
2. Build command: `npm run build`
3. Publish directory: `build`
4. Add environment variable: `REACT_APP_API_URL`

## Post-Deployment

1. Update CORS settings in backend with your frontend URL
2. Test PDF loading functionality
3. Update `.env.production` with actual backend URL

## AI & Chart API Keys (Step-by-step)

Enable the Admin AI assistant and chart-based discovery by configuring providers:

1) OpenAI (optional)
- Sign in: https://platform.openai.com/
- Create a Project → add Billing → create an API Key.
- In `materials-server/.env` set:
   - `OPENAI_API_KEY=...`
   - `OPENAI_MODEL=gpt-4o-mini` (or another model)

2) Anthropic (optional)
- Sign in: https://console.anthropic.com/
- Create an API Key.
- In `materials-server/.env` set:
   - `ANTHROPIC_API_KEY=...`
   - `ANTHROPIC_MODEL=claude-3-haiku-20240307` (or Sonnet/Opus)

3) Soundcharts (charts)
- Get App ID and API token from Soundcharts.
- In `materials-server/.env` set:
   - `SOUNDCHARTS_APP_ID=...`
   - `SOUNDCHARTS_API_KEY=...`

4) Last.fm (fallback popularity)
- Create an API account at https://www.last.fm/api.
- In `materials-server/.env` set:
   - `LASTFM_API_KEY=...`
   - `LASTFM_SHARED_SECRET=...`

5) Apply and test
- Restart the backend.
- Test AI endpoint (must be logged in): POST `/api/ai/query` with `{ "prompt": "Top 10 hits 1995-2000" }`.
- Test charts: GET `/api/search/charts/top?yearStart=1995&yearEnd=1996&limit=5`.

If no AI keys are present, the assistant falls back to a rule-based parser.
