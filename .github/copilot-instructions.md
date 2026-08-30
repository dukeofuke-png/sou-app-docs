# Copilot Instructions: SOU Song Browser & Materials Server

**⚠️ READ MASTER_ARCHITECTURE.MD FIRST ⚠️**

Read `/MASTER_ARCHITECTURE.md` in the project root **before doing anything** in this project. It is the single source of truth for all architecture, data flow, enrichment pipeline, and deployment decisions. Do not rely on any other MD file for architectural decisions.

At the end of every session, add an entry to the SESSION LOG (Section 16) in `MASTER_ARCHITECTURE.md`.

---

## 0. Code Reuse & Script Management
- **Before creating a new script**, always check for existing functionality:
  - Node.js scripts: Check `materials-server/` root directory and review `materials-server/README.md`
  - Python scripts: Check `Song Database/tools/` directory (scripts named `sou_*.py`)
  - Review `materials-server/config.js` for centralized paths and settings
- **Prefer extending existing scripts** over creating new ones with duplicate behavior
- **If you think a new script is needed**:
  1. First propose which existing scripts you inspected
  2. Explain why they are not suitable to modify
  3. Justify the need for a new entry point
- **Never introduce duplicate entry points** for the same task if a CLI already exists
- **Use existing utility modules**:
  - `materials-server/csvUtils.js` - CSV parsing, writing, validation
  - `materials-server/errorHandler.js` - Logging, error handling, retry logic
  - `materials-server/config.js` - Centralized paths and configuration

## 1. Overview
Two runtimes plus offline data prep:
- React frontend (`sou-song-browser/`) shows enriched song metadata & links to PDF sheets.
- Node/Express materials server (`materials-server/`) proxies PDFs and may serve songs JSON.
- Offline enrichment (CSV → enriched CSV → exported JSON) lives in `Song Database/` + `scripts/`.
- Playlist seed database (`materials-server/data/expanded_seed_base.json`) - separate from teaching song database, used for Spotify playlist discovery.

## 2. Data Flow
**Teaching Songs (for app):**
`Song Database/School of Uke Song Sheets Database.csv` (teaching metadata) → SQLite database `materials-server/data/sou_songs.db` (215 songs, 177 columns) → API enrichment (Wikipedia, GetSongBPM, Spotify, Last.fm, YouTube) → `GET /api/songs` endpoint → React frontend. Database uses snake_case fields, API converts to camelCase via `toFrontendFormat()`. See `MASTER_ARCHITECTURE.md` Section 5 for complete data flow diagram.

**Playlist Seed Database (for discovery):**
`materials-server/data/expanded_seed_base.json` - collection of songs from Spotify playlists used for music discovery. Each song has:
- Core fields: `title`, `artist`, `spotifyId`
- Metadata: `genres` (array), `popularity.spotify`, `releaseYear`
- Discovery: `source`, `discoveredDate`
- **Genre enrichment**: Run `materials-server/enrichGenres.js` to populate `genres` array. Fetches from both Last.fm and MusicBrainz APIs, merging results for comprehensive genre coverage (no Spotify due to rate limits)

## 3. Key Directories & Files
- `materials-server/server.js`: Express app; CORS, health, PDF proxy, future auth/CRUD.
- `materials-server/enrichGenres.js`: Genre enrichment for seed database. Fetches from both Last.fm and MusicBrainz APIs, merging results for comprehensive genre coverage (Spotify removed due to rate limits).
- `materials-server/data/expanded_seed_base.json`: Playlist seed database (~47K songs) for music discovery.
- `sou-song-browser/src/`: React components, config, song list rendering.
- `songs_app_export_merged.json`: Primary dataset (array of song objects). Cache by file mtime when serving.
- `Song Database/archive/` & `scripts/`: Historical/utility Python scripts—do not call from runtime.

## 4. Runtime Conventions
- **Backend port:** 3002 (configured in `materials-server/.env`, overrides server.js default of 3001)
- Server endpoints: keep responses JSON `{...}`; errors -> `res.status(500).json({ error: 'message' })`.
- Field names: Database uses snake_case (`bpm_best`, `release_date_consolidated`), API converts to camelCase (`bpm`, `releaseDate`).
- No direct CSV mutations in live server; all operations go through SQLite database.
- Environment vars: frontend uses `REACT_APP_API_URL`; backend can use `PORT`, `FRONTEND_URL`, `MATERIALS_PATH`.
- **Code quality standards**:
  - All new scripts must use `csvUtils.js` for CSV operations (no custom parsers)
  - All new scripts must use `errorHandler.js` for logging and error handling
  - All functions must have JSDoc comments
  - API calls must use `retryWithBackoff` for resilience
  - Rate limits must come from `config.rateLimits` (not hardcoded)
  - File paths must come from `config.paths` (not hardcoded)

## 5. Frontend Data Access Pattern (Example)
```js
const API = process.env.REACT_APP_API_URL || 'http://localhost:3001';
useEffect(() => { fetch(`${API}/songs`).then(r => r.json()).then(setSongs); }, []);
```
Fallback locally to port 3001; never hardcode production URLs.

## 6. Backend Songs Endpoint Pattern
```js
app.get('/songs', (req, res) => {
  // stat sync for mtime, load & cache JSON once, then res.json(cache)
});
```
Avoid re-reading file each request; compare `stats.mtimeMs`.

## 7. PDF Proxy Pattern
Proxy Google Drive file IDs → set headers `Content-Type: application/pdf`, `Content-Disposition: inline`. Handle failures with status passthrough.

## 8. Development Workflow
1. Backend: `cd materials-server && npm install && npm start` (port 3001).
2. Frontend: `npm install && npm start` (CRA dev server port 3000).
3. Validate: `curl http://localhost:3001/health` and open `http://localhost:3000`.
4. If CORS issues appear, restrict/adjust allowed origin (currently permissive). Use `FRONTEND_URL` in production.

## 9. Deployment Notes
- Frontend: `npm run build` → deploy `build/` (S3/CloudFront, Vercel, etc.). Set `REACT_APP_API_URL`.
- Backend: Keep stateless. Future AWS Lambda: wrap Express with `serverless-http` (already dependency).
- Avoid bundling large CSV in Lambda; prefer pre-generated JSON in object storage.

## 10. Future Admin & CRUD (Present but Dormant)
Auth libs (`bcryptjs`, `express-session`, `cookie-parser`) are installed but unused. When enabling CRUD:
- Add `requireAuth` middleware before any write routes.
- Write updated songs to a versioned CSV (`songdb_master_v2_<date>.csv`) rather than overwriting master.

## 11. Extension Guidelines
- New metadata field: add in enrichment pipeline first → regenerate JSON → consume in frontend.
- Large file operations: move to background/offline script; do not block request handler.
- **Seed database metadata enrichment**: After importing new playlists to `expanded_seed_base.json`, run `enrichGenres.js` to populate missing genre data. The script uses Last.fm API with MusicBrainz as fallback (Spotify removed due to rate limit issues). Expected fields after enrichment:
  - `title`, `artist`, `spotifyId` (always present from import)
  - `releaseYear`, `popularity.spotify` (from Spotify track data during import)
  - `genres` array (enriched by `enrichGenres.js` using Last.fm + MusicBrainz)
- **Do NOT attempt to enrich seed database with audio features** (BPM, key, energy, danceability) - these are only for the teaching song database (`Song Database/`).

## 12. Common Pitfalls
- Importing large JSON directly in React increases bundle size—prefer fetch if growth continues.
- Accidentally mutating cached JSON object; clone before modifications if needed.
- Hardcoded absolute paths (e.g., to local Drive mount) break other environments; parameterize.
- **CRITICAL: Spotify Credentials Not Loading** - Shell environment variables (from `.zshrc`, `.bashrc`) override `.env` file values. If scripts fail with "Token request failed: Bad Request" or "invalid_client", run `unset SPOTIFY_CLIENT_ID SPOTIFY_CLIENT_SECRET` before executing. This issue has occurred 10+ times. See `materials-server/README.md` → Configuration → "⚠️ Common Issue: Spotify Credentials Not Loading" for full resolution steps.
- **CRITICAL: GetSongBPM Wrong Endpoint** - The public endpoint `https://api.getsongbpm.com/search/` is protected by Cloudflare bot detection and returns HTML instead of JSON. **Always use `https://api.getsong.co/search/`** with parameters `type=both&lookup=song:{title} artist:{artist}&limit=1`. The Python enrichment scripts (`sou_enrich_getsongbpm.py`) use the correct endpoint. See `ENRICHMENT_SERVICE_SUMMARY.md` for full details.

## 13. Quick Health Check Snippet
```bash
curl -I http://localhost:3001/health
```
Expect 200 with JSON body `{ status: 'ok', ... }` when server running.

Provide feedback if you need deeper coverage on enrichment scripts, PDF ID extraction, or planned auth model.
