# MASTER_ARCHITECTURE.md

**Last Updated:** 20 August 2026  
**Status:** Definitive Source of Truth

**Companion document:** `PRODUCT_ROADMAP.md` (same directory level) is the longer-range strategic/phase roadmap. This document remains the living, every-session-updated record of current technical state and near-term next steps; `PRODUCT_ROADMAP.md` is revisited periodically for longer-range sequencing, not every session.

**Governance rule:** If the two documents ever disagree about what currently exists or how it works, this document is corrected. If they disagree about what to build next or in what order, `PRODUCT_ROADMAP.md` governs. Sequencing, phase numbers, and long-range priority claims belong only in `PRODUCT_ROADMAP.md` — this document should not maintain its own competing phase/priority scheme.

---

## 1. CRITICAL RULES FOR COPILOT

**READ THIS ENTIRE DOCUMENT BEFORE DOING ANYTHING**

1. **Read this entire document** before starting any work in this project
2. **Never rebuild** something already listed as complete in Section 3
3. **Never create a new MD file** without updating this document
4. **Always check this document** before building anything new
5. **If unsure whether something exists**, check the codebase before building
6. **At the end of every session**, add an entry to the SESSION LOG (Section 17)
7. **Where MD files conflict with actual code**, always trust the code
8. **The enrichment pipeline (Section 6)** is definitive — never deviate without explicit instruction from Matthew
9. **Never modify the enrichment source priority order** without Matthew's approval
10. **Check for existing scripts** before creating new ones (see copilot-instructions.md)

---

## 2. PRODUCT VISION

Tutors should be able to use the platform as a comprehensive tool to devise lesson plans, create song sheets (branded chord/lyric sheets and traditional notation), and access theory resources — digital first, with PDF as an output format rather than the primary format. This is the original motivating vision behind the app.

**For current phased direction, priorities, and sequencing (AI song suggestion, digital songsheet creation, curriculum/pedagogy features, eventual student-facing product), see `PRODUCT_ROADMAP.md`.** This document does not separately track forward feature sequencing.

---

## 3. CURRENT STATUS

### ✅ What Works Locally & Deployed

**Backend (materials-server):**
- Express server runs on port 3002 (configured in `.env`)
- SQLite database fully operational: `materials-server/data/sou_songs.db` (218 songs, confirmed via direct query 20 Aug 2026)
- All CRUD endpoints working: `/api/songs`, `/api/songs/:id`, `/api/songs/bulk-update`
- Seed database operations: `/api/seed/search`, `/api/seed/promote`, `/api/seed/export`, `/api/seed/size`
- **Two-stage enrichment pipeline — both fully live:**
  - Lightweight (auto on promote): `enrichmentService.js` → Spotify identity + GetSongBPM + YouTube
  - Rich (on-demand via ⚡): `enrichmentService_sqlite.js` → Wikipedia + Last.fm + GetSongBPM + Spotify cover art + MusicBrainz + release era/season/month derivation (no extra API call)
- **`POST /api/songs/:id/enrich`** — on-demand rich enrichment endpoint, auth-gated
- Chart enrichment (Wikipedia scraper + Soundcharts API)
- BPM/Key enrichment (GetSongBPM API, `api.getsong.co`)
- Spotify integration (track search, artist genres, cover art)
- Last.fm integration (plays, listeners, artist tags)
- YouTube video search (YouTube Data API v3)
- **`wikipedia_intro`** — lead paragraph extraction working; 12/13 pre-credential songs populated
- **Null-overwrite safety** — `dbManager.updateSong()` strips null/undefined before SQL UPDATE
- Auth bypassed for local development only (bypassAuth middleware gated to `NODE_ENV !== 'production'`)
- **Admin panel secured in production** — password-protected via bcrypt + session cookie

**Frontend (sou-song-browser):**
- React app runs on port 3000
- Admin dashboard fully complete (369 lines, stats display, routing)
- **⚡ Enrich Song button** in ManageSOUDatabase — per-row icon (action column) + modal footer button
  - Shows ⏳ while running, ✅ on success with count, ❌ on failure
  - Merges returned song data into table and open edit modal on success
- ManageSOUDatabase component fully complete:
  - Enterprise-grade table with 30+ columns
  - Inline editing (click cell to edit, save on blur)
  - Bulk operations (multi-select, bulk edit)
  - Saved views (save/load column visibility & order)
  - Column drag-to-reorder, sort, search with field selector
- SeedDatabaseSearch component complete:
  - Search 47K+ seed database songs
  - Filter by **title**, artist, year range, genre, popularity (title field added 13 June 2026)
  - Multi-select + "Promote to SOU Database" button (runs lightweight enrichment)
- BulkAddSongs component (bulk import to SOU database)
- **ConversationWorkspace** — AI chat panel with sidebar, multi-turn, Gemini 2.5 Flash, `searchSongs` tool

**Data Pipeline:**
- CSV → SQLite migration completed (3 Dec 2025, 215 songs; 218 as of 20 Aug 2026)
- SQLite database uses snake_case fields
- Field mapping: `toFrontendFormat()` converts snake_case → camelCase for frontend
- Seed database: `expanded_seed_base.json` (47,273 songs) for discovery; `SEED_PATCHES` in `seedService.js` delivers idempotent volume updates on boot (Railway Volume shadows `data/` — committed JSON changes alone have no effect)
- Manual teaching CSV: `School of Uke Song Sheets Database.csv` (teaching metadata)
- **13 pre-credential songs backfilled (13 June 2026):** all have `spotifyTrackId`, `year`, `youtubeVideoId`, `coverArtUrl`; 12/13 have `wikipedia_intro` (A Minha Menina: no Wikipedia infobox match)

### ❌ What's Broken or Incomplete

**Critical Issues:**
1. ~~**Server startup hang**~~ — ✅ **FIXED (6 June 2026)**
   - Root causes: `bcrypt.hashSync` in auth.js (1.7s block), `cheerio` full bundle importing `undici` (infinite hang on Node 22), `uuid` v13 ESM-only (infinite hang in CJS)
   - Fixes: lazy bcrypt hash, `cheerio/slim`, `uuid@9`
   - Server now starts in <1s and binds to port 3002
2. ~~**Teaching data partially imported**~~ — ✅ **212/212 songs complete (6 June 2026)**
   - Shirelles fixed: found by ID `the_shirelles_will_you_love_me_tomorrow`, updated directly (`level='1'`, `sou_keys='C'`)
   - Dawn Penn: was already populated — `level='2'`, `sou_keys='Am'`
3. ~~**Ghost database file**~~ — ✅ **DELETED (6 June 2026)**

**Blocked API Sources:**
4. **Spotify audio features** — Returns 403 Forbidden (Client Credentials flow doesn't support this endpoint)
   - Blocked fields: danceability, energy, valence, mode, key, time_signature, BPM
   - Workaround: GetSongBPM API provides BPM, Key, Mode, Time Signature
5. **GetSongBPM** — Correct endpoint is `api.getsong.co` not `api.getsongbpm.com` (Cloudflare blocks latter)
   - Status: Fixed in code, working correctly
6. **Spotify playlists** — Spotify-created playlists are rate-restricted, only import user playlists

**Not Yet Built:**
7. **AWS deployment** — Never deployed (deadline Dec 2, 2025 was missed)
8. **Songwriter credits** — APIs available (MusicBrainz, Wikidata) but not implemented
9. **Deezer BPM** — Documented as primary BPM source but never built
10. ~~**PDF cloud storage**~~ — ✅ **DONE (7 June 2026)** — 190/191 PDFs on Cloudflare R2. 1 remaining: Espresso (no local file)

### � Deployed to Production (6 June 2026)

- **Backend (Railway):** `https://sou-song-browser-production.up.railway.app` — 215 songs, `/songs` and `/api/songs` return 200
- **Frontend (Vercel):** `https://sou-song-browser.vercel.app` — HTTP 200, bundle has Railway URL compiled in
- **GitHub repos:** `dukeofuke-png/sou-backend` (Railway source), `dukeofuke-png/sou-song-browser` (Vercel source)

### 🎯 Current Deployment Status

**Production app is fully working:**
- Frontend (`https://sou-song-browser.vercel.app`) loads songs correctly
- Backend (`https://sou-song-browser-production.up.railway.app`) serving 216 songs from SQLite
- Railway Volume mounted at `/app/data/`; `sou_songs_seed.db` auto-seeds the volume on first boot
- Auth secured in production; admin panel fully functional
- 190/191 PDFs on Cloudflare R2
- **All 8 enrichment API keys confirmed in Railway:** `SPOTIFY_CLIENT_ID/SECRET`, `LASTFM_API_KEY`, `YOUTUBE_API_KEY`, `GETSONGBPM_API_KEY`, `SOUNDCHARTS_APP_ID/API_KEY`, `SESSION_SECRET` (strong random)
- **⚡ Enrich button live** on Vercel (commit `b88a23d`) — confirmed 42 fields written for "Into the Groove" on first production test
- **13 pre-credential songs backfilled:** `spotifyTrackId`, `year`, `coverArtUrl`, `wikipedia_intro` (12/13), BPM/key/mode/timeSignature (8/13)

**Still outstanding (manual actions):**
1. Upload Espresso PDF to R2 manually when local file located

### ✅ Chord Numerals / Strum Style / Fingerpicking — Corruption Fixed (13 June 2026)

**Root cause:** `importTeachingData.js` used naive `line.split(',')` which cannot handle quoted multi-value CSV fields (e.g. `SOU Keys = "Em, Am"`, `Tags = "R&B, Pop"`). Each internal comma shifted all subsequent columns right, writing genre/tag descriptors into `strum_style` (65 rows affected), `fingerpicking_style` (41 rows), and `chord_numerals` (17 rows). Same family as the earlier `chords`/`level` corruption fixed by `fixTeachingDataImport.py`.

**Fix:** 3 idempotent `UPDATE` statements added to `dbManager.runMigrations()` (commit `2543dfb`). Runs automatically on every server start — no-op once data is clean.

**Legitimate values retained after fix:**
- `strum_style`: `'Reggae Strum'` on Natural Mystic, Tadow, Waiting in Vain (3 songs)
- `chord_numerals`: `'I IV V'` (5 Years Time), `'I IV V vi'` (Beautiful Girls)
- `fingerpicking_style`: all cleared — CSV column was never populated (0 legitimate values)

**⚠️ Warning for future CSV imports:** Never use `String.split(',')` to parse CSV rows. Always use a proper CSV parser (`csv.DictReader` in Python, `csv-parse` or similar in Node.js). Quoted fields containing commas are common in this dataset (Genre, Tags, SOU Keys, Teaching Notes). See `fixTeachingDataImport.py` for the correct Python approach.

---

## 4. TECHNICAL STACK

### Backend (materials-server)

**Runtime:** Node.js 18+  
**Framework:** Express 4.18  
**Database:** SQLite (better-sqlite3) with WAL mode  
**Port:** 3002 (configured in `materials-server/.env`, overrides `server.js` default of 3001)  
**Startup:** `cd materials-server && npm start`  
**Main File:** `materials-server/server.js` (1,035 lines)

**Key Dependencies:**
- `better-sqlite3` — SQLite database driver
- `express-session` + `cookie-parser` — Session management (auth bypassed in dev)
- `cors` — Cross-origin requests (allows localhost:3000)
- `dotenv` — Environment variables
- `chokidar` — File watching for materials folder
- `multer` — File uploads
- `bcryptjs` — Password hashing (unused, auth disabled)

**API Structure:**
```
/api/auth/*           — Login/logout (bypassed in dev)
/api/songs            — GET all songs, POST new song
/api/songs/:id        — GET/PUT/DELETE specific song
/api/songs/bulk-update — PUT bulk edit
/api/seed/*           — Seed database operations (search, promote, export)
/materials/*          — PDF file proxy
/health               — Health check
```

### Frontend (sou-song-browser)

**Framework:** React 18.2 (Create React App)  
**Port:** 3000 (configured in `sou-song-browser/.env`)  
**Startup:** `cd sou-song-browser && npm start`  
**Build:** `npm run build` → outputs to `build/`

**Key Dependencies:**
- `react-router-dom` — Client-side routing
- `react-icons` — Icon library (FiMusic, FiEdit2, etc.)
- Custom CSS modules for each component

**Environment Variables:**
```bash
# sou-song-browser/.env
PORT=3000
REACT_APP_API_URL=http://localhost:3002
REACT_APP_MATERIALS_URL=http://localhost:3002
BROWSER=none  # Don't auto-open browser
```

### Startup Commands (Verified Files)

**Option 1: Keep-Alive Monitor (Auto-Restart)**
```bash
./keep-servers-alive.sh
```
Currently failing — servers restart continuously.

**Option 2: NPM Script (Parallel Start)**
```bash
npm start
```
Starts both servers in parallel (from root `package.json`).

**Option 3: Bash Script**
```bash
./start-all.sh
```
Clears ports, starts servers in background, logs to `logs/backend.log` and `logs/frontend.log`.

**Option 4: Manual (Two Terminals)**
```bash
# Terminal 1
cd materials-server && npm start

# Terminal 2
cd sou-song-browser && npm start
```

---

## 5. DATABASE ARCHITECTURE

### 5.1 SEED DATABASE (Discovery)

**File:** `materials-server/data/expanded_seed_base.json`  
**Size:** ~47,000 songs (~11 MB)  
**Format:** JSON array of song objects  
**Purpose:** Admin catalog expansion — large-scale song discovery from Spotify playlists

**Fields:**
```json
{
  "title": "Song Title",
  "artist": "Artist Name",
  "spotifyId": "7ouMYWpwJ422jRcDASZB7P",
  "releaseYear": 2015,
  "genres": ["pop", "indie"],
  "popularity": { "spotify": 75 },
  "source": "artist_playlists",
  "discoveredDate": "2025-11-23"
}
```

**Access:** Admin only (via `/api/seed/*` endpoints)  
**Usage:** Search → Select → Promote to SOU Teaching Database (runs enrichment)

---

### 5.2 SOU TEACHING DATABASE (SQLite)

**File:** `materials-server/data/sou_songs.db`  
**Size:** 1.2 MB (218 songs, confirmed via direct query 20 Aug 2026; 215 at initial migration 3 Dec 2025)  
**Format:** SQLite database (single table: `songs`)  
**Schema:** `materials-server/schema.sql` (372 lines, 239 columns — confirmed via live `pragma_table_info` query 26 Aug 2026; grew from the original 177 via subsequent `ALTER TABLE` migrations)  
**Migration Date:** 3 December 2025 14:14:06  
**Migration Script:** `materials-server/migrateCSVToSQLite.js` (536 lines)

**Purpose:** Production teaching database with rich metadata, PDFs, and enrichment data

**Schema Summary:**
- Core fields: `id`, `title`, `artist`, `year`, `album`, `duration_s`
- Musical attributes: `bpm_best`, `key_best`, `mode`, `time_signature_best`, `tempo_label`
- Genres & tags: `genres_best`, `genres_spotify`, `genres_lastfm`, `tags_lastfm_track`, `tags_lastfm_artist`
- Release dates: `release_date_consolidated`, `release_year_consolidated`, `release_month`, `release_season`, `release_era`
- Chart positions: `chart_peak_us`, `chart_peak_uk`, `chart_peak_aus`, etc. (12 countries), `top_10`, `top_40`
- Popularity: `spotify_popularity`, `lastfm_plays`, `lastfm_listeners`, `youtube_views`
- Credits: `songwriters_consolidated`, `producer`, `label`
- Cover art: `cover_art_url_best`, `cover_art_url_spotify`, `cover_art_url_lastfm`
- YouTube: `youtube_video_id`, `youtube_url`, `youtube_title`, `youtube_views`
- Spotify: `spotify_track_id`, `spotify_isrc`, `spotify_release_date`
- Wikipedia: `wikipedia_url`, `wikipedia_intro`, `wikipedia_charts_text`
- Materials: `song_sheet_status`, `tab_status`, `song_sheet_path`, `melody_tab_path`, `materials_json`
- Teaching metadata: `level`, `sou_keys`, `num_chords`, `chords`, `chord_numerals`, `strum_style`, `fingerpicking_style`, `teaching_notes`
- Enrichment timestamps: `last_enriched_utc`, `spotify_audio_features_retrieved_at`, `lastfm_retrieved_at`, `getsongbpm_retrieved_at`, etc.

**Teaching metadata status (confirmed via direct query 20 Aug 2026):** import from CSV is complete for most fields — `sou_keys` 209/218, `level` 147/218, `teaching_notes` 127/218. `chords` remains sparse (32/218) — the source CSV itself was never fully populated for this field; see `Song Database/School of Uke Song Sheets Database.csv` for current coverage.

**Access:** Tutors via frontend (`/api/songs`), Admins via ManageSOUDatabase component

**Field Naming:** Database uses `snake_case` (e.g., `bpm_best`, `release_date_consolidated`)

---

### 5.3 MANUAL SOU DATABASE (CSV Override)

**File:** `Song Database/School of Uke Song Sheets Database.csv`  
**Size:** 212 songs (matches SQLite database count approximately)  
**Format:** CSV with teaching-specific columns  
**Purpose:** Original curated song list with teaching metadata maintained by tutors

**Key Fields (Teaching-Specific):**
- `Songsheet` — Yes/No flag for PDF availability
- `Major/Minor` — Mode override (manual first, API fallback)
- `Original Key` — Key override (manual first, API fallback)
- `SOU Keys` — Keys taught in SOU curriculum
- `Level` — Difficulty rating for students
- `TAB` — Yes/No flag for tablature availability
- `Teaching Notes` — Pedagogical notes for tutors
- `Strum Style` — Strumming pattern description
- `Finger-picking Style` — Fingerpicking pattern description
- `No. of Chords` — Chord count
- `Chords` — List of chords used
- `Chord Numerals` — Roman numeral analysis
- `Time Signature` — Manual override
- `BPM` — Manual teaching tempo override
- `Tempo` — Tempo label (Slow, Moderate, Fast, etc.)
- `PDF_ID` — Google Drive file ID for sheet music

**Relationship to SQLite:** This CSV is the SOURCE OF TRUTH for teaching metadata. These fields should be periodically imported into SQLite to populate the empty teaching columns.

---

### 5.4 COMPLETE DATA FLOW DIAGRAM

```
┌──────────────────────────────────────────────────────────────────────┐
│ DISCOVERY PHASE                                                       │
│ Spotify Playlists → Import Script                                    │
│   ↓                                                                   │
│ expanded_seed_base.json (47K songs, minimal metadata)                │
│   ↓                                                                   │
│ Admin searches seed database via SeedDatabaseSearch.js               │
│   ↓                                                                   │
│ Admin selects songs → "Promote to SOU Database" button               │
└──────────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────────┐
│ ENRICHMENT PHASE                                                      │
│ POST /api/seed/promote {spotifyIds[]}                                │
│   ↓                                                                   │
│ enrichmentService_sqlite.enrichSong(songId)                          │
│   ├─ Wikipedia: Charts + Metadata + Songwriters                      │
│   ├─ GetSongBPM: BPM + Key + Mode + Time Signature                   │
│   ├─ Spotify: Track info + Cover art + Artist genres                 │
│   ├─ Last.fm: Plays + Listeners + Tags                               │
│   ├─ YouTube: Video ID + URL                                         │
│   └─ Soundcharts: Charts + Release date + Label (optional)           │
│   ↓                                                                   │
│ dbManager.enrichSong() writes to SQLite                              │
└──────────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────────┐
│ STORAGE PHASE                                                         │
│ sou_songs.db (SQLite, snake_case fields)                             │
│   - API-sourced fields (title, artist, BPM, key, etc.)               │
│   - Teaching fields (populated — imported from CSV, see 5.2)         │
└──────────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────────┐
│ TEACHING DATA IMPORT (NEEDED)                                        │
│ School of Uke Song Sheets Database.csv                               │
│   ↓ (import script needed)                                           │
│ Populates teaching fields in sou_songs.db:                           │
│   - Level, SOU Keys, Chords, Teaching Notes, etc.                    │
└──────────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────────┐
│ API LAYER                                                             │
│ GET /api/songs                                                        │
│   ↓                                                                   │
│ dbManager.getAllSongs() → SELECT * FROM songs                        │
│   ↓                                                                   │
│ toFrontendFormat() converts snake_case → camelCase                   │
│   (bpm_best → bpm, release_date_consolidated → releaseDate)          │
│   ↓                                                                   │
│ JSON response [{id, title, artist, bpm, releaseDate, ...}, ...]      │
└──────────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────────┐
│ FRONTEND                                                              │
│ React App (localhost:3000)                                           │
│   ↓                                                                   │
│ fetch(`${API_URL}/api/songs`) on component mount                     │
│   ↓                                                                   │
│ ManageSOUDatabase.js displays songs in table                         │
│   - Inline editing (click cell → edit → blur to save)                │
│   - Bulk operations                                                   │
│   - Column reordering                                                 │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 6. ENRICHMENT PIPELINE — DEFINITIVE SOURCE ORDER

**THIS SECTION IS THE SINGLE SOURCE OF TRUTH FOR ENRICHMENT LOGIC**

Enrichment runs in two separate, sequential pipelines when a song is promoted from the Seed Database to the SOU Teaching Database — not in a single file.

### 6.0 Two-Pipeline Architecture

**Pipeline 1 — `materials-server/enrichmentService.js` (lightweight, synchronous)**
- Called via `enrichSongs()`/`enrichSong()` directly inside both promote paths — `routes/seed.js` `POST /api/seed/promote` (human bulk-promote) and `services/seedService.js` `promoteSongFromSeed()` (AI single-song tool) — and awaited before the API response returns.
- Provides only what's needed for a song to be immediately usable: BPM (Deezer primary → GetSongBPM fallback), Key/Mode/Time Signature (GetSongBPM), Spotify track identity (accepts an optional known Spotify ID to skip re-search), YouTube video ID.
- Does not write to the database itself — returns an enriched object; the calling route/service inserts it via `dbManager.addSong()`.

**Pipeline 2 — `materials-server/enrichmentService_sqlite.js` (rich, asynchronous)**
- Triggered via `triggerBackgroundEnrichment(songId)` immediately after a successful insert, in both promote paths — fire-and-forget, never awaited, never delays the promote response.
- Writes `enrichment_status` (`pending` → `complete`/`failed`), `enrichment_last_attempt_at`, `enrichment_error` directly to the DB as it runs (columns added Session 18) so the admin UI can show live progress (Complete/Failed/Enriching…).
- Writes all enriched fields directly to SQLite itself via `dbManager` (unlike Pipeline 1).
- Provides the deep data: Wikipedia (chart positions, background/composition/cover-versions text, songwriters, producer, label, release date, recording info), Soundcharts fallback for charts/release date, Last.fm (plays/listeners/tags), Spotify artist genres + cover art, merged/deduplicated genre field, MusicBrainz fallback for release date.
- Also independently invokable on-demand via `POST /api/songs/:id/enrich` (the admin ⚡ Enrich button) or in bulk via `batchEnrichAll.js` — re-running it does not re-trigger Pipeline 1.

The source priority table and execution order below (6.1–6.2) describe **Pipeline 2**. Pipeline 1's scope is limited to the fields listed above, using the same Deezer-first BPM priority described in the table.

### 6.1 Enrichment Source Priority Table (Pipeline 2)

| Field | Primary Source | Fallback 1 | Fallback 2 | Fallback 3 | Fallback 4 | Notes |
|-------|---------------|-----------|-----------|-----------|-----------|-------|
| **BPM** | Deezer | GetSongBPM | Spotify Audio | - | - | Deezer-first as of Session 18 (20 Aug 2026); GetSongBPM fallback; Spotify requires OAuth (blocked) |
| **Key** | GetSongBPM | Spotify Audio | - | - | - | GetSongBPM primary; Spotify blocked |
| **Mode** | GetSongBPM | Spotify Audio | - | - | - | Major/Minor from GetSongBPM |
| **Time Signature** | GetSongBPM | Spotify Audio | - | - | - | GetSongBPM supports 3/4, 5/4, 7/8; Spotify limited |
| **Genre** | MERGED ALL | Wikipedia | GetSongBPM | Last.fm | Spotify | Deduplicated merge from all sources |
| **Tags** | Last.fm Track | Last.fm Artist | - | - | - | Track tags prioritized over artist tags |
| **Release Date** | Wikipedia | Soundcharts | MusicBrainz | Spotify | Wikidata | Wikipedia primary, structured APIs as fallback |
| **Chart Position** | Wikipedia | Soundcharts | - | - | - | Wikipedia scrapes infoboxes; Soundcharts API fallback |
| **Songwriter** | Wikipedia | MusicBrainz | Wikidata | - | - | MusicBrainz NOT YET IMPLEMENTED |
| **Cover Art** | Spotify (640x640) | Soundcharts | Last.fm (300x300) | - | - | Spotify best quality |
| **YouTube URL** | YouTube Data API | - | - | - | - | Searches "{title} {artist} official" |
| **Spotify ID** | Spotify Track Search | - | - | - | - | Already exists from seed import |
| **Popularity** | Spotify Track | Last.fm Plays | YouTube Views | - | - | Multiple popularity metrics available |
| **Duration** | Spotify Track | Wikipedia | - | - | - | Spotify provides in milliseconds |
| **Wikipedia Intro** | Wikipedia API | - | - | - | - | First paragraph of Wikipedia article |
| **Producer** | Wikipedia | - | - | - | - | Extracted from infobox |
| **Label** | Wikipedia | Soundcharts | - | - | - | Record label |
| **Album** | Wikipedia | Spotify | - | - | - | Album name |
| **Recorded** | Wikipedia | - | - | - | - | Recording date/studio info |

### 6.2 Enrichment Execution Order

```javascript
// enrichmentService_sqlite.enrichSong(songId)

1. Wikipedia Metadata + Chart Positions (ALWAYS FIRST)
   - scrapeChartPositions() — US Billboard, UK Singles, 12 countries
   - scrapeMetadata() — Infobox data (songwriters, label, release date, etc.)
   - Prioritization: US > UK > best of others for display
   - Sets: chart_peak_us, chart_peak_uk, top_10, top_40, songwriters_wikipedia, label, etc.

2. GetSongBPM API (api.getsong.co)
   - BPM, Key, Mode, Time Signature
   - Artist genres from GetSongBPM
   - Sets: bpm_getsongbpm, key_getsongbpm, mode_getsongbpm, time_signature_getsongbpm

3. Spotify Audio Features (BLOCKED — 403 Forbidden)
   - Would provide: danceability, energy, valence, key, mode, time_signature, BPM
   - Skipped due to OAuth requirement

4. Spotify Track Info
   - Cover art (640x640 highest quality)
   - Album name
   - Sets: cover_art_url_spotify, cover_art_album_spotify

5. Last.fm Track
   - Plays, Listeners
   - Track tags
   - Sets: lastfm_plays, lastfm_listeners, tags_lastfm_track

6. Last.fm Artist
   - Artist tags (used as genres)
   - Sets: tags_lastfm_artist

7. Spotify Artist
   - Artist genres from Spotify
   - Sets: genres_spotify

8. YouTube Data API
   - Video ID and URL
   - Sets: youtube_video_id, youtube_url, youtube_title, youtube_views

9. Genre Merge
   - Deduplicates genres from all sources
   - Sets: genres_best (merged string), genres_best_sources (provenance)

10. MusicBrainz (NOT YET IMPLEMENTED)
    - Would provide: songwriters, release date, recording ID

11. Write to Database
    - dbManager.enrichSong() updates SQLite record
```

### 6.3 BROKEN OR RATE-LIMITED SOURCES (DO NOT USE)

**❌ Spotify Audio Features**
- **Endpoint:** `https://api.spotify.com/v1/audio-features/{id}`
- **Status:** Returns 403 Forbidden
- **Reason:** Client Credentials flow doesn't support this endpoint (requires user OAuth)
- **Fields Blocked:** `spotify_danceability`, `spotify_energy`, `spotify_valence`, `spotify_mode`, `spotify_key`, `spotify_time_signature`, `spotify_bpm`
- **Workaround:** Use GetSongBPM for BPM, Key, Mode, Time Signature
- **Fix Required:** Would need OAuth flow (not feasible for batch enrichment)

**⚠️ GetSongBPM Endpoint Correction**
- **WRONG Endpoint:** `https://api.getsongbpm.com/search/` (Cloudflare bot protection blocks requests)
- **CORRECT Endpoint:** `https://api.getsong.co/search/`
- **Query Format:** `type=both&lookup=song:{title} artist:{artist}&limit=1`
- **Status:** Fixed in code, working correctly
- **Returns:** BPM, Key (e.g., "D"), Mode, Time Signature (e.g., "4/4"), Artist genres

**❌ Spotify Playlist Import Rate Limits**
- **Issue:** Spotify-created playlists (e.g., "This Is {Artist}") are rate-restricted
- **Workaround:** Only import genuine public user-created playlists
- **Status:** Known limitation, documented

**✅ Deezer BPM (IMPLEMENTED — Session 18, 20 Aug 2026)**
- **Status:** Built 10 July 2026 (Session 15), left uncommitted, re-validated and committed 20 August 2026 (Session 18)
- **Behavior:** Deezer is now the primary BPM source for Pipeline 1 (`enrichmentService.js`); GetSongBPM is the fallback
- **Impact:** Improves BPM accuracy/coverage over GetSongBPM alone

**⏳ MusicBrainz Songwriters (NOT IMPLEMENTED)**
- **Status:** APIs available but not integrated
- **Fields Affected:** All `songwriters_*` fields except `songwriters_wikipedia`
- **Priority:** Medium (Wikipedia provides some songwriter data)

---

## 7. FIELD RULES

### 7.1 Field Source Types

**API-Sourced (Not Editable):**
- Core song data: `title`, `artist`, `year`
- Spotify: `spotifyTrackId`, `spotifyPopularity`
- Chart data: `chartPeakUs`, `chartPeakUk`, `top10`, `top40`
- Media links: `youtubeUrl`, `spotifyTrackId`

**Primary Source (Manual CSV, Editable in Admin):**
- Teaching metadata: `level`, `souKeys`, `teachingNotes`, `strumStyle`, `fingerpickingStyle`
- Materials status: `songSheetStatus`, `tabStatus`

**Override (Manual First, API Fallback, Editable):**
- Musical attributes: `originalKey`, `mode`, `timeSignature`, `bpm`, `tempoLabel`, `numChords`, `chords`, `chordNumerals`
- Release info: `releaseDate`

**Augment (Merge Manual + API, Editable):**
- `songwriters` — Manual CSV + Wikipedia + MusicBrainz (when implemented)
- `genre` — Manual CSV + merged API genres
- `tags` — Manual CSV + Last.fm tags

### 7.2 Field Edit Permissions (in ManageSOUDatabase component)

Each column has a `source` metadata property:
- **`source: 'API'`** → Display only (lock icon, not editable)
- **`source: 'Primary'`** → Fully editable (CSV is authority)
- **`source: 'Override'`** → Editable (manual value overrides API)
- **`source: 'Augment'`** → Editable (merged data from multiple sources)

Example from code:
```javascript
title: { label: 'Title', editable: false, source: 'API' }
level: { label: 'Level', editable: true, source: 'Primary' }
originalKey: { label: 'Original Key', editable: true, source: 'Override' }
songwriters: { label: 'Songwriter', editable: true, source: 'Augment' }
```

### 7.3 Field Transformation: snake_case ↔ camelCase

**Database (SQLite):** Uses `snake_case` field names
- Example: `bpm_best`, `release_date_consolidated`, `chart_peak_us`

**API Response:** Converts to `camelCase` via `toFrontendFormat()`
- Example: `bpm`, `releaseDate`, `chartPeakUs`

**Mapping Rules (server.js lines 169-398):**
```javascript
const fieldMappings = {
  'bpm_best': 'bpm',                           // bpm_best → bpm
  'key_best': 'keyBest',                       // key_best → keyBest
  'release_date_consolidated': 'releaseDate',  // release_date_consolidated → releaseDate
  'chart_peak_position': 'chartPeak',          // chart_peak_position → chartPeak
  'genres_best': 'genre',                      // genres_best → genre
  // ... 50+ more mappings
};
```

**Special Transformations:**
- `materials_json` (string) → `materials` (parsed array)
- `top_10` / `top_40` (0/1) → boolean `true`/`false`
- `tags` — Merges `tags_lastfm_track` and `tags_lastfm_artist`, converts string to array
- `releaseDate` — Smart waterfall logic, standardizes to "9 Jul 1972" format

---

## 8. FILE STRUCTURE

```
SOU App/
├── .github/
│   └── copilot-instructions.md         # Points to this MASTER_ARCHITECTURE.md
├── materials-server/                   # Backend (Node.js/Express)
│   ├── .env                            # Environment config (PORT=3002)
│   ├── server.js                       # Main Express app (1,035 lines)
│   ├── dbManager.js                    # SQLite operations (507 lines)
│   ├── enrichmentService_sqlite.js     # Enrichment pipeline (1,104 lines)
│   ├── spotifyService.js               # Spotify API client (227 lines)
│   ├── lastfmService.js                # Last.fm API client (196 lines)
│   ├── chartService.js                 # Soundcharts API client
│   ├── wikipediaChartScraper.js        # Wikipedia scraper
│   ├── materialsWatcher.js             # PDF folder watcher (315 lines)
│   ├── auth.js                         # Auth middleware (bypassed in dev)
│   ├── csvManager.js                   # Legacy CSV operations (replaced by dbManager)
│   ├── fieldMapper.js                  # Field name conversions
│   ├── errorHandler.js                 # Logging and retry logic
│   ├── config.js                       # Centralized paths and rate limits
│   ├── schema.sql                      # SQLite schema (372 lines, 239 columns)
│   ├── migrateCSVToSQLite.js           # Migration script (536 lines)
│   ├── scanMaterialsToDatabase.js      # PDF scanner script
│   ├── routes/
│   │   └── seed.js                     # Seed database API routes
│   ├── services/
│   │   └── seedService.js              # Seed database operations
│   ├── data/
│   │   ├── sou_songs.db                # SQLite database (1.2MB, 215 songs)
│   │   └── expanded_seed_base.json     # Seed database (47K songs)
│   ├── logs/
│   │   ├── backend.log
│   │   └── frontend.log
│   └── package.json
├── sou-song-browser/                   # Frontend (React 18)
│   ├── .env                            # Environment config (PORT=3000, REACT_APP_API_URL)
│   ├── public/
│   ├── src/
│   │   ├── components/
│   │   │   ├── AdminDashboard.js       # Dashboard (369 lines)
│   │   │   ├── ManageSOUDatabase.js    # Main table (1,489 lines)
│   │   │   ├── SeedDatabaseSearch.js   # Search 47K songs (367 lines)
│   │   │   ├── BulkAddSongs.js         # Bulk import
│   │   │   ├── SongEditor.js           # Edit modal
│   │   │   ├── AdminLayout.js          # Navigation menu
│   │   │   └── *.css                   # Component styles
│   │   ├── App.js
│   │   └── index.js
│   ├── build/                          # Production build output (npm run build)
│   └── package.json
├── Song Database/
│   ├── School of Uke Song Sheets Database.csv  # Teaching CSV (212 songs, SOURCE OF TRUTH for teaching data)
│   ├── data/
│   │   └── songdb_master_v2_enriched.csv       # Intermediate enriched CSV (legacy)
│   └── tools/                          # Python enrichment scripts (offline, not used by app)
├── scripts/
│   └── sanitize_paths.js               # Utility scripts
├── logs/
│   ├── backend.log
│   └── frontend.log
├── docs/                               # (To be created for archived MD files)
│   └── archive/
├── keep-servers-alive.sh               # Auto-restart monitor (currently failing)
├── start-all.sh                        # Start both servers in background
├── START_APP.sh                        # Startup script
├── STOP_APP.sh                         # Shutdown script
├── package.json                        # Root package (npm start runs both servers)
├── vercel.json                         # Vercel deployment config (frontend only)
├── MASTER_ARCHITECTURE.md              # THIS FILE (single source of truth)
├── API_DATAFIELDS_REFERENCE.md         # Field reference (1 Dec 2025)
├── ENRICHMENT_SERVICE_SUMMARY.md       # Enrichment docs
├── DATABASE_TERMINOLOGY.md             # Database names reference
├── MATERIALS_MANAGEMENT_SYSTEM.md      # PDF management docs
├── ADMIN_REORGANIZATION_COMPLETE.md    # Admin dashboard docs
├── START_SERVERS.md                    # Startup instructions
└── [50+ other MD files]                # See Section 14 for obsolete files
```

---

## 9. ADMIN DASHBOARD — CONFIRMED STATUS

### 9.1 AdminDashboard.js (369 lines)

**Status:** ✅ Complete and fully functional

**Features:**
- Stats display: Total Songs, Songs with PDFs, Recently Added, Seed Database Size
- Routing to sub-pages (Seed Search, Manage Database, Settings)
- Fetches data from `/api/songs` and `/api/seed/size`
- Logout handler (bypassed in dev, redirects to login)

**Verified Working:**
- Component renders without errors
- API calls succeed (tested 25 May 2026)
- Stats display correctly (215 songs in SOU DB, 47K in seed DB)

---

### 9.2 ManageSOUDatabase.js (1,489 lines)

**Status:** ✅ Complete and production-ready (enterprise-grade component)

**Features:**
1. **Data Table Display:**
   - 30+ columns with metadata (label, width, group, searchable, sortable, editable, source)
   - Column groups: Core, Classification, Teaching, Musical, Credits, Popularity, Media
   - Dynamic column rendering based on visibility settings

2. **Inline Editing:**
   - Click any editable cell to enter edit mode
   - Auto-save on blur (calls `PUT /api/songs/:id`)
   - Visual feedback (highlight edited cell)
   - Field validation based on `source` type (Primary/Override/Augment/API)
   - API-sourced fields show lock icon (not editable)

3. **Bulk Operations:**
   - Multi-select via checkboxes
   - "Select All" / "Deselect All" buttons
   - Bulk Edit modal (edit multiple songs at once)
   - Calls `POST /api/songs/bulk-update`

4. **Column Management:**
   - Show/hide columns via settings panel
   - Drag-to-reorder columns
   - Column visibility persists in state

5. **Saved Views:**
   - Save current column configuration (visibility + order)
   - Load saved views from dropdown
   - View names stored in state

6. **Search & Filter:**
   - Search by specific field (dropdown selector)
   - Search across all fields (option)
   - Real-time filtering

7. **Sorting:**
   - Click column header to sort
   - Toggle ascending/descending
   - Sort indicator (▲ ▼)

**Backend Integration (Verified):**
- `GET /api/songs` — Fetches all songs (verified in server.js line 402)
- `PUT /api/songs/:id` — Updates single song (verified line 413)
- `POST /api/songs/bulk-update` — Bulk edit (verified line 443)
- All endpoints exist and function correctly

**Known Issues:** None. Component is production-ready.

---

### 9.3 SeedDatabaseSearch.js (367 lines)

**Status:** ✅ Complete and fully functional

**Features:**
- Search 47K+ songs in `expanded_seed_base.json`
- Filter by: Artist name, Year range, Genre, Popularity threshold
- Results table with: Title, Artist, Year, Popularity%, Genres
- Multi-select checkboxes
- "Select All" / "Deselect All" buttons
- **"Promote to SOU Database" button:**
  - Sends selected Spotify IDs to `POST /api/seed/promote`
  - Backend runs full enrichment pipeline
  - Adds songs to SQLite database
  - Returns success/failure count

**Backend Integration (Verified):**
- `POST /api/seed/search` — Search seed database (verified in routes/seed.js)
- `POST /api/seed/promote` — Promote songs with enrichment (verified)
- Both endpoints exist and work correctly

**Workflow:**
1. User searches seed database
2. Results display in table
3. User selects songs via checkboxes
4. User clicks "Promote to SOU Database"
5. Backend enriches songs (Wikipedia, GetSongBPM, Spotify, Last.fm, YouTube)
6. Backend adds enriched songs to SQLite
7. Frontend shows success alert with count

---

### 9.4 Other Admin Components

**BulkAddSongs.js:**
- Status: ✅ Complete
- Purpose: Bulk import from CSV to SOU database
- Features: File upload, CSV parsing, duplicate detection, bulk insert

**SongEditor.js:**
- Status: ✅ Complete
- Purpose: Modal for editing individual song
- Features: Form with all editable fields, field validation, save/cancel buttons

**AdminLayout.js:**
- Status: ✅ Complete
- Purpose: Navigation menu wrapper
- Features: Expandable sub-menu items, routing, active page highlighting

---

## 10. PDF MANAGEMENT

### 10.1 Current PDF Storage (Local)

**Location:** Google Drive (mounted locally)  
**Path:** `/Users/matthew/Library/CloudStorage/GoogleDrive-info.schoolofuke@gmail.com/My Drive/School of Uke Lesson Content/Lesson Content Tutor Access Only - School of Uke /Song Sheets PDF ONLY - School of Uke`

**Structure:**
```
Song Sheets PDF ONLY - School of Uke/
├── (You're the) Devil in Disguise - Elvis Presley (1963)/
│   ├── Devil in Disguise - Elvis Presley - Key F.pdf
│   ├── Devil in Disguise - Elvis Presley - Key G.pdf
│   └── Devil in Disguise TAB - Key F.pdf
├── 5 Years Time - Noah and the Whale (2008)/
│   └── 5 Years Time - Noah and the Whale - Key C.pdf
└── [213 more song folders]/
```

**Scanner:** `materials-server/scanMaterialsToDatabase.js`
- Scans Google Drive folder recursively (depth 1)
- Parses filenames to extract: title, artist, year, key, isTab
- Fuzzy matches to database songs (handles artist variations)
- Updates SQLite database with paths and metadata

**Database Fields:**
- `song_sheet_status` — "Yes" / NULL
- `tab_status` — "Yes" / NULL
- `song_sheet_path` — Full path to first sheet PDF
- `melody_tab_path` — Full path to first TAB PDF
- `song_sheet_rel_path` — Relative path (preferred for API)
- `melody_tab_rel_path` — Relative path (preferred for API)
- `materials_json` — JSON array of all versions:
  ```json
  [
    {
      "filename": "Song - Artist - Key C.pdf",
      "relativePath": "Song - Artist (Year)/Song - Artist - Key C.pdf",
      "absolutePath": "/full/path/to/file.pdf",
      "isTab": false,
      "key": "C"
    }
  ]
  ```

**API Serving:**
- `GET /materials/*` — Serves PDF file by relative path
- Materials base URL: `http://localhost:3002` (local dev)

**Scanner Results (Last Run):**
- Folders scanned: 214
- PDFs processed: 363
- Songs matched: 197
- Songs with sheets: 195
- Songs with TABs: 49

---

### 10.2 Deployment Target (Cloudflare R2) — ✅ IMPLEMENTED

**Platform:** Cloudflare R2 (S3-compatible object storage)  
**Reason:** Free tier, no egress fees, S3-compatible API  
**Bucket Name:** `sou-song-sheets`  
**Public URL base:** `https://pub-e43364bf5aa34598832e4b2e860e074d.r2.dev`

**Status (confirmed 26 Aug 2026):**
- Song sheets (`song_sheet_path`): 190/191 migrated to R2. 1 remaining (Espresso — no local file located).
- TAB files (`melody_tab_path`): 49/49 migrated to R2 as of today's session (previously 0/49 — see Session Log).
- Migration scripts: `materials-server/uploadToR2.py` (song sheets), `materials-server/uploadTabsToR2.py` (TABs — distinct `{song_id}_tab.pdf` object key, same bucket, no collisions).
- `/materials/*` endpoint retired — intentionally returns `410 Gone`; frontend reads `songSheetPath`/`melodyTabPath` directly.

**R2 Configuration:**
- Public read access for PDF files
- CORS enabled for frontend domain
- S3-compatible API for programmatic access

---

## 11. STARTUP COMMANDS

**All startup scripts verified to exist as files (25 May 2026):**

### 11.1 keep-servers-alive.sh

```bash
./keep-servers-alive.sh
```

**Status:** ⚠️ Currently failing — servers restart continuously  
**Purpose:** Start both servers with auto-restart on crash  
**Monitors:** Every 10 seconds  
**Logs:** `logs/backend.log`, `logs/frontend.log`  
**Stop:** Press Ctrl+C

---

### 11.2 NPM Script (Parallel Start)

```bash
npm start
```

**Location:** Root `package.json`  
**Command:** Uses `concurrently` to start both servers in parallel  
**Output:** Color-coded (backend = blue, frontend = green)  
**Stop:** Press Ctrl+C once

---

### 11.3 start-all.sh

```bash
./start-all.sh
```

**Purpose:** Start both servers in background with log files  
**Features:**
- Clears stuck ports automatically
- Saves PIDs to files
- Displays URLs and log locations
**Stop:** Press Ctrl+C (or kill PIDs)

---

### 11.4 Manual (Two Terminals)

**Terminal 1 (Backend):**
```bash
cd materials-server
npm start
```

**Terminal 2 (Frontend):**
```bash
cd sou-song-browser
npm start
```

**Ports:**
- Backend: `http://localhost:3002` (from `materials-server/.env`)
- Frontend: `http://localhost:3000` (from `sou-song-browser/.env`)

---

## 12. DEPLOYMENT ARCHITECTURE (AS IMPLEMENTED)

**Target (achieved):** Railway (backend) + Vercel (frontend) + Cloudflare R2 (PDFs) — this documents the setup as actually built and deployed, not a forward plan. For current live status and URLs see Section 3; for any future infrastructure changes see `PRODUCT_ROADMAP.md`.

**AWS Lambda approach is ABANDONED** — Too complex, 6 months of failed attempts. Do not reference `README_AWS.md`.

---

### 12.1 Frontend Deployment (Vercel)

**Platform:** Vercel (static site hosting)  
**Repository:** Connect GitHub repo to Vercel  
**Build Command:** `npm run build` (in `sou-song-browser/`)  
**Output Directory:** `build/`  
**Framework:** Create React App (auto-detected)

**Environment Variables (Vercel Dashboard):**
```
REACT_APP_API_URL=https://sou-materials-server.up.railway.app
REACT_APP_MATERIALS_URL=https://r2-bucket-url.cloudflare.com
```

**Deployment Config:** `vercel.json` (already exists at root)
```json
{
  "version": 2,
  "builds": [{ "src": "package.json", "use": "@vercel/static-build" }],
  "routes": [
    { "src": "/static/(.*)", "dest": "/static/$1" },
    { "src": "/(.*)", "dest": "/index.html" }
  ]
}
```

**Steps:**
1. Push code to GitHub
2. Connect repo to Vercel
3. Set root directory to `sou-song-browser/`
4. Add environment variables
5. Deploy (auto-builds on push)

**Domain:** (TBD — Vercel provides default subdomain, can add custom domain)

---

### 12.2 Backend Deployment (Railway)

**Platform:** Railway (Heroku alternative, easier than AWS)  
**Repository:** Connect GitHub repo to Railway  
**Start Command:** `npm start` (in `materials-server/`)  
**Port:** Auto-assigned by Railway (`$PORT` env var)

**Environment Variables (Railway Dashboard):**
```
PORT=3002  # (Railway may override with its own $PORT)
NODE_ENV=production
FRONTEND_URL=https://sou-app.vercel.app
SESSION_SECRET=<generate-random-string>
SPOTIFY_CLIENT_ID=<from-spotify-dashboard>
SPOTIFY_CLIENT_SECRET=<from-spotify-dashboard>
LASTFM_API_KEY=<from-lastfm-account>
YOUTUBE_API_KEY=<from-google-cloud-console>
GETSONGBPM_API_KEY=<from-getsongbpm>
SOUNDCHARTS_APP_ID=<from-soundcharts>
SOUNDCHARTS_API_KEY=<from-soundcharts>
MATERIALS_PATH=  # Empty (PDFs will be on R2, not local)
```

**Database:** SQLite file persistence
- Railway provides ephemeral filesystem (resets on deploy)
- **Solution:** Use Railway Volumes to persist `data/sou_songs.db`
- Or: Migrate to PostgreSQL (Railway provides managed Postgres)

**Railway Volume shadow pattern:** The Railway Volume is mounted at `/app/data/`. Any file committed to `data/` in the repo will be shadowed (invisible) if the Volume is mounted there. For files that must survive deploys AND be present from the first boot, the pattern used in this project is:
1. Commit a bundled copy **outside** `data/` (e.g. `sou_songs_seed.db`, `expanded_seed_base_seed.json` at the repo root)
2. On startup, if the target file in `data/` is missing, copy the bundled source into it
3. Subsequent boots skip the copy (file already present in volume)

**Large static data files (future consideration):** `expanded_seed_base.json` (18MB, 47K songs) is currently committed to the repo and bootstrapped onto the volume on first boot. This is fine while the catalog is largely static. If frequent updates are needed (new playlist imports, genre enrichment passes), consider moving it to Cloudflare R2 alongside the songsheet PDFs and fetching it via HTTP in `seedService.js` — avoids committing large JSON diffs and makes updates instant without a redeploy.

**Steps:**
1. Push code to GitHub
2. Connect repo to Railway
3. Set root directory to `materials-server/`
4. Add environment variables
5. Add Volume for `data/` directory (or migrate to Postgres)
6. Deploy

**Domain:** Railway provides default subdomain (e.g., `sou-materials-server.up.railway.app`)

---

### 12.3 Session Persistence: MemoryStore Limitation

**Current state:** `express-session` is configured with no `store:` option, so it uses the built-in default `MemoryStore`. Session data lives entirely in the Node.js process heap.

**Consequence:** Every Railway redeploy (new container) starts with an empty MemoryStore. All browser session cookies referencing old session IDs become invalid. Users see 401 / "Failed to load songs database" on the next page load without any visible error message, as if auth broke.

**Workaround (current):** Log out and log back in after each deploy. One-admin scenario makes this tolerable.

**When to fix:** When a second tutor account is added, or when deploy-frequency makes the logout disruptive.

**Migration path:**
- `connect-sqlite3` — simplest, backed by the existing `sou_songs.db`. Add `npm install connect-sqlite3`, pass `store: new SQLiteStore({db: 'data/sou_songs.db', table: 'sessions'})` to `session()`. Zero new infrastructure.
- Postgres sessions — natural fit once the song DB migrates to Postgres (Railway Postgres add-on), using `connect-pg-simple`.

**Do NOT implement now.** Documented at `server.js` line ~50 with a `NOTE:` comment.

---

### 12.4 PDF Storage (Cloudflare R2)

**Platform:** Cloudflare R2 (S3-compatible object storage)  
**Free Tier:** 10 GB storage, unlimited egress (no bandwidth fees)

**Steps:**
1. Create Cloudflare account
2. Create R2 bucket (e.g., `sou-song-sheets`)
3. Configure public read access
4. Enable CORS for Vercel frontend domain
5. Upload PDFs from Google Drive to R2 (preserve folder structure)
6. Update database `materials_json` paths (replace local paths with R2 URLs)
7. Update frontend `REACT_APP_MATERIALS_URL` to R2 bucket URL

**R2 Bucket URL Format:**
```
https://pub-<bucket-id>.r2.dev/Song%20Name%20-%20Artist%20(Year)/file.pdf
```

**Backend Changes:**
- Remove `/materials/*` proxy endpoint (frontend fetches directly from R2)
- Or keep proxy for access control (requires signed URLs)

---

### 12.4 Deployment Order (as executed)

1. **Set up Cloudflare R2** (upload PDFs, get bucket URL)
2. **Deploy backend to Railway** (connect repo, add env vars, configure Volume/Postgres)
3. **Deploy frontend to Vercel** (connect repo, add env vars with Railway/R2 URLs)
4. **Test full stack** (verify API calls, PDF access, enrichment pipeline)
5. **Configure custom domain** (optional, point DNS to Vercel)
6. **Enable SSL** (automatic on Vercel and Railway)
7. **Monitor logs** (Railway dashboard, Vercel dashboard)

---

## 13. PRODUCT ROADMAP

Forward product direction, phased sequencing, and priorities are maintained exclusively in `PRODUCT_ROADMAP.md` — see that document for the current phase plan (Studio tool stabilisation → PDF/legacy content extraction → Digital Songsheet Builder → Curriculum integration → Pedagogy Layer). This section intentionally does not duplicate phase numbers or priority claims; see the Governance rule at the top of this document.

---

## 14. OUTDATED FILES

**These MD files are now superseded or obsolete. Move to `/docs/archive/` folder:**

### 14.1 Superseded by MASTER_ARCHITECTURE.md

| File | Reason | Superseded By |
|------|--------|---------------|
| `PROJECT_STATUS.md` | Outdated status (claimed deployment needed, but app works locally) | This document (Section 3) |
| `PROJECT_FEATURES_AND_ROADMAP.md` | Empty file (0 bytes) | `PRODUCT_ROADMAP.md` |
| `README_AWS.md` | AWS Lambda approach abandoned (6 months of failed attempts) | This document (Section 12 — Railway deployment) |
| `ADMIN_REORGANIZATION_PLAN.md` | Implementation complete (24 Nov 2025) | `ADMIN_REORGANIZATION_COMPLETE.md` |
| `ADMIN_REORGANIZATION_SUMMARY.md` | Duplicate of COMPLETE.md | `ADMIN_REORGANIZATION_COMPLETE.md` |
| `ENRICHMENT_REFACTOR_PLAN.md` | Refactor complete (Dec 2025) | `ENRICHMENT_SERVICE_SUMMARY.md` |

### 14.2 Outdated Content

| File | Reason | Last Updated |
|------|--------|--------------|
| `Song Database/SOU_SongDB_STATUS.md` | Very outdated (Oct 2025, before SQLite migration) | Oct 2025 |
| `CHART_AGGREGATION_SUMMARY.md` | Chart enrichment complete | Unknown |
| `BULK_ADD_FIXES_COMPLETE.md` | Component complete | Nov 2025 |

### 14.3 Keep as Reference (Do Not Archive)

| File | Reason |
|------|--------|
| `API_DATAFIELDS_REFERENCE.md` | Comprehensive field reference (1 Dec 2025) — still accurate |
| `ENRICHMENT_SERVICE_SUMMARY.md` | Enrichment docs with API priorities — still accurate |
| `DATABASE_TERMINOLOGY.md` | Database naming reference — still useful |
| `MATERIALS_MANAGEMENT_SYSTEM.md` | PDF management docs (complete, 3 Dec 2025) — still accurate |
| `ADMIN_REORGANIZATION_COMPLETE.md` | Admin dashboard implementation record — historical reference |
| `START_SERVERS.md` | Startup instructions — still accurate (servers fail but docs correct) |
| `FIELD_MAPPING_ARCHITECTURE.md` | Field transformation reference — still useful |

### 14.4 Archive Action

**Create `/docs/archive/` folder and move these 9 files:**
1. `PROJECT_STATUS.md` → Superseded by Section 3
2. `PROJECT_FEATURES_AND_ROADMAP.md` → Superseded by `PRODUCT_ROADMAP.md` (empty file)
3. `README_AWS.md` → AWS approach abandoned
4. `ADMIN_REORGANIZATION_PLAN.md` → Implementation complete
5. `ADMIN_REORGANIZATION_SUMMARY.md` → Duplicate
6. `ENRICHMENT_REFACTOR_PLAN.md` → Refactor complete
7. `Song Database/SOU_SongDB_STATUS.md` → Very outdated (Oct 2025)
8. `CHART_AGGREGATION_SUMMARY.md` → Work complete
9. `BULK_ADD_FIXES_COMPLETE.md` → Component complete

**Do not delete** — These files contain historical context that may be useful for understanding past decisions.

---

## 15. DECISIONS LOG

### 25 May 2026 — Deployment Target Changed

**Decision:** Use Railway (backend) + Vercel (frontend) instead of AWS Lambda  
**Reason:** AWS too complex, 6 months of failed deployment attempts, Railway/Vercel much simpler  
**Impact:** Abandon `README_AWS.md`, update deployment plan in this document  
**Alternatives Considered:** Heroku (expensive), AWS Lambda (too complex), DigitalOcean (requires more DevOps), Railway (chosen — easiest)

---

### 25 May 2026 — PDF Storage Target Set

**Decision:** Use Cloudflare R2 for PDF storage  
**Reason:** Free tier (10 GB storage), no egress fees, S3-compatible API, simpler than AWS S3  
**Impact:** PDFs move from local Google Drive to cloud storage, frontend fetches directly from R2  
**Alternatives Considered:** AWS S3 (expensive egress), Google Cloud Storage (complex), Backblaze B2 (less known), Cloudflare R2 (chosen)

---

### 25 May 2026 — Admin Dashboard Confirmed Complete

**Decision:** Do not rebuild ManageSOUDatabase.js or other admin components  
**Reason:** Comprehensive code inspection reveals 1,489-line production-ready component with all features working  
**Impact:** Focus on deployment, not rebuilding existing functionality  
**Evidence:** Component imports verified, API endpoints verified, inline editing working, bulk operations working

---

### 3 December 2025 — SQLite Migration Completed

**Decision:** Migrate from CSV (csvManager.js) to SQLite (dbManager.js)  
**Reason:** Better performance, ACID compliance, concurrent writes, easier querying  
**Impact:** 215 songs migrated to `sou_songs.db`, csvManager.js deprecated but not removed (legacy)  
**Migration Script:** `migrateCSVToSQLite.js` (536 lines)  
**Result:** 1.2 MB database file, 177 columns, WAL mode enabled

---

### November 2025 — Spotify Audio Features Parked

**Decision:** Do not use Spotify Audio Features endpoint until OAuth implemented  
**Reason:** Client Credentials flow returns 403 Forbidden, requires user OAuth (not feasible for batch enrichment)  
**Impact:** Use GetSongBPM for BPM/Key/Mode/Time Signature instead  
**Blocked Fields:** `spotify_danceability`, `spotify_energy`, `spotify_valence`, `spotify_mode`, `spotify_key`, `spotify_time_signature`, `spotify_bpm`

---

### November 2025 — GetSongBPM Endpoint Corrected

**Decision:** Use `api.getsong.co` instead of `api.getsongbpm.com`  
**Reason:** `api.getsongbpm.com` is behind Cloudflare bot protection, returns HTML instead of JSON  
**Impact:** GetSongBPM now working correctly, provides BPM, Key, Mode, Time Signature, Artist genres  
**Documentation:** See `ENRICHMENT_SERVICE_SUMMARY.md` for details

---

### 24 November 2025 — Admin Dashboard Reorganized

**Decision:** Restructure admin menu, separate Seed Database operations from SOU Database management  
**Reason:** Clarity — admins were confused about which database they were editing  
**Impact:** Menu now has "Song Discovery" (seed) and "Manage SOU Database" (teaching) as separate sections  
**Components Created:** `SeedDatabaseSearch.js` (367 lines), `BulkExport.js`, updated `AdminLayout.js`  
**Documentation:** `ADMIN_REORGANIZATION_COMPLETE.md`

---

### 3 December 2025 — Materials Scanner Completed

**Decision:** Scan Google Drive folder and populate SQLite `materials_json` field with all PDF versions  
**Reason:** Support multiple keys/arrangements per song, easier frontend display  
**Impact:** 197 songs matched, 195 with sheets, 49 with TABs, `materials_json` stores array of all versions  
**Script:** `scanMaterialsToDatabase.js`  
**Documentation:** `MATERIALS_MANAGEMENT_SYSTEM.md`

---

### 25 May 2026 — Server Initialization Best Practice

**Decision:** Do not attempt to connect to external APIs during service module initialization  
**Reason:** Causes server startup to hang before app.listen() is reached, blocking all deployment and testing  
**Impact:** Services must use lazy initialization — defer API connections until first actual use, not during require()  
**Example:** searchService, chartService, enrichmentService_sqlite should initialize connection properties to null and only connect when methods are called  
**Status:** Identified during debugging session 25 May 2026, not yet implemented in codebase

---

### 13 June 2026 — Teaching Field Corruption Fix (CSV Column Shift)

**Decision:** Always use a proper CSV parser (Python `csv.DictReader`, Node `csv-parse`) for any import touching teaching fields. Never `line.split(',')`.  
**Reason:** `importTeachingData.js` naive split corrupted `strum_style` (65 rows), `fingerpicking_style` (41 rows), `chord_numerals` (17 rows) by shifting columns right at every internal comma in quoted fields.  
**Fix:** 3 idempotent `UPDATE` statements in `dbManager.runMigrations()`. Applied to local DB directly; applied to Railway production DB via startup migration on next deploy.  
**Commit:** `2543dfb` on sou-backend

---

### 13 June 2026 — Enrichment Must Never Overwrite Non-Null Fields With Null

**Decision:** All enrichment code paths must guard against writing `null`/`undefined` over existing DB values. Two defence layers:
1. **`dbManager.updateSong()`** strips `null` and `undefined` from the updates object before building the SQL `UPDATE` statement. This is a last-resort safety net for any caller.
2. **`enrichmentService_sqlite.js`** guards each individual field assignment with `!song.fieldName` before adding to `enrichmentData`. This prevents accumulating useless API calls even if the DB safety net would catch them.

**Reason:** Several early enrichment runs wrote `null` to `bpm_best`, `wikipedia_composition`, `wikipedia_recording`, and other fields, silently destroying previously-enriched data.  
**Impact:** Any enrichment caller wanting to intentionally blank a field must send `""` (empty string), never `null`. Callers using React controlled inputs already satisfy this (React always sends `e.target.value` as string).  
**Exception:** Last.fm `plays`/`listeners` are intentionally always-overwrite (no `!song.field` guard) because they are fresh live metrics that should update on every enrich run.  
**Commits:** `ede2cc2` (dbManager + enrichmentService_sqlite guards)

---

### 13 June 2026 — Studio Layout: VS Code-style Docked Chat Panel

**Decision:** The AI Studio will be implemented as a persistent docked chat panel in `AdminLayout`, following the VS Code + GitHub Copilot reference model — a resizable panel docked to the right side of the screen that persists across admin navigation changes.  
**Reason:** Keeps the AI assistant always accessible without losing conversation context during navigation; matches the tutor's mental model of a creative workspace tool rather than a separate page.  
**Alternatives considered:** Separate full-page view (loses context on nav), modal overlay (too intrusive), floating widget (hard to resize).  
**Impact:** `AdminLayout.js` will need a resizable right panel; conversation state must lift to layout level so it survives page switches.  
**Status:** Decision made, not yet implemented. Blocked on resolving 4 open questions in `PROJECT_FEATURE_MAP.md` §Studio.  
**Trigger for implementation:** When Matthew resumes Studio work.

---

### 6 June 2026 — Node 18 Required for CRA Builds

**Decision:** All development for this project must use Node.js 18 (managed via nvm), not Node 22  
**Reason:** `npm run build` hangs indefinitely on Node 22 due to ESM deadlock in `fork-ts-checker-webpack-plugin` (via cosmiconfig) and `workbox-webpack-plugin` (via common-tags)  
**Impact:** Switch to Node 18 before any frontend build or local development. Use `nvm use 18`. `better-sqlite3` must be rebuilt after any Node version switch (`npm rebuild better-sqlite3`)  
**Environment:** nvm installed at `~/.nvm`, Node 18.20.8 confirmed working

---

### 8 June 2026 — Admin Panel Authentication Secured

**Decision:** Gate `bypassAuth` middleware behind `NODE_ENV !== 'production'`  
**Reason:** Audit revealed the bypass was unconditional — production admin panel had zero authentication, anyone who found `/admin` had full write access  
**Impact:** In production (Railway), `requireAuth` now enforces session authentication. Login requires password set in `ADMIN_PASSWORD` env var.  
**Local dev:** Unchanged — auto-authenticates as before  
**Commit:** `dc4709c` on sou-backend

---

### 13 June 2026 — SEED_PATCHES Pattern for Railway Volume JSON Updates

**Decision:** Future additions to `expanded_seed_base.json` on Railway must go through `SEED_PATCHES` in `seedService.js`, not by committing to `data/expanded_seed_base.json`.  
**Reason:** Railway Volume mounts at `/app/data/` and shadows the entire directory. Committed files in `data/` are invisible to a running deployment that already has a volume. The existing copy-on-boot guard only fires when the file is absent, not when it is outdated — so a second push of a modified `data/expanded_seed_base.json` is silently ignored.  
**Fix:** `SEED_PATCHES` array in `seedService.js` checked synchronously at module load. Each entry checked by `spotifyId`; missing entries appended to the live volume file and written back. Idempotent — no-op after first application.  
**Pattern:** Mirrors `runMigrations()` in `dbManager.js` (same root cause — Railway Volume data can only be patched at runtime, not replaced by deploy).  
**Maintenance:** All future manually-added canonical seed entries must be added to `SEED_PATCHES`. `expanded_seed_base_seed.json` (the bootstrap copy outside `data/`) should also be updated so fresh-volume deployments start consistent.  
**Commit:** `7b8075d` on sou-backend

---

### 8 June 2026 — Railway Volume Seeding via Bundled Seed DB

**Decision:** Bundle `sou_songs_seed.db` at the repo root (outside `data/`) as a seed copy of the production database; `dbManager.getDb()` auto-copies it to the volume path on first boot if the volume is empty  
**Reason:** Railway Volumes are empty on first mount and completely shadow the `data/` directory, hiding the committed `sou_songs.db`. Copying a seed file outside `data/` ensures it is always accessible in the container image regardless of the volume mount.  
**Impact:** On each fresh Railway redeploy to an empty volume, the seed copy is used automatically. Subsequent deploys use the persisted volume copy (no overwrite).  
**Maintenance:** When schema changes or significant data is added locally, refresh the seed with `cp data/sou_songs.db sou_songs_seed.db && git add sou_songs_seed.db && git commit`  
**Commit:** `d5ba19b` on sou-backend

---

### 8 June 2026 — Seed Promote Writes to SQLite Not CSV

**Decision:** `POST /api/seed/promote` must use `dbManager.addSong()`, not `csvManager.createSong()`  
**Reason:** The app serves data from SQLite; writing to the CSV file means promoted songs never appear in the running app. csvManager is a legacy module retained for the original CSV-based workflow.  
**Impact:** Songs promoted via Search & Add or Bulk Add now appear immediately in the admin song list and the public song browser  
**Commit:** `e1c2c62` on sou-backend

---

### 6 June 2026 — SQLite Database Shipped in Git Repo for Railway

**Decision:** Include `sou_songs.db` (1.2 MB) in the `sou-backend` git repo for initial Railway deployment  
**Reason:** Simplest path to get a working deployment — no separate data migration step needed  
**Impact:** DB content is static at deploy time; enrichment script writes will be lost on Railway redeploy  
**Future Action Required:** ~~Set up a Railway Volume mounted at `/app/data/` for SQLite persistence if enrichment scripts need to run in production~~ ✅ **Done** — Railway Volume implemented and mounted at `/app/data/`, confirmed live in Section 3  
**Trade-off Accepted:** Acceptable for now since all enrichment runs locally; Railway is read-only API serving

---

## 16. REFERENCE FILES

**⚠️ WARNING:** Do not read all reference files at session start. Only read files relevant to the current task. Reading unnecessary files wastes context window and increases the risk of conflicting information. When in doubt about which file to read, check this table first. If a task spans multiple areas, read all relevant files before starting but confirm with Matthew if more than 4 reference files seem necessary for a single task — this may indicate the task is too broad and should be broken into smaller steps.

| Task Area | Files to Read | Why Important |
|-----------|---------------|---------------|
| **SESSION START (always read)** | MASTER_ARCHITECTURE.md | Single source of truth, read in full before anything |
| **ENRICHMENT PIPELINE** (read when doing any metadata fetching or enrichment) | API_DATAFIELDS_REFERENCE.md<br>ENRICHMENT_SERVICE_SUMMARY.md<br>API_SOURCES_REFERENCE.md<br>DATA_PIPELINE_INVENTORY.md<br>API_SETUP_GUIDE.md | Definitive field-by-field API priority table (Dec 1 most recent)<br>Corrected API priorities based on actual performance testing<br>Complete reference for all third-party APIs used<br>522 lines cataloguing all import, enrichment and export pipelines<br>API key configuration, read if any keys are missing or expired |
| **SONGWRITER PIPELINE** (read when touching songwriter data) | GENIUS_FIRST_PIPELINE.md<br>GENIUS_SETUP.md<br>SONGWRITER_PIPELINE_V2.md<br>SONGWRITER_API_RESEARCH.md | Explains why Genius is primary source<br>Genius API credentials and setup<br>Improved songwriter enrichment logic<br>API comparison research, context for decisions made |
| **GENRE PIPELINE** (read when touching genre data) | GENRE_PIPELINE_SETUP.md | 4-API genre enrichment pipeline |
| **BPM AND KEY** (read when touching tempo or key data) | GETSONGBPM_FIX.md | CRITICAL: documents corrected API endpoint api.getsong.co, previously a major bug source, do not use api.getsongbpm.com |
| **CHART AND POPULARITY DATA** (read when touching chart positions or popularity) | SOUNDCHARTS_SETUP.md<br>POPULARITY_APIS.md<br>POPULARITY_ENHANCEMENT_SUMMARY.md<br>ARCHITECTURE_POPULARITY_NEXT_STEPS.md | Chart position data via Soundcharts API<br>Notes on popularity scoring APIs<br>UI enhancements for popularity display<br>Roadmap for popularity scoring features |
| **MEDIA AND COVER ART** (read when touching images or media) | MEDIA_ENRICHMENT_STATUS.md<br>MEDIA_ENRICHMENT_PLAN.md | Current status: 203 of 212 songs have cover art<br>Plan for cover art and thumbnail enrichment |
| **SPOTIFY SPECIFICALLY** (read before any Spotify API work) | SPOTIFY_SETUP.md<br>SPOTIFY_AUDIO_FEATURES_ISSUE.md | Credentials and setup<br>CRITICAL: documents 403 error on audio features, do not attempt to use Spotify audio features without reading this first |
| **YOUTUBE SPECIFICALLY** (read before any YouTube API work) | YOUTUBE_SETUP.md | YouTube Data API setup, note 100 requests per day limit |
| **GENIUS SPECIFICALLY** (read before any Genius API work) | GENIUS_SETUP.md | Access token setup |
| **FIELD MAPPING AND DATA EDITING** (read when touching any data fields) | FIELD_MAPPING_ARCHITECTURE.md<br>DATABASE_TERMINOLOGY.md<br>DATABASE_INVENTORY.md | 858 lines, essential for any field mapping work<br>Definitions of all three databases and field rules<br>Complete inventory of all database files with paths and sizes |
| **PDF AND MATERIALS** (read when touching PDF serving or file management) | MATERIALS_MANAGEMENT_SYSTEM.md | Most recently updated file in project (Dec 3), covers PDF watcher |
| **SEARCH AND DISCOVERY** (read when touching search features) | SONG_SEARCH_ENGINE.md<br>UNIFIED_SEARCH_IMPLEMENTATION.md | 512 lines, multi-source fallback strategy<br>Unified advanced search with fuzzy chart parsing |
| **ADMIN DASHBOARD** (read when touching admin UI or backend integration) | ADMIN_REORGANIZATION_COMPLETE.md<br>ADMIN_BACKEND_INTEGRATION_COMPLETE.md<br>ADMIN_DASHBOARD.md | What was built and how it is structured<br>Backend API routes and integration details<br>User guide, useful for understanding intended behaviour |
| **FRONTEND AND STUDENT UI** (read when touching the tutor or student facing frontend) | STUDENT_UI_DATA_AUDIT.md<br>POPULARITY_ENHANCEMENT_SUMMARY.md | 342 lines, documents all 57 fields in frontend (Dec 1)<br>UI display of popularity and chart data |
| **SERVER AND INFRASTRUCTURE** (read when touching server config or startup) | SERVER_PERSISTENCE_FIX.md<br>CLEANUP_SUMMARY.md | Documents port conflict fix and keep-servers-alive setup<br>Documents what was cleaned up in materials-server |
| **TESTING** (read when writing or running tests) | TESTING_ADMIN_DASHBOARD.md<br>UNIT_TESTING_SUMMARY.md<br>START_TESTING.md | 626 lines, comprehensive admin test plan<br>Jest unit testing documentation<br>Quick testing checklist |
| **CODE QUALITY** (read when doing refactoring or cleanup) | CODE_AUDIT.md | Full codebase audit with cleanup recommendations |
| **PRODUCT VISION** (read when making any product or feature decisions) | ADMIN_INTERFACE_VISION.md | Original vision document, check for consistency with current roadmap |

**Song Database Folder Setup Guides:**  
These live in `Song Database/` folder and cover Python enrichment scripts specifically. Only read if working on Python pipeline scripts, not Node.js services.

---

## 17. SESSION LOG

### 25 May 2026 — Re-Entry Audit with Claude (Session 1)

**Attendees:** Matthew (user), Claude (AI assistant)  
**Duration:** ~2 hours  
**Context:** Matthew returned after 6-month absence (Nov 2025 → May 2026)

**Findings:**
- Confirmed backend port: 3002 (not 3001 as copilot-instructions.md claimed)
- Confirmed SQLite database populated: 1.2 MB, 215 songs, migrated 3 Dec 2025
- Confirmed admin dashboard complete: AdminDashboard.js (369 lines), ManageSOUDatabase.js (1,489 lines), SeedDatabaseSearch.js (367 lines)
- Identified ghost database file: Empty 0-byte `sou_songs.db` at wrong path
- Identified teaching data missing: 20+ fields empty in database (Level, SOU Keys, Chords, Teaching Notes)
- Resolved MD file contradictions: 55 MD files audited, categorized by topic, identified 9 for archival
- Traced data flow: SQLite → dbManager.getAllSongs() → toFrontendFormat() → GET /api/songs → React frontend

**Decisions:**
- Deployment target: Railway + Vercel (AWS Lambda abandoned)
- PDF storage: Cloudflare R2 (free tier, S3-compatible)
- Admin dashboard: Confirmed complete — do not rebuild

**Actions Taken:**
- Created `MASTER_ARCHITECTURE.md` (this document) as single source of truth
- Updated `.github/copilot-instructions.md` to reference this document
- Identified 9 MD files for archival (moved to `/docs/archive/`)
- Documented enrichment pipeline with definitive source order (Section 6)
- Documented deployment plan for Railway + Vercel (Section 12)

**Outstanding Issues:**
- Servers failing to start (keep-servers-alive.sh shows continuous restart loop) — cause unknown
- Teaching data not in database (needs import from CSV)
- Ghost database file should be deleted

**Next Steps:**
- Investigate server startup failures (check logs, dependencies, env vars)
- Import teaching data from `School of Uke Song Sheets Database.csv`
- Delete ghost database file at `materials-server/sou_songs.db` (0 bytes)
- Begin deployment to Railway + Vercel + Cloudflare R2

---

### 25 May 2026 — Teaching Data Import & Server Debugging (Session 2)

**Attendees:** Matthew (user), Claude (AI assistant)  
**Duration:** ~3 hours  
**Context:** Continuation session — resolved two critical deployment blockers

**Issue 1: Teaching Data Import**
- **Problem:** SQLite database migrated from enriched template CSV, not actual teaching CSV with tutor-maintained metadata
- **Root Cause:** Database created from `songdb_master_v2_enriched.csv` which lacked teaching fields (Level, SOU Keys, Chords, Teaching Notes, Strum Style, Fingerpicking Style)
- **Solution:** Created `importTeachingData.js` (203 lines) with fuzzy title/artist matching and duplicate prevention
- **Execution Results:**
  - 192 songs updated successfully (89.3%)
  - 2 songs not found in database (CSV formatting issues)
  - 18 songs failed: Missing `time_signature` column in schema
  - Fields imported: `level`, `sou_keys`, `num_chords`, `chords`, `chord_numerals`, `strum_style`, `fingerpicking_style`, `teaching_notes`, `song_sheet_status`, `tab_status`
- **Remaining Work:** Add `time_signature` column to schema, re-run import for 18 affected songs

**Issue 2: better-sqlite3 Architecture Mismatch**
- **Problem:** `dlopen(...better_sqlite3.node...): mach-o file, but is an incompatible architecture (have 'x86_64', need 'arm64e' or 'arm64')`
- **Root Cause:** Native module compiled for Intel x86_64, but Node.js running on Apple Silicon ARM64
- **Solution:** `npm uninstall better-sqlite3 && npm install better-sqlite3` (recompiled for ARM64 in ~1 minute)
- **Validation:** `node -e "require('better-sqlite3'); console.log('SQLite OK')"` → Success
- **Impact:** Fixed both import script execution AND underlying server crash issue

**Issue 3: Server Startup Hang Identified**
- **Problem:** Server process starts (PID visible) but never binds to port 3002, health endpoint returns connection refused
- **Diagnosis Steps:**
  - Verified PORT=3002 in `.env` file
  - Confirmed `app.listen()` code exists at server.js line 1004
  - Checked process with `ps` → running with 0.08s CPU time
  - Checked network with `lsof -p [PID] -a -i | grep LISTEN` → no ports bound
  - Examined service file headers (enrichmentService_sqlite.js, searchService.js, chartService.js, aiAssistant.js)
- **Root Cause:** One or more service modules attempting external API connections during require() phase, blocking before app.listen() is reached
- **Likely Culprits:** searchService (Spotify token), chartService (Soundcharts/Wikipedia), enrichmentService_sqlite (multiple APIs)
- **Solution:** Refactor services to use lazy initialization — initialize connection properties to null during require(), only connect on first method call
- **Status:** Identified but not yet fixed

**Documentation Updates:**
- Updated `.github/copilot-instructions.md` with correct port 3002
- Added REFERENCE FILES section to MASTER_ARCHITECTURE.md (comprehensive task → file mapping table)
- Updated CURRENT STATUS section with teaching data import progress and server hang details
- Added DECISIONS LOG entry: "Do not attempt to connect to external APIs during service module initialization"

**Files Created:**
- `materials-server/importTeachingData.js` (203 lines) — CSV import with fuzzy matching

**Files Archived:**
- 9 outdated MD files moved to `docs/archive/` (API setup guides, old enrichment docs)

**Next Priority:**
1. **Critical:** Fix server startup hang by implementing lazy initialization in services
2. **High:** Add `time_signature` column to database schema
3. **High:** Re-run `importTeachingData.js` to complete import for 18 songs
4. **Medium:** Investigate 2 songs not found during import
5. **Medium:** Delete ghost database file at `materials-server/sou_songs.db`
6. **Low:** Begin deployment to Railway + Vercel + Cloudflare R2

---

### 6 June 2026 — Server Startup Hang Fixed (Session 3)

**Attendees:** Matthew (user), Claude (AI assistant)  
**Duration:** ~1.5 hours  
**Context:** Continuing from Session 2 — fixed the server startup hang that prevented the app from running

**Root Causes Found (4 separate issues):**

1. **`auth.js` — `bcrypt.hashSync()` at module load (1.7s blocking)**
   - `bcrypt.hashSync(ADMIN_PASSWORD, 10)` ran synchronously on `require('./auth')`, blocking the event loop for ~1700ms
   - Fix: Replaced with a lazy async `getPasswordHash()` function — hash computed on first login attempt only
   - Result: `auth.js` now loads in 5ms

2. **`cheerio` v1.1.2 — `undici` import hang (infinite)**
   - `wikipediaChartScraper.js` used `require('cheerio')` (full bundle)
   - Cheerio v1.1.2 full bundle imports `undici` v7.16.0 which hangs indefinitely on Node.js v22
   - Fix: Changed to `require('cheerio/slim')` (HTML parsing only, no network fetch features)
   - Result: `wikipediaChartScraper.js` now loads in 29ms

3. **`uuid` v13.0.0 — ESM-only, hangs in CJS context (infinite)**
   - `searchSessionStore.js` used `require('uuid')` with uuid v13 installed
   - uuid v13 is ESM-only with no CJS build — attempting to require it hangs Node.js v22
   - Fix: Downgraded to `uuid@9` (`npm install uuid@9 --save`), which has full CJS support
   - Result: `searchSessionStore.js` now loads instantly

4. **Apparent "hang" after all fixes — actually buffered output**
   - After fixing the 3 real hangs, server appeared to still hang (0 output, port not bound)
   - Root cause: `console.log` output buffered — server WAS running on port 3002 within seconds
   - Confirmed with `lsof -i :3002 | grep LISTEN` → `3002 BOUND`
   - `/health` responded: `{"status":"ok","timestamp":"..."}`
   - `/api/songs` returned 215 songs correctly

**Additional Work (teaching data + cleanup):**
- Added `time_signature` column: `ALTER TABLE songs ADD COLUMN time_signature TEXT;`
- Re-ran `importTeachingData.js`: 210/212 songs updated (up from 192/215)
- 2 remaining not-found songs are known bad CSV rows (malformed or mismatched)
- Deleted ghost 0-byte database at `materials-server/sou_songs.db`

**Verified Working:**
- `curl http://localhost:3002/health` → `{"status":"ok",...}`
- `curl http://localhost:3002/api/songs` → 215 songs returned
- Module load time: 265ms for all server modules (was: infinite hang)

**Current Status Update:**
- ✅ Server startup hang: FIXED
- ✅ Teaching data import: 210/212 songs (was 192/215)
- ✅ Ghost database file: DELETED
- ❌ 2 songs still not imported (known bad CSV rows — minor)
- ❌ Deployment to Railway + Vercel: still pending

**Next Priority:**
1. Deploy backend to Railway
2. Upload PDFs to Cloudflare R2
3. Deploy frontend to Vercel

---

### 6 June 2026 — Full Deployment to Railway + Vercel (Session 4)

**Attendees:** Matthew (user), Claude (AI assistant)  
**Duration:** ~3 hours  
**Context:** Continuation of Session 3 — server startup fixed, app running locally; goal was deploying to production

**Issue 1: Node 22 Incompatibility with CRA Build**
- **Problem:** `npm run build` in sou-song-browser hung indefinitely on Node.js v22
- **Root Cause 1:** `fork-ts-checker-webpack-plugin` uses `cosmiconfig` → ESM deadlock on Node 22
- **Root Cause 2:** `workbox-webpack-plugin` imports `common-tags` → hangs on Node 22
- **Solution:** Installed Node 18.20.8 via nvm (`nvm install 18 && nvm use 18`), fresh `npm install`, `npm run build` succeeded
- **Impact:** All Node.js development must use Node 18 (see DECISIONS LOG)

**Issue 2: better-sqlite3 ABI Mismatch After Node Switch**
- **Problem:** Native module compiled for Node 22 (ABI 127) failed on Node 18 (ABI 108)
- **Solution:** `npm rebuild better-sqlite3` in `materials-server/` — recompiled for Node 18 in ~30s

**Issue 3: Remaining Teaching Data (2 songs)**
- Shirelles: Found by ID `the_shirelles_will_you_love_me_tomorrow`, updated directly in SQLite: `level='1'`, `sou_keys='C'`
- Dawn Penn: Already had `level='2'`, `sou_keys='Am'` — was already complete
- **Result: Teaching data 212/212 (100% complete)**

**Deployment: Backend to Railway**
- Created `materials-server/.gitignore` (excludes `node_modules/`, `.env`, `logs/`, `*.node`)
- Initialized git repo in `materials-server/`, pushed to `https://github.com/dukeofuke-png/sou-backend.git`
- Commit `b3a80bd`: 171 files, 7.70 MB — includes `sou_songs.db` in repo for initial deploy
- Connected Railway to `sou-backend` GitHub repo → auto-deployed
- **Railway URL:** `https://sou-song-browser-production.up.railway.app`
- Verified: `/songs` and `/api/songs` both return 215 songs

**Deployment: Frontend to Vercel**
- Initialized git repo in `sou-song-browser/`, pushed to `https://github.com/dukeofuke-png/sou-song-browser.git`
- Committed all components: AdminDashboard, ManageSOUDatabase, SeedDatabaseSearch, SongDetailModal, etc.
- Vercel build command: `CI=false npm run build` (required — `CI=true` treats ESLint warnings as errors, blocking build)
- Set Vercel env vars: `REACT_APP_API_URL` and `REACT_APP_MATERIALS_URL` → Railway URL
- Triggered redeploy after env vars set (CRA bakes env vars into bundle at build time)
- **Vercel URL:** `https://sou-song-browser.vercel.app`
- Verified: Bundle contains `railway.app` not `localhost:3002`

**Security Issue: GitHub Token Exposed**
- ⚠️ A GitHub personal access token was accidentally shared in chat during this session
- **Action Required:** Revoke at https://github.com/settings/tokens immediately

**Files Created/Modified:**
- `materials-server/.gitignore` (created)
- `sou-song-browser/.gitignore` (updated to exclude `.env`, `.env.local`)
- `materials-server/auth.js` (modified — lazy bcrypt hash, already done in Session 3)
- `materials-server/wikipediaChartScraper.js` (modified — `cheerio/slim`, already done in Session 3)

**Outstanding Issues:**
1. **Railway CORS** — `FRONTEND_URL=https://sou-song-browser.vercel.app` must be added to Railway env vars; `server.js` uses this for production CORS origin (line 34-35)
2. **SQLite persistence** — DB is in repo (works now); enrichment writes will be lost on redeploy; needs Railway Volume at `/app/data/`
3. **PDF storage** — PDFs still served from local Google Drive; needs migration to Cloudflare R2
4. **Revoke exposed GitHub token** — see security note above

**Next Priority:**
1. **Immediate:** Set `FRONTEND_URL=https://sou-song-browser.vercel.app` in Railway environment variables
2. **Immediate:** Revoke exposed GitHub token at https://github.com/settings/tokens
3. **High:** Set up Railway Volume for SQLite persistence
4. **Medium:** Migrate PDFs to Cloudflare R2

---

### 7 June 2026 — Genre, Era & Tags Filter Fixes (Session 5)

**Attendees:** Matthew (user), Claude (AI assistant)  
**Duration:** ~1.5 hours  
**Context:** App live on Vercel/Railway; three frontend filter bugs reported, plus follow-up data architecture investigation

**Bug 1: Era dropdown showing nothing**
- **Root cause:** `release_era` auto-converts to camelCase `releaseEra`; frontend reads `song.era`. No explicit mapping existed (unlike `release_season → season`).
- **Fix:** Added `'release_era': 'era'` to `fieldMappings` in `toFrontendFormat()` — one line. Commit `32a8d7d`.

**Bug 2: Genre filter showing artist names and noise tags**
- **Root cause:** `genres_best` is a deduped merge of ALL sources including Last.fm, which includes artist names ("50 Cent"), location tags ("New York"), playlist tags ("Guilty Pleasure"), year tags ("70s"). 32 of 215 songs had no clean genre data and fell through to `genres_best`.
- **Fix 1 (waterfall, commit `32a8d7d`):** Replaced single-source value with waterfall: `genres_wikipedia → genre (CSV) → genres_getsongbpm → genres_best`
- **Fix 2 (drop genres_best, commit `d0b0701`):** Removed `genres_best` from waterfall entirely after confirming 32 fallback songs were all noisy. 183/215 songs (85%) covered by clean sources. 32 songs with no clean genre return null and are omitted from dropdown.
- **Data architecture confirmed:** `genres_best` stays in DB unchanged, useful for future full-text search. Not used in any API-facing field.

**Bug 3: Materials links dead (assessed, not fixed)**
- Confirmed all 197 PDF paths are absolute local macOS paths; no fix possible until Cloudflare R2 migration.

**Follow-up: Tags field architecture investigation**
- **Finding:** The DB has a raw `tags` column (col 234, added post-schema) containing human-curated teaching tags from the CSV `Tags` column: mood/context descriptors like `"Classic Pop"`, `"Christmas, Rock"`, `"Ballroom, Slow Tempo, Latin Standard"`. 137/212 songs populated.
- **Previous bug:** `toFrontendFormat()` was mapping `tags_lastfm_track → tags`, overwriting the curated CSV tags entirely with Last.fm crowdsourced data.
- **Fix (commit `c7b7e5f`):** Added `tags` waterfall: `tags (CSV) → tags_lastfm_track → tags_lastfm_artist`. Curated tags now take priority; Last.fm augments only when CSV is empty.

**Final field semantics (API output):**
- `genre` — clean: `genres_wikipedia || genre (CSV) || genres_getsongbpm || null`
- `tags` — curated: `tags (CSV) || tags_lastfm_track || tags_lastfm_artist || null`
- `genres_best` — stays in DB, not sent to frontend (available for future search)

**Commits:** `32a8d7d`, `d0b0701`, `c7b7e5f` on `dukeofuke-png/sou-backend`

**Next Priority:**
1. Verify fixes live on Railway after deploys complete
2. Set `FRONTEND_URL=https://sou-song-browser.vercel.app` in Railway env vars (CORS — outstanding from Session 4)
3. Migrate PDFs to Cloudflare R2

---

### 7 June 2026 — Frontend Polish, CSV Corruption Fix & Favicon (Session 5 cont.)

**Attendees:** Matthew (user), Claude (AI assistant)  
**Duration:** ~2 hours  
**Context:** Continuation of same-day session — additional data-quality fixes and cosmetic improvements

**Bug 4: Genre dropdown capitalisation duplicates (`blues`/`Blues`, `folk`/`Folk`)**
- **Root cause:** Genre values stored with inconsistent casing in DB; Set insertion was case-sensitive.
- **Fix:** Added `toTitleCase()` normalisation in `genres` useMemo before Set insertion. Commit `2e699f0` on `sou-song-browser`.

**Bug 5: Decade tokens in genre dropdown (`80s`)**
- **Root cause:** Bananarama "Venus" had `genre = "Pop, Rock, 80s"` in the CSV — `80s` is a tag, not a genre.
- **Fix 1 (DB, commit `ef386de`):** Cleared `80s` from `genre` column for that row; value retained in `tags` column.
- **Fix 2 (frontend, commit `c0b492d`):** Added `isDecade()` regex safety net (`/^\d+0s$/i`) in genres useMemo to filter any future decade tokens.

**CSV Column-Shift Corruption (78 songs)**
- **Root cause:** `importTeachingData.js` (Session 2) used naive `line.split(',')` which broke on quoted fields (e.g. `Tags = "R&B, Pop"` shifted all subsequent columns right). 66 songs had non-numeric `level`, 30 had non-numeric `num_chords`.
- **Fix:** Created `materials-server/fixTeachingDataImport.py` (294 lines) — uses Python `csv.DictReader` (handles quoted fields correctly), matches songs by ID/title+artist/title-only, patches only fields that differ. Commit `1606d8b`.
- **Result:** 78 records patched. Script retained in repo for future reference.
- **Fields fixed:** `level`, `sou_keys`, `num_chords`, `chords`, `chord_numerals`, `time_signature`, `strum_style`, `fingerpicking_style`, `teaching_notes`, `tags`, `song_sheet_status`, `tab_status`

**Stray `"` chars in `teaching_notes` (107 songs)**
- **Root cause:** Same CSV import bug — quoted fields left residual `"` characters.
- **Fix:** `UPDATE songs SET teaching_notes = TRIM(TRIM(teaching_notes, '"'), '"')`. Commit `e694d2d`.

**Favicon & PWA icons replaced**
- **Source:** `sou-song-browser/src/assets/sou-logo.png` (4601×4059, palette mode PNG, transparent background)
- **Generated with Python Pillow:**
  - `favicon.ico` — multi-size ICO: 16×16, 32×32, 48×48 (4.2 KB). Replaced CRA default.
  - `logo192.png` — 192×192 RGBA PNG (15 KB). Replaced CRA default.
  - `logo512.png` — 512×512 RGBA PNG (53 KB). Replaced CRA default.
  - Logo fitted into square canvas with transparent padding (aspect ratio ~1.13:1).
- **Also updated:** `public/manifest.json` — `short_name: "SOU Songs"`, `name: "School of Uke Song Browser"`
- **Commit:** `d15cf28` on `sou-song-browser` → auto-deployed to Vercel

**All commits this session (sou-backend):** `32a8d7d`, `d0b0701`, `c7b7e5f`, `ef386de`, `1606d8b`, `e694d2d`  
**All commits this session (sou-song-browser):** `2e699f0`, `c0b492d`, `d15cf28`

**Outstanding (manual actions required):**
1. Set `FRONTEND_URL=https://sou-song-browser.vercel.app` in Railway dashboard env vars (production CORS)
2. Set up Railway Volume at `/app/data/` for SQLite persistence
3. Migrate PDFs to Cloudflare R2

---

### 7 June 2026 — Favicon, Railway DB Fix, R2 Migration, Mode/Key Fixes & Admin Audit (Session 6)

**Attendees:** Matthew (user), Claude (AI assistant)  
**Duration:** ~4 hours  
**Context:** Continuation of same-day session — cosmetic fixes, two production emergencies, major infrastructure migration, data-quality fixes, and full admin panel audit with 4 bug fixes

**Favicon quality upgrade**
- **Problem:** Previous favicon used `sou-logo.png` (4601×4059 palette PNG, different visual from website)
- **Fix:** Fetched official `SCHOOL_logo_2000px.png` (2000×1629 RGBA, 208 KB) directly from Cloudflare CDN used by schoolofuke.co.uk. Regenerated all three icon sizes with Pillow.
- **Pitfall discovered:** Pillow's `append_images` parameter does not work for ICO format; must use `sizes=[(16,16),(32,32),(48,48)]` on a 48px source image.
- **Commit:** `91ea142` on sou-song-browser

**Railway DB malformed (production emergency)**
- **Root cause:** `sou_songs.db-wal` was committed alongside the main DB. Railway replayed a stale WAL against the wrong DB version → `{"error":"database disk image is malformed"}` on all API calls.
- **Fix:** `PRAGMA wal_checkpoint(TRUNCATE)` + `PRAGMA journal_mode=DELETE` to flush and permanently disable WAL; removed WAL/SHM files from git; added `data/*.db-wal` and `data/*.db-shm` to `.gitignore`.
- **Commits:** `b422037`, `8029f63` on sou-backend

**Cloudflare R2 PDF migration (190 PDFs)**
- **Goal:** Replace all local Google Drive PDF paths with public R2 URLs
- **Script:** `materials-server/uploadToR2.py` — iterates DB rows with local paths, uploads via `wrangler r2 object put`, updates `song_sheet_path` in DB after each success. Object key pattern: `{song_id}.pdf`.
- **Result:** 190/191 songs uploaded. 1 skipped: Espresso (no local PDF file exists).
- **One transient failure:** "Never Let Me Down Again" (Depeche Mode) — `fetch failed` at position 112/190. Retried manually with wrangler, then patched DB with Python.
- **R2 bucket:** `sou-song-sheets`, public base URL `https://pub-e43364bf5aa34598832e4b2e860e074d.r2.dev`
- **Frontend:** `SongDetailModal.js` updated — if `songSheetPath.startsWith('https://')`, show direct R2 link; otherwise fall through to legacy `materials_json` logic.
- **Backend:** `/materials/*` endpoint replaced with `410 Gone` response.
- **Commits:** `378f9ef` backend, `8b1913b` frontend

**Mode/Key display fix (3 songs with corrupted numeric values)**
- **Root cause:** Original Spotify audio-features import stored wrong values: `mode` column contained loudness values (-14.24, -16.65); `original_key` contained Spotify numeric key (6.04) or tempo (193.46).
- **Affected songs:** In Da Club (50 Cent), Get Rich Or Die Tryin' (50 Cent), I Got The (Joanna Wang)
- **Fix:** Added validation waterfalls in `toFrontendFormat()`:
  - `mode`: validates DB value is `major`/`minor`; falls back `mode_getsongbpm → derived from key_best → null`; title-cases output
  - `originalKey`: validates value is a musical key (not numeric); falls back `key_getsongbpm → original_key (if valid) → key_best`
- **Commits:** `f05f8bf` backend, `20ff6c7` frontend

**Admin panel comprehensive audit**
- Audited all 12 admin components (AdminDashboard, AdminLayout, AdminLogin, ManageSOUDatabase, SongEditor, BulkImport, BulkExport, BulkAddSongs, SeedDatabaseSearch, ChartsView, PopularityCatalog, AdminAIHelper) plus all backend admin routes
- **Auth:** Production bypass confirmed active — `bypassAuth` middleware ran unconditionally including in production
- **Delete bug:** `handleDeleteSong()` was passing the array index (`index`) to `DELETE /api/songs/:id` instead of `song.id` — would delete the wrong song
- **Seed promote:** `POST /api/seed/promote` called `csvManager.createSong()` (writes to CSV file), not `dbManager.addSong()` (writes to SQLite) — promoted songs would not appear in the running app
- **Dashboard PDF count:** Still checking `songSheetStatus === 'Yes'` — should check `songSheetPath.startsWith('https://')` after R2 migration
- **Working features confirmed:** song CRUD (GET/PUT/POST/DELETE), bulk update, CSV import (preview + commit), bulk export, seed search, column picker, inline edit, saved views (localStorage), enrichment endpoint

**4 Admin Fixes Applied & Deployed**

| Fix | File | Commit |
|-----|------|--------|
| Auth bypass gated to `NODE_ENV !== 'production'` | `server.js` | `dc4709c` sou-backend |
| Seed promote writes to SQLite via `dbManager` | `routes/seed.js` | `e1c2c62` sou-backend |
| Delete uses `song.id` not array index | `AdminDashboard.js` | `78b31df` sou-song-browser |
| Dashboard PDF count checks R2 URLs | `AdminDashboard.js` | `78b31df` sou-song-browser |

**All commits this session (sou-backend):** `8029f63`, `b422037`, `378f9ef`, `f05f8bf`, `dc4709c`, `e1c2c62`  
**All commits this session (sou-song-browser):** `2e699f0` (prev), `c0b492d` (prev), `d15cf28` (prev), `91ea142`, `8b1913b`, `20ff6c7`, `78b31df`

**Outstanding (manual actions required):**
1. Set `FRONTEND_URL=https://sou-song-browser.vercel.app` in Railway dashboard (production CORS)
2. Set up Railway Volume at `/app/data/` for SQLite persistence across redeploys
3. Upload Espresso PDF to R2 manually when file is located

---

### 8 June 2026 — Admin Panel Audit & Security Fixes (Session 7)

**Attendees:** Matthew (user), Claude (AI assistant)  
**Duration:** ~2 hours  
**Context:** New session — admin panel had never been formally audited; user requested comprehensive review before continuing feature work

**Admin Panel Audit**

Performed full audit of all 12 admin components and all backend admin API routes:
- **AdminDashboard.js** (368 lines) — routing, stats, song list, delete/edit handlers
- **AdminLayout.js** (128 lines) — sidebar nav with Song Discovery submenu
- **AdminLogin.js** (79 lines) — password form → `POST /api/auth/login`
- **ManageSOUDatabase.js** (1,488 lines) — full database table with inline/modal/bulk edit
- **SongEditor.js** (345 lines) — individual song form, delegates save to parent
- **BulkImport.js** (320 lines) — manual text entry + CSV file upload
- **BulkExport.js** (180 lines) — seed DB export
- **BulkAddSongs.js** (560 lines) — paste artist/title list → seed search → promote
- **SeedDatabaseSearch.js** (366 lines) — filter 47K seed songs → promote
- **ChartsView.js** (244 lines), **PopularityCatalog.js** (286 lines), **AdminAIHelper.js** (108 lines)

**4 Bugs Found & Fixed**

**Bug 1 (CRITICAL): Auth bypass active in production**
- `bypassAuth` middleware ran unconditionally — every request to `/api/*` was auto-authenticated in all environments including Railway production
- Fix: Gated inside `if (process.env.NODE_ENV !== 'production')` block
- Commit: `dc4709c` on sou-backend

**Bug 2 (CRITICAL): Delete passes array index, not song ID**
- `AdminDashboard.handleDeleteSong()` called with `index` (0, 1, 2…) from the `.map()` loop
- `DELETE /api/songs/4` would delete whatever DB row has `id = 4`, not the displayed song
- Fix: Changed call site to `handleDeleteSong(song.id)`
- Commit: `78b31df` on sou-song-browser

**Bug 3 (HIGH): Seed promote writes to CSV not SQLite**
- `POST /api/seed/promote` called `csvManager.createSong()` — writes to CSV file, not the SQLite database the app actually serves from
- Songs promoted via Search & Add or Bulk Add would never appear in the running app
- Fix: Replaced with `dbManager.getSongsByTitleArtist()` (duplicate check) + `dbManager.addSong()` (insert); removed `csvManager` import
- Commit: `e1c2c62` on sou-backend

**Bug 4 (MEDIUM): Dashboard PDF count stale after R2 migration**
- Dashboard "With PDFs" stat checked `songSheetStatus === 'Yes'` — a field that predates the R2 migration
- After migration, the definitive signal is `songSheetPath.startsWith('https://')`
- Fix: Updated filter expression
- Commit: `78b31df` on sou-song-browser

**Admin Panel Feature Status (post-audit)**

| Feature | Status |
|---|---|
| Song CRUD (edit, add, delete) | ✅ Working — SQLite |
| Bulk update | ✅ Working — SQLite |
| CSV import (preview + commit) | ✅ Working — SQLite |
| Bulk export | ✅ Working |
| Seed search & filter | ✅ Working |
| Promote seed → SOU DB | ✅ Working (post-fix) — SQLite |
| Column picker, saved views | ✅ Working — localStorage |
| Inline cell edit | ✅ Working — SQLite |
| Settings page | ❌ Placeholder ("coming soon") |
| Admin panel authentication | ✅ Secured in production (post-fix) |

**Decisions Logged:**
- Auth bypass must be dev-only: `NODE_ENV !== 'production'`
- Seed promote must use dbManager (SQLite), not csvManager (CSV legacy)

**Commits this session (sou-backend):** `dc4709c`, `e1c2c62`  
**Commits this session (sou-song-browser):** `78b31df`

**Outstanding (manual Railway dashboard actions):**
1. Set `FRONTEND_URL=https://sou-song-browser.vercel.app` (production CORS) — outstanding since Session 4
2. Create Railway Volume at `/app/data/` (SQLite persistence across redeploys) — outstanding since Session 4
3. Upload Espresso PDF to R2 manually when local file is located

---

### 8 June 2026 — Infrastructure, Volume Seeding & Product Vision Update (Session 8)

**Attendees:** Matthew (user), GitHub Copilot (Claude Sonnet 4.6)  
**Duration:** ~1.5 hours  
**Context:** Continuation of Session 7 — completed the pending PROJECT_FEATURE_MAP replacement, then resolved two production issues (Railway Volume empty on first boot, Vercel env var missing)

**Task 1: PROJECT_FEATURE_MAP.md replaced with new product vision**
- Old Nov 2025 feature map (Tutor UI / Admin UI / End-user UI / Genius Bar structure) archived to `docs/archive/PROJECT_FEATURE_MAP_archive_Nov2025.md`
- `PROJECT_FEATURE_MAP.md` replaced with new June 2026 document: "SCHOOL OF UKE — PRODUCT VISION"
- New document defines two distinct products: **The Tutor Platform** (creative workspace, conversation-first AI, split-screen editor, content publishing) and **The Student Platform** (Songfinder, digital materials, theory database)
- Covers: role-based sidebar (Super Admin / Tutor), songsheet creation workflow, two forms of every sheet (digital interactive + print/PDF), content ownership model, chord library architecture, and key principles

**Task 2: Railway Volume empty-on-first-boot fix**
- **Problem:** Railway Volume mounted at `/app/data/` is empty on first deploy and completely shadows the `data/` directory, hiding the committed `sou_songs.db`. Server threw `Database not found` on startup.
- **Root cause confirmed:** `DB_PATH` in `dbManager.js` was already `path.join(__dirname, 'data', 'sou_songs.db')` which resolves to `/app/data/sou_songs.db` — matches the volume mount path exactly. The path itself was correct; the volume was simply empty.
- **Fix:**
  - Copied `data/sou_songs.db` → `sou_songs_seed.db` at repo root (outside `data/`, so not shadowed by volume)
  - Updated `dbManager.getDb()`: if `DB_PATH` missing, check for `sou_songs_seed.db`; if found, copy to `DB_PATH` and log `"Database seeded from ..."`; if seed also missing, throw original error
  - Made `DB_PATH` configurable via `process.env.DB_PATH` for future flexibility
- **Commit:** `d5ba19b` on sou-backend (pushed, Railway auto-redeploy triggered)

**Task 3: Vercel REACT_APP_API_URL missing (frontend showing port 3002 error)**
- **Symptom:** Production Vercel app showed "Error loading songs: Failed to fetch songs: Make sure the server is running on port 3002"
- **Root cause:** `App.js` line 10: `const API_URL = process.env.REACT_APP_API_URL || 'http://localhost:3002'`. The env var was not set on Vercel, so it fell back to localhost.
- **Fix:** Manual action in Vercel Dashboard — user set `REACT_APP_API_URL=https://sou-song-browser-production.up.railway.app` and triggered a redeploy
- **Result:** App confirmed working by user ("all good now")
- **Note:** This env var was set during Session 4 but not at the correct Vercel project scope / was lost. Now confirmed present.

**Decisions Logged:**
- Railway Volume seeding strategy: bundle `sou_songs_seed.db` at repo root; `dbManager.getDb()` auto-copies to volume path on first boot

**Commits this session (sou-backend):** `d5ba19b`  
**Commits this session (sou-song-browser):** none (Vercel env var change, no code change)

**Current production state (end of session):**
- ✅ Frontend: `https://sou-song-browser.vercel.app` — loading songs correctly
- ✅ Backend: `https://sou-song-browser-production.up.railway.app` — 215 songs, seeded from `sou_songs_seed.db` on volume first-boot
- ✅ Auth: production-secured, admin panel accessible with password
- ✅ PDFs: 190/191 on Cloudflare R2
- ✅ Product vision document: updated and archived

**Outstanding:**
1. Set `FRONTEND_URL=https://sou-song-browser.vercel.app` in Railway env vars (CORS) — still manual action required
2. Upload Espresso PDF to R2 when local file located
3. Refresh `sou_songs_seed.db` whenever significant DB changes are made locally

---

### 12 June 2026 — CORS Verification (Session 9)

**Attendees:** Matthew (user), GitHub Copilot (Claude Sonnet 4.6)  
**Duration:** ~15 minutes  
**Context:** New session — verifying the outstanding CORS item that had been flagged since Session 4

**Task: CORS configuration audit**

Matthew confirmed `FRONTEND_URL=https://sou-song-browser.vercel.app` is already set in the Railway dashboard. Before marking the CORS issue fully resolved, performed a code audit of `server.js` CORS setup.

**Findings:**

1. **Single CORS call confirmed** — only one `app.use(cors(corsOptions))`, no overrides or duplicate headers elsewhere in server.js
2. **Two Railway vars required, not one:**
   - `FRONTEND_URL` — confirmed set ✅
   - `NODE_ENV=production` — **not yet verified** ⚠️. Railway does NOT auto-set this (unlike Heroku/Vercel). Without it:
     - `corsOptions.origin` falls to `'http://localhost:3000'` (Vercel frontend gets CORS errors — fails closed)
     - More critically: `bypassAuth` middleware condition (`NODE_ENV !== 'production'`) stays active → admin panel unprotected in production
3. **Failure mode analysis:**
   - `NODE_ENV=production` + `FRONTEND_URL` set → ✅ Correct (origin locked to Vercel URL)
   - `NODE_ENV=production` + `FRONTEND_URL` unset → `origin = undefined` → cors package allows all origins (fails open)
   - `NODE_ENV` missing + anything → `origin = 'http://localhost:3000'` → Vercel frontend broken + auth bypass active (fails insecurely)

**No code changes made this session.**

**Resolution (confirmed same session by Matthew):**
- `NODE_ENV=production` confirmed present in Railway dashboard ✅
- Both `FRONTEND_URL` and `NODE_ENV` verified — CORS and auth are correctly locked down in production ✅
- Item fully closed.

**Outstanding:**
1. Upload Espresso PDF to R2 when local file is located

---

### 12 June 2026 — AI Conversation Feature (Session 10)

**Attendees:** Matthew (user), GitHub Copilot (Claude Sonnet 4.6)  
**Duration:** ~3 hours  
**Context:** Built the full AI conversation feature — backend schema, API, AI provider wrapper, and React frontend — from scratch in a single session.

**Task 1: Chat DB schema design & migration**

Five new tables added to `sou_songs.db` via `addChatTables.js` migration:

| Table | Purpose |
|---|---|
| `tutors` | User accounts (Matthew seeded as `id=1, role=super_admin`) |
| `tutor_profiles` | Per-tutor context notes injected into AI system prompt |
| `conversations` | Conversation threads (UUID PK, title, archived flag) |
| `messages` | Individual messages (role: `tutor` / `assistant` / `system`, content, model, token_count) |
| `messages_fts` | FTS5 virtual table mirroring messages for full-text search |

Migration is idempotent (`IF NOT EXISTS` + `INSERT OR IGNORE`). Safe to re-run.

**Task 2: AI provider wrapper — `services/aiProvider.js`**
- Provider-agnostic interface: `callAI(messages, systemPrompt)` → `{text, provider, model, usage}`
- Supports Gemini (`@google/generative-ai` v0.24.1) and Anthropic (`@anthropic-ai/sdk` v0.27.3)
- Provider selected via `AI_PROVIDER` env var (default: `gemini`); model via `AI_MODEL` (default: `gemini-2.5-flash`)
- Role mapping: `tutor → user`, `assistant → model` (Gemini) / `assistant` (Anthropic)
- Lazy `require()` inside each provider function to avoid module-load hangs
- Added `@google/generative-ai` to `package.json`; `GEMINI_API_KEY` added to `.env` locally

**Task 3: Chat API — `routes/chat.js`**

Five endpoints mounted at `/api/chat` behind `requireAuth`:

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/conversations/:id/messages` | Send message (pass `new` as `:id` for new conversation) |
| `GET` | `/conversations` | List non-archived conversations (tutor_id=1), newest first |
| `GET` | `/conversations/:id/messages` | Full message history for a conversation |
| `GET` | `/profile` | Tutor context notes |
| `PATCH` | `/profile` | Update tutor context notes |

Key design choices:
- `TUTOR_ID = 1` hardcoded at top of file — single place to update for multi-tutor migration
- `BASE_SYSTEM_PROMPT` is a placeholder — AI personality / brand-tuning deferred to a future session
- FTS index synced on every message store (both tutor and assistant messages)
- New conversation title auto-set to first 60 chars of opening message
- Full conversation history is fetched and passed to AI on every turn (no windowing yet)

**Live end-to-end test (same session):** POST to `/api/chat/conversations/new/messages` with Gemini 2.5 Flash — full AI response received, message stored in DB, conversation created. ✅

**Task 4: ConversationWorkspace frontend — 4 files**

- `ConversationWorkspace.js` — two-panel React component:
  - Left sidebar: conversation list with timestamps + "New" button
  - Right panel: message history with optimistic tutor message render, typing indicator, auto-scroll
  - Input row: textarea with Enter-to-send / Shift+Enter-for-newline
  - On first send: creates new conversation, sets `activeConvId`, reloads sidebar
  - On resume: `GET /messages` fetches full history, renders in order
- `ConversationWorkspace.css` — SOU orange `#FF6B35` accent; tutor bubbles right/orange, assistant bubbles left/white/bordered; typing bounce animation; responsive two-column layout
- `AdminDashboard.js` — added `import ConversationWorkspace` and `case 'conversation'` in `renderPage()`
- `AdminLayout.js` — added "Conversation" nav item with `FiMessageSquare` icon (between Dashboard and Song Discovery)

**Task 5: PROJECT_FEATURE_MAP.md updated**
- Added "Known Follow-Up: Conversation as Default Landing Page" — deferred until role-based sidebar is built alongside multi-tutor auth migration

**Commits this session:**
- `d2aeb3f` on `sou-backend` — all backend chat feature files
- `b30aa93` on `sou-song-browser` — all frontend chat UI files

**Railway env vars required (manual action — not yet done):**
- `GEMINI_API_KEY` — Gemini API key (check local `.env` for value)
- `AI_PROVIDER=gemini`
- `AI_MODEL=gemini-2.5-flash`

**Current production state (end of session):**
- ✅ Backend: conversation feature code deployed to Railway (pending env vars above — chat will fail in production until set)
- ✅ Frontend: ConversationWorkspace available as nav item on Vercel
- ✅ `sou_songs_seed.db` refreshed — chat tables will be present on next Railway volume re-seed
- ⚠️ System prompt is placeholder ("You are a helpful music teaching assistant for SOU...") — AI personality tuning deferred
- ⚠️ Chat routes will return 500 on Railway until `GEMINI_API_KEY` is added to Railway dashboard

**Outstanding:**
1. **Immediate:** Add `GEMINI_API_KEY`, `AI_PROVIDER=gemini`, `AI_MODEL=gemini-2.5-flash` to Railway dashboard env vars
2. Upload Espresso PDF to R2 when local file is located
3. AI system prompt tuning — craft SOU brand voice, teaching context, example songs reference (future session)
4. Multi-tutor auth migration (trigger: when second tutor account is needed)

---

### 12 June 2026 — searchSongs tool (Session 10 cont.)

**Attendees:** Matthew (user), GitHub Copilot (Claude Sonnet 4.6)

**Task: Metadata audit + searchSongs AI tool**

**Metadata population audit (215 songs):**

| Field | Coverage | Usable? |
|---|---|---|
| `original_key` / `sou_keys` | 98–97% | ✅ Primary filter |
| `level` | 68% | ✅ Range-aware matching needed |
| `genre` / `tags` | 63–66% | ✅ Partial LIKE match |
| `bpm_best` | 62% | ✅ Range queries |
| `chords` | ~15% (corrupt) | ❌ Deferred |
| `time_signature`, `key_best` | <10% / 1% | ❌ Too sparse |

Chord column analysis: of 32 non-null entries, ~18 contain tag/mood text (`Melancholic`, `Key change`, `1940s`) — CSV column-shift corruption not caught by previous fix because the source CSV itself had the shift. Chord filtering deferred to a future vision/audio extraction phase.

**services/searchSongs.js (new file)**
- Filters: `key` (matches `original_key` OR any token in `sou_keys`), `level` (range-aware: `level=2` matches `"2"`, `"1, 2"`, `"2, 3"`, `"1,2"`), `genre` (partial LIKE), `tags` (partial LIKE), `bpmMin`/`bpmMax`, `limit` (default 10, max 30)
- Returns lean 9-field subset: `id, title, artist, key, souKeys, level, genre, tags, bpm`
- Named params (`:key`, `:level`, etc.) — safe against SQL injection at the SQLite layer

**services/aiProvider.js (updated)**
- `SONG_SEARCH_TOOL` declaration added — Gemini function-calling format
- `callGemini()` rewritten to handle multi-turn tool-call flow: detect `functionCalls()`, run `executeTool()`, inject `functionResponse` turn, call `generateContent` again, sum token counts across both calls
- `executeTool()` dispatcher added — extensible for future tools
- Tool description tuned after first-attempt failure: Gemini tried `tags="Upbeat"` which returned 0 results (tag vocabulary doesn't contain "Upbeat"). Fixed by: listing known tag values in description, directing Gemini to use `bpmMin` for upbeat/energetic queries, adding retry-with-fewer-constraints guidance

**Test results:**
- `key=C, level=2` → 10 results, range-aware level matching verified across `"2"`, `"2, 3"`, `"1, 2"`, `"1,2"`, `"1, 2, 3"` variants
- `"upbeat pop under 120 BPM, level 3"` → Gemini correctly chose `genre=Pop, bpmMax=120, level=3` (no tags); returned 7 DB-grounded results: Borderline (117), Murder On The Dancefloor (117), Rock With You (112), Let's Dance (116), etc.

**Commit:** `d3b942b` on `sou-backend` — pushed, Railway redeploy triggered

**Outstanding:**
1. **Immediate:** Add `GEMINI_API_KEY`, `AI_PROVIDER=gemini`, `AI_MODEL=gemini-2.5-flash` to Railway dashboard env vars (chat + tool will fail in production until set)
2. Upload Espresso PDF to R2 when local file is located
3. AI system prompt tuning — SOU brand voice, teaching context (future session)
4. Multi-tutor auth migration (trigger: when second tutor account is needed)

---

### 12 June 2026 — Tool-forgetting fix + key rotation (Session 11)

**Attendees:** Matthew (user), GitHub Copilot (Claude Sonnet 4.6)

**Bug found: Gemini stops calling `searchSongs` in long conversations**

After several general-knowledge turns, Gemini answered database questions from training data instead of calling `searchSongs`. Root cause: (a) tool-call turns are not persisted to the messages table, so the model has no evidence in history that it has previously used a tool; (b) conversational conditioning — repeated general-knowledge answer pattern leads Gemini to infer this is a general assistant, not a DB-connected one.

**Fix: stronger system prompt + message-count-scaled reminder**

- `BASE_SYSTEM_PROMPT` in `routes/chat.js` rewritten with explicit `DATABASE ACCESS RULE` section listing every query pattern requiring `searchSongs` (key, level, genre, tags, BPM, "what do we have", recommendations). Instruction: "Never answer these from general knowledge alone — this applies regardless of how many general-knowledge questions have been asked earlier in the conversation."
- `buildSystemPrompt(contextNotes, messageCount)` signature updated — when `history.length >= 6` (3+ full turns), a REMINDER paragraph re-injecting the tool-use rule is appended.
- Call site updated: `buildSystemPrompt(profile?.context_notes, history.length)`
- **Commit:** `6276d0a` on `sou-backend`

**Observability: tool-call logging added**

Added `console.log` in `services/aiProvider.js` at the tool-call detection point — Railway logs now show `[aiProvider] tool call: searchSongs {...args}` and `[aiProvider] tool result: N songs` on every invocation.
- **Commit:** `569ea62` on `sou-backend`

**Gemini API key rotation + billing upgrade**

- Old key (`AQ.Ab8RN6JCOyRU...`) disabled — had hit free-tier 20 req/day quota during testing; key also appeared in session conversation history (security)
- New key generated on same project (`AQ.Ab8RN6JBmOFg...`)
- Google Cloud billing enabled on the project → paid tier active, 20 req/day cap removed
- The `AQ.` prefix = new Google AI Studio "Authorization key" format (bound to service account). Standard `AIza` keys being deprecated (all standard keys rejected Sep 2026). The format is correct and valid — quota issue was purely free-tier, not a credential-type problem.
- Railway dashboard updated with new key; local `materials-server/.env` updated to match

**Verification: automated multi-turn test (`testMultiTurnToolUse.js`)**

New test script at `materials-server/testMultiTurnToolUse.js` — runs two scenarios against the live server:

| Scenario | Pattern | Final question | Result |
|---|---|---|---|
| A | 3 general-knowledge turns (music theory) | "What songs we have in the key of Bb?" | ✅ PASS — 5 DB songs returned |
| B | 2 teaching-help turns (screenshot repro) | "What songs in C major for absolute beginners?" | ✅ PASS — 10 DB songs with levels returned |

Both scenarios confirmed `searchSongs` was called correctly (AI returned specific SOU song titles with levels/keys, not generic suggestions). Tool-forgetting bug resolved.

**Outstanding:**
1. Upload Espresso PDF to R2 when local file is located
2. AI system prompt tuning — SOU brand voice, teaching context (future session)
3. Multi-tutor auth migration (trigger: when second tutor account is needed)

---

### 12 June 2026 — Production deploy chain reaction (Session 12)

**Attendees:** Matthew (user), GitHub Copilot (Claude Sonnet 4.6)

**Context:** A sequence of production bugs was discovered and fixed in the same session, all triggered by the first-ever real browser session on `sou-song-browser.vercel.app` after the `ConversationWorkspace` Vercel redeploy (`b30aa93`) cleared previously-cached browser auth state. Each fix revealed the next issue.

---

**Bug 1: "Manage SOU Database" shows 0/0 songs — SameSite cookie fix**

- **Symptom:** `ManageSOUDatabase` showed "Failed to load songs database". Backend returned 200 to server-side curl but 401 to browser fetches from Vercel.
- **Root cause:** Session cookie was set with `SameSite=Lax`, which browsers silently drop for cross-site requests (Vercel → Railway). The `!response.ok` branch fired → "Failed to load songs database".
- **Fix:** `server.js` session config updated to `secure: process.env.NODE_ENV === 'production'`, `sameSite: process.env.NODE_ENV === 'production' ? 'none' : 'lax'`. Development keeps `lax`; production uses `none` for cross-origin compatibility.
- **Commit:** `0c81215`
- **Pre-existing bug** (since initial commit) — not introduced by today's chat work.

---

**Bug 2: Session cookie not issued at all — trust proxy fix**

- **Symptom:** After `0c81215`, the Set-Cookie header was still absent in some cases; `api/seed/size` and `api/seed/search` returned 401 even immediately after login.
- **Root cause:** Railway terminates TLS at its load balancer. Without `app.set('trust proxy', 1)`, Express sees `req.secure=false` (the app receives plain HTTP from the proxy), so `express-session` refuses to set a `Secure` cookie — the `Set-Cookie` header is silently omitted.
- **Fix:** Added `app.set('trust proxy', 1)` before the CORS/session middleware in `server.js`. Express now trusts `X-Forwarded-Proto` from Railway's proxy, making `req.secure=true` for HTTPS requests.
- **Commit:** `0eb07b2`
- **Verified:** Login now issues `sessionId` cookie with `Secure; SameSite=None`. `/api/songs` returns 200 and session persists across multiple subsequent requests.

---

**Bug 3: "Search & Add Songs" returns "Search failed" — missing seed catalog**

- **Symptom:** `POST /api/seed/search` returned `500 {"error":"Seed database not available"}` even after auth was fixed.
- **Root cause:** `seedService.js` reads `data/expanded_seed_base.json` (47K songs, 18MB). This file was committed to `data/` in the initial commit (`b3a80bd`) but the Railway Volume mounted at `/app/data/` shadows all repo files in that directory. The volume predated the seed file and never had it.
- **Fix:** Applied the same bootstrap pattern already used for `sou_songs_seed.db`:
  - `expanded_seed_base_seed.json` — bundled copy committed **outside** `data/` (not shadowed by volume)
  - `seedService.js` — on module load, if `data/expanded_seed_base.json` is absent but the source exists, copy it in. Idempotent on subsequent boots.
- **Commit:** `461cdf2`
- **Note added to `MASTER_ARCHITECTURE.md` Section 12.2:** Railway Volume shadow pattern documented; R2 migration noted as a future option if the catalog needs frequent updates.

---

**Also this session: Edit Song fix (pre-existing bug)**

- `PUT /api/songs/:id` passed the full 232-field camelCase song object to `dbManager.updateSong()`, which used every key as a literal SQL column name. SQLite threw `table has no column named songSheetUrl` → 500 → "Failed to save song".
- **Fix:** `getValidSongColumns()` (PRAGMA-based allowlist, cached), `FRONTEND_TO_DB` reverse map (`songwriters` → `songwriters_best`), array → comma-string serialization for `tags`.
- **Commit:** `24f6bfd`
- Same fix applied to `bulkUpdateSongs`. All three augment fields (genre, tags, songwriter) verified with persistence checks.

---

**Verification results (production, post all fixes):**

| Check | Result |
|---|---|
| Login issues `Secure; SameSite=None` cookie | ✅ |
| `/api/songs` → 200 (215 songs) | ✅ |
| Session persists across 3+ requests | ✅ |
| `/api/seed/size` → 200 (47,272 songs) | ✅ |
| `POST /api/seed/search` artist=madonna → 5 results | ✅ |
| Edit Song (genre/tags/songwriter) saves correctly | ✅ (verified earlier) |

---

**Architectural decision documented:**

Studio concept (persistent chat panel, VS Code + Copilot reference model) added to `PROJECT_FEATURE_MAP.md`. Four open questions to resolve next session before any implementation. Split-screen editor work blocked on Studio layout decision.

---

**Outstanding:**
1. Upload Espresso PDF to R2 when local file is located
2. AI system prompt tuning — SOU brand voice, teaching context (future session)
3. Multi-tutor auth migration (trigger: when second tutor account is needed)
4. Studio layout planning — resolve 4 open questions, then implement persistent panel in `AdminLayout`

---

### 12 June 2026 — Full session log (Sessions 12 + 13, combined closing entry)

**Attendees:** Matthew (user), GitHub Copilot (Claude Sonnet 4.6)

**Session started:** Picking up mid-fix from compacted summary (addSong fix code-complete but uncommitted). Session ended after enrichment pipeline mapping and documentation pass.

---

#### Fix 1 — Promote Song: "table has no column named bpm" (`7f7bc7f`)

- `POST /api/seed/promote` failed for every song. `enrichSong()` returns camelCase keys (`bpm`, `originalKey`, `spotifyTrackId`, `youtubeVideoId`, `timeSignature`); `addSong()` used them verbatim as SQLite column names.
- Fix: extracted `FRONTEND_TO_DB` reverse map to module level in `dbManager.js` (shared by `addSong`, `updateSong`, `bulkUpdateSongs`). `addSong()` now normalizes → filters through `getValidSongColumns()` → INSERT. Same pattern as Edit Song fix (`24f6bfd`).
- Verified production: Blessed Madonna / "Happier" → `{"promoted":1,"failed":0}` ✅

---

#### Investigation — MemoryStore / deploy-logout (`67669bb`, docs-only)

- After `7f7bc7f` deploy, admin dashboard showed 0/0 songs again. Fresh login immediately fixed it.
- Confirmed: `express-session` uses default `MemoryStore` (no `store:` configured). Every Railway redeploy starts a new process → empty store → all browser session IDs invalid → silent 401.
- No code fix applied (single-admin scenario, tolerable). Added inline comment at `server.js` line ~50 documenting the limitation and migration path (`connect-sqlite3` or `connect-pg-simple`).
- Documented in §12.3.
- **Trigger for real fix:** when second tutor account is added.

---

#### Investigation — Promoted song not visible (user report: "Get Into the Groove")

- Not a bug. Spotify's canonical title is **"Into the Groove"** (not "Get Into the Groove"). Promote succeeded; song is in DB as `madonna_into_the_groove`. User was searching for the wrong title.
- Incidentally: all enriched fields (BPM, key, year) were null on the promoted song — prompted the enrichment pipeline investigation below.

---

#### Fix 2 — GetSongBPM URL + waterfall restructure (`5b44455`)

Two bugs in `enrichmentService.js`:

1. **Wrong endpoint:** `api.getsongbpm.com/search/` is Cloudflare-protected, returns HTML. Correct endpoint: `api.getsong.co/search/` with `type=both&lookup=song:{title} artist:{artist}&limit=1`. Also fixed: wrong response field names (`song_key` → `key_of`, added `time_sig`, `tempo` is a string not a number, mode derived from trailing `m` suffix on `key_of`).

2. **Inverted waterfall:** Spotify `audio-features` was primary for BPM/key; GetSongBPM was fallback. But `audio-features` returns 403 under Client Credentials flow (restricted since mid-2025). Restructured: GetSongBPM is now primary for BPM/key/mode/timeSignature; `audio-features` call removed entirely; Spotify is identity-only (trackId, releaseYear, genre).

Verified locally: `Holiday / Madonna → bpm:117, key:Bm, mode:Minor, timeSignature:4/4` ✅

Future refinement: add Deezer as primary BPM source (broader catalog coverage), GetSongBPM as secondary. Documented in `PROJECT_FEATURE_MAP.md` §"Known Follow-Up: Deezer as Primary BPM Source".

---

#### Finding — Two enrichment pipelines, rich one never runs in live app

Discovered that `enrichmentService.js` (lightweight, automatic on promote) and `enrichmentService_sqlite.js` (rich, Wikipedia + Last.fm + Soundcharts) are completely separate with no connection. The rich pipeline is CLI-only (`node batchEnrichAll.js`) — it never runs when a song is promoted through the UI.

| Pipeline | File | Trigger | Sources |
|---|---|---|---|
| Lightweight | `enrichmentService.js` | Auto on promote | Spotify + GetSongBPM + YouTube |
| Rich | `enrichmentService_sqlite.js` | CLI only | Wikipedia + Last.fm + GetSongBPM + Spotify + Soundcharts |

This explains why promoted songs show "Popularity data pending enrichment", null Wikipedia, null Last.fm — the data sources exist and are functional, but nothing calls them from the live app.

**Root causes for "Into the Groove" nulls specifically:**
- BPM/key null: song not in GetSongBPM DB (data coverage gap)
- YouTube null: `YOUTUBE_API_KEY` likely not set in Railway
- Last.fm/Wikipedia/charts null: expected — rich pipeline never runs automatically
- "No learning materials available yet": correct — song sheets are manual teaching artefacts, not enrichment data

**Next session priority:** Wire `enrichmentService_sqlite.js` into the live app via `POST /api/songs/:id/enrich` + admin "Enrich" button. Also confirm `YOUTUBE_API_KEY`, `LASTFM_API_KEY`, `SOUNDCHARTS_*`, `MUSICBRAINZ_USER_AGENT` are set in Railway. Test case: `madonna_into_the_groove`. Full plan in `PROJECT_FEATURE_MAP.md` §"Next Session Priority: Wire Rich Enrichment Pipeline Into Live App".

---

#### Commits this session

| Commit | Change |
|---|---|
| `7f7bc7f` | fix: addSong applies FRONTEND_TO_DB map + column allowlist before INSERT |
| `67669bb` | docs: annotate MemoryStore limitation and migration path in server.js |
| `5b44455` | fix: GetSongBPM becomes primary BPM/key source; drop Spotify audio-features |

---

**Outstanding (carried forward):**
1. Wire rich enrichment (`enrichmentService_sqlite.js`) via admin endpoint + UI button — **next session priority**
2. Confirm Railway env vars: `YOUTUBE_API_KEY`, `LASTFM_API_KEY`, `SOUNDCHARTS_*`, `MUSICBRAINZ_USER_AGENT`
3. Add Deezer as primary BPM source, GetSongBPM as secondary
4. Upload Espresso PDF to R2 when local file is located
5. AI system prompt tuning — SOU brand voice, teaching context
6. Multi-tutor auth migration (trigger: when second tutor account is needed)
7. Studio layout planning — resolve 4 open questions in `PROJECT_FEATURE_MAP.md`, then implement persistent panel in `AdminLayout`

---

### 13 June 2026 — On-demand enrichment wired to admin UI

**Attendees:** Matthew (user), GitHub Copilot (Claude Sonnet 4.6)

**Session focus:** Implement item 1 from the outstanding list above — wire the rich enrichment pipeline into the live app via a new API endpoint and admin UI controls.

---

#### Implementation — `POST /api/songs/:id/enrich` (`f49843a`)

Added endpoint in `materials-server/server.js` between the DELETE and POST song routes:

```js
app.post('/api/songs/:id/enrich', auth.requireAuth, async (req, res) => {
  const song = dbManager.getSongById(id);
  if (!song) return res.status(404).json({ error: 'Song not found' });
  const result = await enrichmentService.enrichSong(id);  // enrichmentService_sqlite
  if (result.success) {
    const updatedSong = dbManager.getSongById(id);
    res.json({ success: true, enrichmentCount: result.enrichmentCount,
               fields: result.fields, song: toCamelCase(updatedSong) });
  } else {
    res.json({ success: false, enrichmentCount: 0, message: result.message });
  }
});
```

`enrichmentService` in `server.js` is already aliased to `enrichmentService_sqlite` (line 9), so no import change was needed.

Verified locally against `craig_david_7_days`:
- 10 fields updated: chart peaks (AUS, FR, DE, IE, NL, NZ, SE, CH, UK), `last_enriched_utc`
- `lastfmPlays: 2396832`, `lastfmListeners: 380313`
- `wikipediaIntro`: "7 Days is a song by British singer Craig David…"
- `genre: R&B, Pop`, `bpm: 83`

---

#### Implementation — ManageSOUDatabase.js enrich button (`b88a23d`, frontend repo)

**State added:**
- `enrichingSongs` — `Set<songId>`: song IDs currently being enriched (disables button, shows ⏳)
- `enrichResults` — `Object<songId, {success, enrichmentCount, error}>`: per-song feedback

**Handler `handleEnrichSong(songId, e)`:**
1. Guards against double-click
2. POSTs to `/api/songs/${songId}/enrich` with `credentials: 'include'`
3. On success: merges `result.song` into local `songs` state and into `editFormData` (if the edit modal is open for that song)
4. On failure: records error in `enrichResults`
5. Always removes songId from `enrichingSongs` in finally block

**UI controls:**
- `⚡ action-col` column in song table (header: ⚡ tooltip, per-row button): shows ⏳ while enriching, ✅ on success, ❌ on failure
- `⚡ Enrich Song` button in edit modal footer (purple `btn-enrich` style): same feedback states; positioned between Cancel and Save Changes
- Pulse animation via CSS `@keyframes pulse` while enriching

---

#### Commits this session

| Commit | Repo | Change |
|---|---|---|
| `f49843a` | sou-backend | feat: POST /api/songs/:id/enrich — on-demand rich enrichment endpoint |
| `b88a23d` | sou-song-browser | feat: Enrich Song button — ⚡ row icon + modal button in ManageSOUDatabase |

---

#### Production Testing — ⚡ Enrich verified on Railway

**Railway env var audit completed.** All 8 required credentials confirmed present:
`SPOTIFY_CLIENT_ID`, `SPOTIFY_CLIENT_SECRET`, `LASTFM_API_KEY`, `YOUTUBE_API_KEY`, `GETSONGBPM_API_KEY`, `SOUNDCHARTS_APP_ID`, `SOUNDCHARTS_API_KEY`, `SESSION_SECRET` (strong random generated this session).

**`SESSION_SECRET` generated:** `sXgQQPxQ9LY/QXNrcqjCdPRlp+ngrSObl3BV22ObioaZ//9QMqfesYMZOBoM2VVH` — set in Railway dashboard. Previously the server was running without one (Express auto-generated ephemeral secret on each boot, invalidating all sessions on every redeploy).

**Production test — "Into the Groove" ⚡ Enrich:**
- 42 fields written: chart peaks (UK:#1, AUS:#1, IRE:#1, NL:#1 + 8 more), Wikipedia sections (background, composition, recording, reception), MusicBrainz release date, Last.fm plays/listeners, cover art (Spotify), genres (Wikipedia + Last.fm merged)
- BPM/key remain null — "Into the Groove" is not in GetSongBPM catalog (expected, documented gap)
- `lastfmPlays`/`lastfmListeners` both null (possible title mismatch — flagged as deferred investigation)

**Production test — "Africa" promote (Toto):**
- Lightweight enrichment on promote: `bpm:92`, `key:C♯m`, `mode:Minor`, `timeSignature:4/4`, `year:1982`, `spotifyTrackId` ✅
- Rich enrich: `coverArtUrl` ✅, `lastfmPlays:19428680`, `chartPeakUk:3` ✅

---

#### Null-overwrite safety (`ede2cc2`)

Two-layer defence added:
1. **`dbManager.updateSong()`** — strips `null`/`undefined` from updates object before SQL `UPDATE`. Defence-in-depth for all callers.
2. **`enrichmentService_sqlite.js`** — added `!song.bpm_best`, `!song.wikipedia_composition`, `!song.wikipedia_recording` guards (previously missing). `wikipedia_intro` guard added in same pass.

All other guarded fields were already correct. Last.fm plays/listeners intentionally always-overwrite (fresh metrics). See DECISIONS LOG entry.

**Commit:** `ede2cc2` on sou-backend.

---

#### `wikipedia_intro` extraction (`db46bfe`)

**`wikipediaChartScraper.scrapeMetadata()`** extended to extract the article lead section — all `<p>` tags before the first `<h2>`, keeping only those >60 chars (skips coordinate/date boilerplate paragraphs). Returns first qualifying paragraph as `metadata.intro`.

**`enrichmentService_sqlite.enrichSong()`** writes `metadata.intro` → `wikipedia_intro` DB column, guarded with `!song.wikipedia_intro`. Increments `enrichmentCount`.

**Tested locally:** "Into the Groove" → `'"Into the Groove" is a song by American singer Madonna, featured in the 1985 film Desperately Seeking Susan...'` ✅

**Commit:** `db46bfe` on sou-backend — pushed; Railway redeployed.

---

#### 13-song Spotify identity backfill

12 songs imported in the November 2025 bulk CSV batch predated Pipeline A (lightweight enrichment on promote). "Into the Groove" was promoted before Spotify credentials were on Railway. All 13 had null `spotifyTrackId`, `year`, `youtubeVideoId`, BPM/key, and genre.

**Backfill method:** Node.js script using `enrichmentService.enrichSong(title, artist)` locally (bypassing Railway for speed), then PUTting results to production for each song.

**Results:**

| Song | spotifyTrackId | year | BPM/key | genre | youtubeVideoId |
|---|---|---|---|---|---|
| All 13 | ✅ | ✅ | — | — | ✅ |
| 8 of 13 | — | — | ✅ | ✅ | — |
| 5 songs (A Minha Menina, Guantanamera, Hey There Delilah, Tadow, Texas Hold Em, Addams Family Theme) | — | — | ❌ not in GetSongBPM | varies | — |

**Complication:** Railway redeployed mid-run (commit `db46bfe` trigger) wiping MemoryStore → 9 songs got 401. Re-logged in and completed second pass. All 13 successfully backfilled.

---

#### Re-enrich all 13 for coverArtUrl + wikipedia_intro

After backfill gave all 13 songs a `spotifyTrackId`, and after `db46bfe` added `wikipedia_intro` extraction, all 13 were re-enriched via `POST /api/songs/:id/enrich`.

**Results:**
- 12/13 songs: ✅ `coverArtUrl` + ✅ `wikipedia_intro`
- "A Minha Menina" (Os Mutantes): ✅ `coverArtUrl`, ❌ `wikipedia_intro` — Os Mutantes Wikipedia article has no matching infobox structure for the scraper

---

#### All commits this session (continuation)

| Commit | Repo | Change |
|---|---|---|
| `f49843a` | sou-backend | feat: POST /api/songs/:id/enrich (carried from earlier in session) |
| `b88a23d` | sou-song-browser | feat: ⚡ Enrich Song button in ManageSOUDatabase |
| `ede2cc2` | sou-backend | fix: null-overwrite safety in dbManager + enrichmentService_sqlite |
| `db46bfe` | sou-backend | feat: extract and store wikipedia_intro (lead paragraph) |

---

#### Known Issue Flagged — Next Session

**Chord Numerals / Strum Style / Fingerpicking column data corruption:** In "Manage SOU Database" (local and production), these three columns display values that appear to be genre/tag data (`"R&B"`, `"Pop"`, `"Reggae Strum"`, `"Classic"`, `"Banger"`, `"Nostalgic"`, `"Political"`) often with stray leading/trailing quote marks (`'Pop"'`, `'"Reggae'`, `'Classic"'`). Strongly resembles the CSV column-shift corruption pattern already fixed in the `chords` field this year (`fixTeachingDataImport.py` / `sou_fix_discogs_columns.py` territory). Possibly a similar shift affecting these three columns across many of the 216 songs. **Next session: audit scope (% affected), identify shift pattern, propose and run fix.**

---

**Outstanding (carried forward):**
1. Upload Espresso PDF to R2 when local file is located
2. Investigate "Into the Groove" Last.fm null (plays/listeners null despite `LASTFM_API_KEY` set — possible title mismatch in Last.fm API)
3. ~~**Next session priority: Chord Numerals / Strum Style / Fingerpicking column data corruption**~~ ✅ **FIXED (13 June 2026) — see below**
4. Add Deezer as primary BPM source, GetSongBPM as secondary
5. AI system prompt tuning — SOU brand voice, teaching context
6. Multi-tutor auth migration (trigger: when second tutor account is needed)
7. Studio layout planning — resolve 4 open questions in `PROJECT_FEATURE_MAP.md`, then implement docked panel in `AdminLayout`

---

### 13 June 2026 — Teaching field corruption fix (Session 14)

**Attendees:** Matthew (user), GitHub Copilot (Claude Sonnet 4.6)

**Session focus:** Audit and fix the Known Issue flagged at end of Session 13 — Chord Numerals / Strum Style / Fingerpicking columns showing genre/tag values with stray quote marks.

---

#### Audit findings

Full audit of `chord_numerals`, `strum_style`, `fingerpicking_style` across all 216 songs:

| Column | Non-null in DB | Legitimate | Corrupted |
|---|---|---|---|
| `strum_style` | 68 | **3** | **65** (96%) |
| `fingerpicking_style` | 41 | **0** | **41** (100%) |
| `chord_numerals` | 19 | **2** | **17** (89%) |

**Root cause:** `importTeachingData.js` (Session 2) used naive `line.split(',')` to tokenise CSV rows. Any quoted field containing internal commas (SOU Keys `"Em, Am"`, Genre `"R&B, Pop"`, Tags `"R&B, Pop, Rap"`) produced extra tokens, shifting all subsequent fields rightward. Confirmed with token trace: "7 Days" `strum_style` received `' Pop"'` — the second half of the quoted `"R&B, Pop"` Tags field.

Shift is variable (not a fixed N-column offset) — the magnitude depends on how many commas are inside quoted fields to the left of the target column, so different songs show different corrupted values.

**Why `fixTeachingDataImport.py` didn't catch it:** The script's `build_patch()` only has corruption detectors for `level` and `num_chords`. For the three affected columns, when the CSV is empty (99%+ of rows) and the DB has something, the script silently skipped — leaving corrupted values in place.

**CSV ground truth** (Python `csv.DictReader` reads correctly):
- `strum_style`: 3 songs legitimately have `'Reggae Strum'` (Natural Mystic, Tadow, Waiting in Vain)
- `fingerpicking_style`: 0 songs — column was never populated in the CSV
- `chord_numerals`: 2 songs — `'I IV V'` (5 Years Time), `'I IV V vi'` (Beautiful Girls)

---

#### Fix applied

1. **Local DB:** 3 `UPDATE` statements run in a transaction — 65 + 41 + 17 = 123 rows cleared. Verified: keepers intact, Craig David `strum_style` now null.

2. **`dbManager.runMigrations()`:** Same 3 UPDATEs added as idempotent startup migrations (commit `2543dfb`). Runs automatically on every server start; matches 0 rows once DB is clean.

3. **`sou_songs_seed.db` refreshed** from corrected local DB.

4. **Pushed to Railway** (commit `2543dfb` → `sou-backend` main → auto-deploy triggered).

---

#### Commits this session

| Commit | Repo | Change |
|---|---|---|
| `2543dfb` | sou-backend | fix: clear strum/fingerpicking/chord_numerals corruption from naive CSV import |

---

#### ⚠️ Rule for future imports
Never use `String.split(',')` to parse CSV rows in this project. Always use `csv.DictReader` (Python) or `csv-parse` (Node.js). Quoted fields containing commas are common: Genre, Tags, SOU Keys, Teaching Notes.

---

**Outstanding (carried forward):**
1. ~~Verify production DB post-deploy~~ ✅ Done — all 5 keepers correct, Craig David null
2. Upload Espresso PDF to R2 when local file is located
3. ~~Investigate "Into the Groove" Last.fm null~~ ✅ Resolved — plays/listeners were already populated; tagsLastfmTrack gap is minor and will self-heal on next enrich
4. Add Deezer as primary BPM source, GetSongBPM as secondary
5. AI system prompt tuning — SOU brand voice, teaching context
6. Multi-tutor auth migration (trigger: when second tutor account is needed)
7. Studio layout planning

---

### 13 June 2026 — Season/Era/Month derivation wired into ⚡ Enrich (Session 14 cont.)

**Attendees:** Matthew (user), GitHub Copilot (Claude Sonnet 4.6)

**Context:** Follow-up to Into the Groove null field audit. `release_season`, `release_era`, `release_month` were populated for only ~52% of songs because the one-off `deriveReleaseMetadata.js` script was never called during promote or ⚡ Enrich — it had been run once during the original bulk CSV migration.

---

#### Fix: inline derivation in enrichSong()

Three pure-arithmetic helpers added to `enrichmentService_sqlite.js` (commit `08e9cd0`):
- `getEra(year)` — maps year → decade string (`'1980s'`, `'2000s'`, etc.)
- `extractMonthFromDate(dateStr)` — parses month from any date format ("July 15, 1985", "15 Jul 1985", ISO "1985-07-15", etc.)
- `getSeason(month)` — maps numeric month → `Spring/Summer/Fall/Winter`

New derivation block runs at the end of `enrichSong()`, before MERGE GENRES (no API dependency):
- `release_era`: only set if currently null (guard: `!song.release_era`)
- `release_month`: extracted from `release_date_consolidated → release_date_wikipedia → release_date_mb → release_date_spotify` waterfall, only if currently null
- `release_season`: derived from month, only if currently null

All three guards prevent overwriting manually-curated CSV values (e.g. Guantanamera's `era='1950s and earlier'` set intentionally by tutor).

**Tested locally:** A Teenager In Love (`'March 30, 1959'`) → month=3, season=Spring ✅  
**Verified on production:** Into the Groove post-⚡ Enrich → `season=Summer`, `era=1980s`, `releaseMonth=7` ✅

---

#### Last.fm / Into the Groove — closed

Investigated the Session 13 "Into the Groove Last.fm null" flag. Finding: `lastfmPlays` (4,795,184) and `lastfmListeners` (706,811) are in fact populated in production — they were written during the Session 13 re-enrichment pass. The only remaining gap is `tagsLastfmTrack` is null (artist tags are populated). Direct API test confirmed `track.getInfo` returns correct data — the null was a transient empty response during a specific run. Will self-heal on next enrich. No code change needed.

---

#### Commits this session (continuation)

| Commit | Repo | Change |
|---|---|---|
| `08e9cd0` | sou-backend | feat: derive release_era/season/month during enrichSong (no API calls) |

---

**Outstanding (carried forward):**
1. Upload Espresso PDF to R2 when local file is located
2. Add Deezer as primary BPM source, GetSongBPM as secondary
3. AI system prompt tuning — SOU brand voice, teaching context
4. Multi-tutor auth migration (trigger: when second tutor account is needed)
5. Studio layout planning

---

### 13 June 2026 — Seed Catalog Gap + Search UI Fix (Session 14 cont.)

**Attendees:** Matthew (user), GitHub Copilot (Claude Sonnet 4.6)

**Context:** "Like a Virgin" by Madonna not found when Matthew searched the "Search & Add Songs" admin page. Follow-on from Thread #3 started at end of Session 14.

---

#### Root cause: two separate issues

**Issue A — Missing Title search field (UI bug)**  
`SeedDatabaseSearch.js` had no Title input. The backend `seedService.search()` already supported a `title` filter (case-insensitive substring), but it was never exposed in the form. When Matthew typed "Like a Virgin" into the Artist field, zero results returned because no artist is named that.

**Issue B — Only 2022 remaster in catalog (data gap)**  
The canonical 1984 "Like a Virgin" single was absent. Only `Like A Virgin (7" Version) - 2022 Remaster` (source: `artist_playlists`, spotify ID `2plWClrDhcXKE4BcO4n0Zx`) was present — it was pulled when the "This Is Madonna" Spotify playlist was scraped (Spotify's official playlist features remasters over originals).

---

#### Fixes

**1. Title field added to `SeedDatabaseSearch.js`** (frontend change — Vercel deploy needed)

Three changes in one pass:
- Added `const [title, setTitle] = useState('');` state
- Wires into filter object: `if (title.trim()) filters.title = title.trim();`
- Title field rendered as first input in Search Criteria section (above Artist), with `onKeyDown Enter` handler

**2. Canonical 1984 entry added via `SEED_PATCHES` boot mechanism** (backend — Railway deploy)

**Critical discovery:** The Railway Volume mounts at `/app/data/` and shadows that entire directory. Committing an updated `data/expanded_seed_base.json` to git has **no effect** on an existing Railway deployment — the volume's copy wins every time. This is the same issue that was solved for `sou_songs.db` (seed copy outside `data/`), but the `expanded_seed_base.json` copy-on-boot guard only fires when the file is *absent*, not when it's *outdated*.

**Fix — SEED_PATCHES block in `seedService.js`** (mirrors `runMigrations()` pattern in `dbManager.js`):
```js
// Runs synchronously at module load before any request handling
const SEED_PATCHES = [ /* array of entries with spotifyId, title, artist, ... */ ];

if (existsSync(SEED_DB_PATH)) {
  const liveIds = new Set(liveData.map(s => s.spotifyId));
  const missing = SEED_PATCHES.filter(p => !liveIds.has(p.spotifyId));
  if (missing.length > 0) {
    missing.forEach(e => liveData.push(e));
    writeFileSync(SEED_DB_PATH, JSON.stringify(liveData));  // patches the volume file in place
  }
}
```

Idempotent — checked by `spotifyId` on every boot, no-op after first application. For all future manual canonical additions: add to `SEED_PATCHES`, commit, deploy.

New entry added via this mechanism:
```json
{
  "title": "Like a Virgin",
  "artist": "Madonna",
  "spotifyId": "1ZPlNanZsJSPK5h9YZZFbZ",
  "genres": ["pop", "dance", "female vocalists", "80s", "electronic", "dance-pop", "art pop", "electropop", "contemporary r&b"],
  "releaseYear": 1984,
  "popularity": { "spotify": 75 },
  "source": "manual_canonical",
  "discoveredDate": "2026-06-13"
}
```

Also updated `expanded_seed_base_seed.json` (the outside-`data/` bootstrap copy) so fresh volumes without any prior file also start with the entry. Total seed size: 47,273 songs.

---

#### Broader catalog investigation findings

- Spot-checked 20 major hits across 1980s–2020s: **20/20 found** — no systematic coverage gaps
- Remaster-only pattern confirmed for: "Vogue (Single Version)", "Material Girl", "Like a Virgin" (now fixed) — all three were pulled from the "This Is Madonna" Spotify playlist which defaults to 2022 remasters
- `Holiday` has both canonical + remaster (canonical came from a SOU playlist import later)
- The 47K catalog is comprehensive; isolated remaster-only cases are the exception not the rule

---

#### Verified (local simulation)

Simulated `seedService.search({title: 'Like a Virgin', limit: 10})` against the updated JSON:

| Title | Year | Popularity | Source |
|---|---|---|---|
| Like A Virgin (7" Version) - 2022 Remaster | 2022 | 41 | artist_playlists |
| Like a Virgin | 1984 | 75 | manual_canonical |

Both entries returned ✅

#### Commits & Deploy

| Commit | Repo | Change |
|---|---|---|
| `4e0ee34` | sou-song-browser → Vercel | feat: add Title search field to SeedDatabaseSearch |
| `7b8075d` | sou-backend → Railway | fix: add SEED_PATCHES mechanism; add Like a Virgin 1984 canonical entry |

Railway redeployed at 2026-06-13T18:06:03Z. Health check ✅. Seed patch runs synchronously at module load — fires on first restart, idempotent thereafter.

**Production verified ✅** Admin → Search & Add Songs → Title: "Like a Virgin" → 2 results: `Like a Virgin` (1984, pop=75) + `Like A Virgin (7" Version) - 2022 Remaster` (2022, pop=41).

#### Key architectural rule added

**Railway Volume shadows `data/` — updating committed JSON files in `data/` has no effect on existing deployments.** To deliver data changes to the volume, use the `SEED_PATCHES` pattern in `seedService.js` (check by `spotifyId`, append if missing, write back). This mirrors the `runMigrations()` pattern in `dbManager.js`. Any future manually-added canonical songs must go through `SEED_PATCHES`.

---

**Outstanding (carried forward):**
1. Upload Espresso PDF to R2 when local file is located
2. Add Deezer as primary BPM source, GetSongBPM as secondary
3. AI system prompt tuning — SOU brand voice, teaching context
4. Multi-tutor auth migration (trigger: when second tutor account is needed)
5. Studio layout planning

---

### 13 June 2026 — Session 14 Full Closing Summary

**Attendees:** Matthew (user), GitHub Copilot (Claude Sonnet 4.6)  
**Session type:** Bug-fix + pipeline wiring + seed catalog investigation

Four threads completed this session:

| # | Thread | Outcome |
|---|---|---|
| 1 | CSV column-shift corruption (strum_style / fingerpicking_style / chord_numerals) | ✅ Fixed — 123 rows cleared, 5 legitimate values retained, idempotent startup migration |
| 2 | Season / Era / Month not populating on newly-promoted songs | ✅ Fixed — derivation helpers wired into `enrichSong()`, verified on Into the Groove (Summer / 1980s / July) |
| 3 | Last.fm null on Into the Groove | ✅ Closed — plays (4,795,184) + listeners (706,811) already populated; null tag was transient, self-heals on next ⚡ Enrich |
| 4 | "Like a Virgin" not found in Search & Add Songs | ✅ Fixed — Title field added to UI; canonical 1984 entry added via new SEED_PATCHES boot mechanism; production verified |

**All commits this session:**

| Commit | Repo | Change |
|---|---|---|
| `2543dfb` | sou-backend | fix: clear strum/fingerpicking/chord_numerals corruption from naive CSV import |
| `08e9cd0` | sou-backend | feat: derive release_era/season/month during enrichSong (no API calls) |
| `7b8075d` | sou-backend | fix: add SEED_PATCHES mechanism; add Like a Virgin 1984 canonical entry |
| `4e0ee34` | sou-song-browser | feat: add Title search field to SeedDatabaseSearch |

**Key new architectural rules established this session:**
1. **Never `String.split(',')` for CSV** — always `csv.DictReader` (Python) or `csv-parse` (Node.js); quoted fields with internal commas are common in this dataset
2. **Enrichment must never overwrite non-null with null** — guards at two layers: `dbManager.updateSong()` strips null/undefined, `enrichmentService_sqlite.js` checks `!song.field` before assigning
3. **Railway Volume SEED_PATCHES pattern** — committed changes to `data/*.json` don't reach running Railway deployments; use `SEED_PATCHES` in `seedService.js` (mirrors `runMigrations()` in `dbManager.js`)

**Production state at session end:**
- 216 songs in teaching DB, all strum/fingerpicking/chord corruption cleared
- 47,273 songs in seed catalog
- ⚡ Enrich now derives release_era / release_season / release_month for any song with a known release date
- Search & Add Songs now searchable by title (in addition to artist, year, genre, popularity)
- All Railway + Vercel deploys live and healthy as of 2026-06-13T18:06:03Z

**Outstanding — carried to next session:**
1. Upload Espresso PDF to R2 when local file is located
2. Add Deezer as primary BPM source, GetSongBPM as secondary
3. AI system prompt tuning — SOU brand voice / teaching context (Gemini 2.5 Flash system prompt)
4. Multi-tutor auth migration (trigger: when second tutor account is needed)
5. Studio layout — resolve 4 open questions in `PROJECT_FEATURE_MAP.md` §Studio, then implement docked panel in `AdminLayout`

---

### 10 July 2026 — Deezer-first BPM implementation (Session 15)

**Attendees:** Matthew (user), GitHub Copilot (GPT-5.3-Codex)

**Session focus:** Resume engineering work and implement the previously outstanding BPM priority change: Deezer as primary source, GetSongBPM as fallback.

#### What was changed

1. **Lightweight promote pipeline updated (`enrichmentService.js`)**
    - Added `getDeezerData(title, artist)` using Deezer search + track endpoints.
    - Reordered enrichment flow:
       - Deezer first for BPM (and year fallback)
       - GetSongBPM as BPM fallback + key/mode/time-signature source
       - Spotify identity/year/genres unchanged
       - YouTube unchanged

2. **Rich on-demand pipeline updated (`enrichmentService_sqlite.js`)**
    - Added `getDeezerBpmData(title, artist)` helper.
    - Added explicit **DEEZER ENRICHMENT** block before GetSongBPM:
       - Writes `bpm_best`, `bpm_best_source='deezer'`, `bpm_best_confidence=0.93`, `bpm_best_retrieved_at`, `deezer_retrieved_utc`.
    - Updated GetSongBPM block semantics:
       - BPM only written if Deezer did not already provide `bpm_best`.
       - Still fetches/writes key, mode, time signature, and genres when needed.
       - Stores `bpm_getsongbpm` even when Deezer wins `bpm_best`.

#### Validation performed

- Syntax checks passed:
   - `node --check enrichmentService.js`
   - `node --check enrichmentService_sqlite.js`
- Editor diagnostics: no errors in modified files.
- Runtime smoke tests (live API calls):
   - `enrichmentService.getDeezerData('Into the Groove', 'Madonna')` → `{ tempo: 116.5, year: 2009 }`
   - `enrichmentService_sqlite.getDeezerBpmData('Into the Groove', 'Madonna')` → `{ tempo: 116.5 }`

#### Status at end of session

- ✅ Deezer-primary BPM implemented in both active enrichment paths.
- ⚠️ Not yet deployed to Railway in this session.

**Updated outstanding list:**
1. Upload Espresso PDF to R2 when local file is located
2. Deploy and production-verify Deezer-primary BPM behavior
3. AI system prompt tuning — SOU brand voice / teaching context (Gemini 2.5 Flash system prompt)
4. Multi-tutor auth migration (trigger: when second tutor account is needed)
5. Studio layout — resolve 4 open questions in `PROJECT_FEATURE_MAP.md` §Studio, then implement docked panel in `AdminLayout`

---

### 19 August 2026 — Production Outage Recovery, Git Repo Cleanup, Studio Rename (Session 16)

**Attendees:** Matthew (user), Claude (tech manager), GitHub Copilot (agent mode)
**Duration:** ~1.5 hours
**Context:** Live app found down at session start. Investigated and resolved, then handled a UI rename that surfaced an unrelated git structure issue.

**Task 1: Production outage — Railway trial expired**

- Symptom: Live app (`sou-song-browser.vercel.app`) showing "Failed to fetch," backend returning Railway's "Not Found" (service not routing)
- Root cause: Railway trial period ended — project (`caring-eagerness`, since renamed `SOU-app`) showed 0/1 service online, all deployment history marked "REMOVED"
- Fix: Upgraded to Railway Hobby plan ($5/month — appropriate given single-admin usage, no team seats needed). Redeployed `dukeofuke-png/sou-backend` repo from Railway dashboard. Confirmed all env vars survived the trial expiry. Build succeeded, domain restored.
- Verified: `/health` endpoint returns 200 OK; public song browser loads all 218 songs correctly; admin dashboard required re-login (expected — MemoryStore sessions wipe on redeploy, known issue #2 in Section 11)

**Task 2: UI label change — "Conversation" → "Studio Chat"**

- Simple rename in `AdminLayout.js`, uncovered a git structure issue in the process (see below)
- Change committed and pushed from the correct repo; confirmed live post-Vercel-deploy

**Discovery: Duplicate/stale outer git repo at workspace root**

- `/Users/matthew/Documents/SOU App/` (outer, `package.json` name `"sou-app"`) has its own independent `.git`, remote pointing to the same `sou-song-browser.git` URL as the real frontend repo — but with zero local commits ("unborn" branch) and a stale `src/` folder from Nov 2025 (predates the AdminLayout.js/ConversationWorkspace.js-era restructure)
- The **real, active frontend repo** is nested at `/Users/matthew/Documents/SOU App/sou-song-browser/` (`package.json` name `"sou-song-browser"`), confirmed in sync with `origin/main` (both at `4e0ee34`) and containing the correct, current 15+ commit history
- Outer repo appears to be an abandoned early setup, harmless as-is, but a trap for future confusion
- **Working rule going forward: always `cd` into `sou-song-browser/` specifically before any git operations on the frontend — never operate from the outer `SOU App/` root.**
- Outer repo cleanup (removing its stray `.git` folder) is a safe future task, not urgent — not done today

**Bug logged: STUDIO-01**

- Internal tool responses (raw `{"searchSongs_response": ...}` JSON blocks) are exposed directly in the Studio chat UI during multi-step searches, instead of being hidden behind a loading state (e.g. "Searching SOU library...") until the final answer renders
- Found during admin's own testing session with ChatGPT
- Not yet fixed — logged for future UI polish pass

**Next dev priority identified: PDF/lyric content search**

- Admin's real-world use case (finding "summer" songs by lyric/vibe/title, not just metadata) exposed a genuine capability gap: the AI (`searchSongs` tool) can only query structured SQLite fields (title, artist, genre, tags, season-from-release-date) — it has no access to PDF content or lyrics at all
- PDFs (Keynote-built chord/lyric sheets + Guitar Pro notation exports) are stored on Cloudflare R2 but never text-extracted; no content search layer exists
- Confirmed: all PDFs are native digital exports (Keynote/Guitar Pro), not scanned — so no OCR needed, but Guitar Pro notation exports may not yield clean extractable text the way Keynote chord/lyric sheets will
- Scoped as a new build: (1) PDF text extraction proof-of-concept on a mixed sample, (2) new DB field/table for extracted text, (3) batch extraction script across ~190 PDFs, (4) new AI tool (e.g. `searchSongContent`) wired into chat route, (5) testing
- Estimated ~4–6 hours in a single dev session, assuming clean text extraction — proof-of-concept step should run first to confirm
- **Agreed: this is priority #1 for the next development session.**

**Outstanding (carried forward):**
1. **PDF/lyric content search — priority #1 next session** (see above)
2. STUDIO-01 — tool response leakage in Studio chat UI
3. Outer stale git repo at `SOU App/` root — safe to clean up, not urgent
4. Espresso PDF missing from R2 (carried forward, longstanding)
5. Multi-tutor auth migration (trigger: second tutor onboarded — unchanged)

---

### 19 August 2026 — Production Outage Recovery, Git Repo Cleanup, UI Renames, Seed Catalogue AI Search/Promote, System Prompt Design Review (Session 17)

**Attendees:** Matthew (user), Claude (tech manager), GitHub Copilot (agent mode)
**Duration:** ~5 hours
**Context:** Live app found down at session start. Resolved, then handled UI renames, then built and tested a significant new AI capability (seed catalogue search/promote), then investigated a real production behavior discrepancy that led to identifying a fundamental design flaw in the existing system prompt.

**Task 1: Production outage — Railway trial expired**

- Symptom: Live app down, backend returning Railway's generic "Not Found" (service not routing)
- Root cause: Railway trial period ended — project (`caring-eagerness`, renamed `SOU-app` this session) showed 0/1 service online, deployment history marked "REMOVED"
- Fix: Upgraded to Railway Hobby plan ($5/month). Redeployed `dukeofuke-png/sou-backend` from Railway dashboard. All env vars survived. Build succeeded, domain restored, verified via `/health`, public song browser (218 songs), and admin dashboard (required re-login — expected, MemoryStore sessions wipe on redeploy, known issue #2 in Section 11)

**Task 2: UI label changes**

- "Conversation" → "Studio Chat", "Conversations" → "Chats", "Manage SOU Database" → "Manage SOU Catalog" (sidebar label + `ManageSOUDatabase.js` page header) — all in `sou-song-browser` frontend
- "SOU Catalog" adopted as the standing term for the 218-song teaching library going forward, to distinguish clearly from the 47K-song discovery/seed catalogue
- Uncovered and resolved a stale duplicate git repo at the outer `SOU App/` workspace root (separate `.git`, same remote URL, zero local commits, stale Nov 2025 files) — the real, active frontend repo is nested at `SOU App/sou-song-browser/`, confirmed in sync with `origin/main`. **Working rule established: always `cd` into `sou-song-browser/` specifically before any git operations on the frontend.** Outer repo cleanup deferred, not urgent.
- All three renames committed and pushed across two commits, confirmed live on Vercel.

**Bug logged: STUDIO-01, now confirmed live in production**

- Internal tool responses (raw `{"searchSongs_response": ...}` JSON) exposed directly in Studio chat UI during multi-step searches, instead of a loading state until the final answer renders
- Initially found via admin's own ChatGPT-assisted testing; independently reconfirmed later in this session with a concrete real example pulled directly from production conversation `ef3f3321-8e6e-4f6b-af05-339323d3ded9` (11:34–11:44 UTC) — a real tutor received a wall of raw, untruncated tool-response JSON as the assistant's answer. Not yet fixed.

**Task 3: AI seed-catalogue search + promote — built and tested this session**

Prompted by a real use case: admin wanted to find "summery" songs, exposing that `searchSongs` can only query the 218-song teaching library (SOU Catalog), not the 47K-song discovery catalogue, and cannot read PDF content at all (see Task 4).

**Built (in `services/seedService.js`, backend repo `materials-server`):**
- `searchSeedCatalog(query, filters)` — read-only, wraps existing `search()` filter logic, layers free-text query as OR-match on title/artist, caps at 12 results. **Bug found and fixed during testing:** initial version applied the free-text query against an artificially truncated 500-record candidate pool rather than the full 47,273-song catalogue, causing false "0 results." Fixed to run structured filters first (unbounded), then free-text query against the full filtered set, capping only at the end.
- `promoteSongFromSeed(spotifyId)` — single-song only (no bulk), reuses the exact same `dbManager.getSongsByTitleArtist()` duplicate-check, `dbManager.addSong()` insert, and `enrichmentService.js` auto-enrichment as the existing human "Promote" flow in `routes/seed.js`. Self-contained try/catch, returns `{success, alreadyExisted, songId, ...}` or `{success: false, error}`.

**Wired into AI (in `services/aiProvider.js` and `routes/chat.js`):**
- Both registered as Gemini-callable tools alongside existing `searchSongs`, with explicit tool-description language distinguishing the teaching library (218) from the discovery catalogue (47K)
- `promoteSongFromSeed` carries an explicit confirm-before-write rule in both its tool description and `BASE_SYSTEM_PROMPT`: only call after explicit tutor confirmation of a specific named song; never proactively or in an unconfirmed bulk loop
- **Correctness fix required for the new tools to function at all:** `executeTool()` was being called without `await` — harmless while only synchronous `searchSongs` existed, silently broken for the new `async` functions. Made `executeTool` async, awaited at the call site.
- Minor cosmetic fix applied: `[aiProvider] tool result:` console log always showed "0 songs" regardless of actual count, due to an `Array.isArray()` check against an object-shaped response. Fixed to report the real count.

**Testing — real findings, not just a clean pass:**

1. First live promote attempt failed: the model passed a **fabricated, plausible-looking `spotifyId`** rather than the real one from a prior `searchSeedCatalog` result. Failed safely (existing lookup correctly returned "not found," no incorrect write occurred) — but the root cause (hallucinated ID) was a genuine reliability concern.
   - **Mitigation applied:** explicit "copy the spotifyId verbatim, character-for-character; never reconstruct from memory" instruction added to both the tool description and `BASE_SYSTEM_PROMPT`. Retested successfully — exact ID match confirmed via log comparison.
2. **Significant finding, not yet fixed:** tool call/result data is not persisted anywhere. Only the AI's final natural-language text is saved to the `messages` table (schema: `id, conversation_id, role, content, created_at, model, token_count` — no tool-call column). No audit trail exists for any AI-driven search or promotion.
3. **Significant finding, not yet fixed, directly resulting from (2):** the AI can generate confident completion language (e.g. "I'm promoting it to the teaching library now") **without the corresponding tool call actually having been made.** Observed directly: chat text claimed an in-progress promotion while the backend log showed no `promoteSongFromSeed` call that turn. Chat text alone is not reliable evidence a write action occurred.
4. Duplicate-detection path (`alreadyExisted: true`) verified genuinely working, but only after two false starts caused by finding (3) above — first two "retry" attempts produced confident chat confirmations with no real tool call underneath; a third attempt, with an explicit unambiguous follow-up in the same thread, finally exercised the real code path. Confirmed via log (no `Enriching`/`Added song` lines) and DB query (exactly one row, unchanged).
5. Bulk/multi-song confirmation flow (e.g. "promote all the summer songs you found") — **not yet tested with a verified backend log.** Deferred given findings 2 and 3 should be addressed first.
6. Reconfirmed the seed catalogue's known data-model limitation: free-text query is title/artist substring matching only — no mood/season/vibe data exists for the 47K catalogue. "Summery" as a thematic request surfaces false positives (e.g. artist literally named "Summer Walker," a classical piece titled "Midsummer").

**Scope discussion — logged, not built this session:**

- Admin clarified the intended broader design: the AI should not treat the 47K seed catalogue as a finite world. Desired future capability: AI suggests songs from general/broader knowledge, checks whether they already exist in seed catalogue or SOU Catalog, and if not, can bring a new song into the seed catalogue via a real API lookup (e.g. Spotify) — not from AI memory alone, to avoid importing fabricated metadata. Distinct, larger capability beyond today's build; to be scoped as its own dedicated piece of work.

**Not committed or pushed this session:** all of Task 3's code changes (`services/seedService.js`, `services/aiProvider.js`, `routes/chat.js`) remain local, uncommitted in `materials-server`, deliberately — not considered production-ready given findings 2 and 3 above.

**Task 4: PDF/lyric content search — reconfirmed as priority, not built this session**

- Real-world task (finding "summer" songs by lyric/vibe/title) exposed that neither `searchSongs` nor `searchSeedCatalog` can access PDF or lyric content — no extraction pipeline exists
- Confirmed: all 190+ PDFs are native digital exports (Keynote chord/lyric sheets + Guitar Pro notation), not scanned — no OCR needed. Guitar Pro exports may not yield clean extractable text the way Keynote sheets will; worth confirming in the proof-of-concept step
- Estimated ~4–6 hours for a full build, assuming clean text extraction
- **Remains priority #1 for the next development session**

**Investigation: apparent inconsistency in AI behavior, root-caused with real production evidence**

- Admin noticed the AI complied with a general-knowledge brainstorming request in one live conversation but declined similar requests in later conversations, despite no code changes shipping in between
- Investigated via local git/DB checks first (inconclusive — local database ≠ production database, an important distinction surfaced during this investigation: Railway Volume-mounted production `sou_songs.db` is entirely separate from the local file, and is not reachable from a local terminal without the Railway CLI)
- **Resolved with real evidence** after installing Railway CLI and downloading a read-only copy of the production database: pulled the actual production conversation (`ef3f3321-8e6e-4f6b-af05-339323d3ded9`, 11:34–11:44 UTC, 10 messages/5 turns) behind the original screenshots
- Confirmed: at 11:41 UTC, `searchSeedCatalog` did not exist in production (still local/uncommitted) — when asked to "search beyond the SOU teaching library," the AI correctly had no tool for this, generated from general knowledge, and **accurately disclosed that** when directly challenged. Not a bug — an honest answer given real constraints at the time, and the direct real-world motivation for building `searchSeedCatalog` today.
- **One genuine inaccuracy found in this conversation:** two turns later, asked "what tool would you use to search the 47,000-song discovery database," the AI answered "I would use the searchSongs tool" — incorrect; `searchSongs` only ever queries the 218-song SOU Catalog, never the discovery catalogue. A real hallucination about its own tool capabilities.
- Checked `buildSystemPrompt()`'s 6-message reminder threshold as a possible explanation for the inconsistency: confirmed the reminder was active for the conversation's final two turns (including the inaccurate turn above), but this doesn't cleanly explain the pattern — the reminder didn't prevent the inaccuracy. Not a confirmed causal explanation, just an observed correlation.

**Major design finding: the existing "always use the tool, never use general knowledge" rule in `BASE_SYSTEM_PROMPT` is a disproportionate fix and needs a full rework**

- Traced to the 12 June tool-forgetting fix, which added a blanket instruction preventing the AI from ever answering from general knowledge when a question could relate to song recommendations — intended to stop the AI guessing about SOU's teaching library contents
- **Identified as the wrong fix, not just an incomplete one:** it solved a narrow accuracy problem (unverified claims about SOU's own data) by suppressing the AI's general usefulness entirely (brainstorming, discovery, general music knowledge) — confirmed today by real, inconsistent production behavior depending on phrasing and conversation history, not on the actual nature of each request
- **Agreed replacement design (three principles, not yet drafted into prompt wording):**
  1. **Full scope by default** — no blanket restriction on general knowledge, brainstorming, or open discussion
  2. **Verify before asserting SOU-specific facts** — anything stated as fact specifically about SOU's actual data (what's currently taught, what a record contains, database existence checks) must be grounded in a real tool lookup, not guessed. This is the only place a hard rule is justified, and it should be scoped narrowly to this case
  3. **Ask, don't assume, when ambiguous** — questions like "do we already teach X" are ambiguous in ways that change the correct answer (currently active? ever taught, even retired? could/should we teach it — a judgment call, not a lookup?). The AI should ask for clarification rather than silently pick an interpretation
- The message-26 "searchSongs can search the discovery catalogue" hallucination (above) folded into the same rework — the AI should be equally careful not to misstate its own tool capabilities, not just SOU's data
- **Deliberately deferred as its own dedicated piece of work** — actual prompt wording not yet drafted. Should be sequenced together with, and touches the same file (`routes/chat.js`) as, the audit-trail and chat-text-verification fixes from Task 3

**Discovery: unrelated, pre-existing uncommitted work found during the production-DB investigation**

- `enrichmentService.js` and `enrichmentService_sqlite.js` found modified in the backend repo, untouched by anyone this session
- Traced via `git diff` to match exactly the "10 July 2026 — Deezer-first BPM implementation (Session 15)" work already documented in this log — smoke-tested at the time but explicitly flagged as "not yet deployed to Railway," and has sat uncommitted in the working tree, untouched, for over a month
- `testMultiTurnToolUse.js` (untracked) almost certainly a leftover test script from the same session
- Deliberately not committed or reviewed further today — flagged as its own outstanding item

**Outstanding (carried forward, in priority order):**

*(Near-term items only — longer-range/phase-level items belong in `PRODUCT_ROADMAP.md`, not duplicated here.)*

1. **PDF/lyric content search** — priority #1, unchanged from last session
2. **System prompt rework** — replace the blanket "always use tool, never general knowledge" rule with the three-principle design above (full scope by default / verify SOU-specific facts / ask when ambiguous). Include fixing the tool-capability misstatement pattern (e.g. message 26's `searchSongs` error). Needs dedicated drafting session, testing against real scenarios including today's production conversations as regression cases — **✅ RESOLVED 30 August 2026, see entry below.**
3. **Persist tool call/result data** — no current audit trail for AI-driven searches or promotions; needed before the seed-catalogue search/promote feature can be trusted in real tutor use, and would also help diagnose future behavior questions like today's investigation far faster
4. **Fix or mitigate chat-text-vs-actual-action divergence** — AI can claim an action occurred when it didn't; likely addressed partly by (3) if the UI renders from persisted tool-result ground truth rather than trusting freeform AI text
5. Bulk/multi-song promote confirmation flow — retest with verified backend logs once (3) and (4) are addressed
6. Scope the broader "general knowledge suggestion + existence check + real-API import" capability as its own dedicated build
7. STUDIO-01 — tool response leakage in Studio chat UI, now confirmed live with a real production example
8. Review and decide on the dormant Deezer-first BPM work (`enrichmentService.js`, `enrichmentService_sqlite.js`, uncommitted since 10 July) — commit with fresh review, or deliberately discard
9. Outer stale git repo at `SOU App/` root — safe to clean up, not urgent
10. Espresso PDF missing from R2 (longstanding)
11. Multi-tutor auth migration (trigger: second tutor onboarded — unchanged)

---

### 20 August 2026 — Phase 0 Closeout (Audit Trail, STUDIO-01 Guard), Seed Search & Auto-Enrichment Fixes, Enrichment Pipeline Bug Fixes, Deezer BPM Commit (Session 18)

**Phase 0 — 3 of 4 items closed**

1. **Audit trail** — new `tool_calls` table (status: pending → completed/error;
   outcome: success/no_change/rejected/error). `executeToolWithAudit()` wraps every
   tool dispatch: INSERT before execution, UPDATE after. `TOOL_METADATA` flags which
   tools are mutations (currently just `promoteSongFromSeed`) — if the pending audit
   INSERT fails for a mutation tool, the mutation is blocked entirely; no write can
   happen unaudited. `conversationId` threads through `callAI` → `callGemini`/
   `callAnthropic`; both return `toolCallIds` so `routes/chat.js` can backfill
   `tool_calls.message_id` once the assistant message row exists.

2. **STUDIO-01 guard** — Gemini's narration turn after a tool call occasionally just
   echoes the raw tool result instead of describing it. `looksLikeRawToolJson()`
   detects this via two signals (either sufficient): parses as JSON with keys
   matching known tool-response fields, or near-exact match of the actual
   `toolResponse` from this turn. On detection: one controlled re-prompt for natural
   language; if that also fails, falls back to a synthesized sentence built from the
   structured result, logged via `console.warn`. Frontend backstop added in
   `ConversationWorkspace.js` (renders a fallback message if raw tool-JSON somehow
   still reaches the client). Both layers tested, including confirming ordinary
   narration mentioning numbers/song data never false-triggers.

3. **System prompt rework — NOT DONE.** Confirmed via full-text search of the live
   `BASE_SYSTEM_PROMPT`: the original blanket "always call searchSongs, never answer
   from general knowledge alone" rule is still live in production, word for word.
   The intended 3-principle replacement (general knowledge by default; ground
   SOU-specific factual claims in a real tool call; ask only when ambiguity would
   materially change the answer/data source/action; tool-capability honesty) was
   designed and agreed but never actually written into `routes/chat.js`. This remains
   the one open item from Phase 0's original scope — flagged as priority #1 for the
   next session.

   **Update 30 August 2026: ✅ RESOLVED — see the 30 August 2026 entry below. All four
   Phase 0 items are now closed.**

**Found and fixed (not originally scoped)**

4. **Seed catalogue search bug** — `searchSeedCatalog`'s free-text query used pure
   OR-substring matching against title/artist separately, so "`<title>` by `<artist>`"-
   shaped queries (e.g. "Hot Thing by Prince") returned zero results even when both
   the title and artist existed cleanly in the data. Fixed via `splitTitleByArtist()` —
   splits only when the query contains exactly one " by " delimiter, falls through
   to the original substring match otherwise (avoids ambiguous cases like titles
   that themselves contain "by"). Tool description and system prompt updated to
   prefer structured title/artist params when both are already known.

5. **Auto-enrichment background trigger** — promotion (both the human bulk-promote
   route and the AI's `promoteSongFromSeed`) now fires
   `enrichmentServiceSqlite.triggerBackgroundEnrichment()` fire-and-forget immediately
   after a successful promotion, rather than requiring a manual "⚡ Enrich" click. New
   columns: `enrichment_status`, `enrichment_last_attempt_at`, `enrichment_error`. New
   "Enrichment" badge column in `ManageSOUDatabase.js` (Complete/Failed/Enriching…).
   Explicitly out of scope for this pass: batch-backfill of existing promoted songs
   missing rich data, and automatic retry on failure.

6. **Enrichment pipeline bugs** (`enrichmentService_sqlite.js` /
   `wikipediaChartScraper.js`):
   - **Stale-snapshot guard bug** — every "only set if not already set" guard read
     the immutable song object captured at function-start instead of the
     in-progress `enrichmentData` object, so whichever data source ran last in code
     order silently won regardless of reliability (confirmed: MusicBrainz was
     overwriting correct Wikipedia/Soundcharts release dates purely by running
     later, not by being more accurate). Also broke Season/Era derivation the same
     way. Fixed via a shared `current(field)` helper (checks `enrichmentData` first,
     falls back to song), applied consistently across all 30 affected guard sites.
   - **Wikipedia `{{hlist}}` template bug** — multi-value infobox fields (genre,
     songwriters, labels, producers) render as `<li>` items with zero separator
     character in the extracted text at all (no comma, no `<br>`, nothing — the
     visible "·" separators are pure CSS). cheerio's `.text()` concatenated adjacent
     list items into one corrupted string (e.g. "R&Bfunksoulrockavant-pop"). Fixed
     via `extractListValues()`, which reads `<li>` items directly when present,
     falling back to the original comma/newline split for plain-text infoboxes.
     Verified against real raw HTML on real Wikipedia pages for all four affected
     fields, plus a real non-hlist fallback case.
   - Removed a dead, duplicate genre-merge block (confirmed dead via full
     execution-order trace before deletion — its output was always overwritten by a
     second, later block before ever being read).

7. **`spotify_track_id` passthrough bug** — `promoteSongFromSeed` already has a
   verified, correct `spotifyId` (used to look the song up in the seed catalogue), but
   `enrichSong(title, artist)` in `enrichmentService.js` never received it and instead
   tried to independently re-derive a Spotify match via its own title/artist search —
   which failed for both reported cases ("Hot Thing", "1999"), permanently blocking
   cover art and the Spotify Track link. Fixed: `enrichSong`/`enrichSongs`/
   `getSpotifyData` now accept an optional `knownSpotifyId` and fetch the track
   directly by ID when provided. Separately and more importantly: `spotifyTrackId`
   now initializes from `knownSpotifyId` as a baseline value and is only overwritten
   on a successful fetch — never reset to null just because a fetch failed. Verified
   live under a forced total fetch failure.

8. **Deezer-first BPM reordering** (originally implemented 10 July, Session 15, left
   uncommitted since then) — re-validated tonight against live Deezer/GetSongBPM
   APIs and against all of tonight's other changes to `enrichmentService_sqlite.js`
   (no conflicts found), then committed.

**Commits (11 total, all pushed and independently verified against a real fetch of origin/main)**

- **materials-server:** 6f7af60, e309306, 9c6d608, 6d2c3a8, 50edd77, 6546420, 3e41891, dfabb8b
- **sou-song-browser:** d776586, d3f8fca, eccc27c

**Still open for next session**

1. ~~**System prompt rework** (Phase 0 item 4) — see above, top priority.~~ ✅ **RESOLVED 30 August 2026 — see entry below.**
2. Duplicate `module.exports` in `wikipediaChartScraper.js` — found, not cleaned up.
3. `ManageSOUDatabase.js`'s admin table still displays raw `youtubeUrl` text in one
   column rather than using `youtubeVideoId` — the `SongDetailModal` fix didn't
   extend to this table.
4. Batch-backfill for existing promoted songs missing rich enrichment data.
5. git identity not explicitly configured in the `sou-song-browser` repo
   (auto-detected fallback used — not an error, just unset).

**Environment note**

A macOS sandbox permission issue recurred multiple times tonight during git
operations (commit and push) — "mktemp: mkstemp failed... Operation not permitted" /
Electron `codesign_util` `SecCodeCheckValidity` error. Each time, the underlying git
operation had actually completed successfully despite the error message — confirmed
via manual verification each time. A full VS Code/Copilot restart did not fully
resolve it (recurred once more after restart, then cleared on retry without further
intervention). If this recurs: check `git log`/`git status` first before assuming
failure; a manual terminal command (outside the sandbox) reliably works as a
fallback if verification shows the operation genuinely didn't complete.

---

### 26 August 2026 — TAB File R2 Migration & SongDetailModal Rendering Fix

**Bug fix — TAB files migrated to R2**

- Confirmed root cause: all 49 populated `melody_tab_path` values were still local
  macOS/Google Drive absolute paths — never migrated to R2, unlike `song_sheet_path`
  (190/191 already on R2). Frontend only rendered R2 links for paths starting with
  `https://`, so every TAB link fell through to `buildMaterialsUrl()`, which hits the
  now-retired `/materials/*` route (`410 Gone`) — every TAB link was broken for real
  users.
- Fixed via `materials-server/uploadTabsToR2.py`, adapted directly from the existing
  `uploadToR2.py` (which migrated song sheets). Same bucket (`sou-song-sheets`), same
  wrangler-based upload approach, same dry-run/`--apply` safety pattern. Uses a
  distinct object key, `{song_id}_tab.pdf`, so it can never collide with the
  song-sheet object (`{song_id}.pdf`) for the same song.
- Result: **49/49 uploaded successfully, 0 failures.** `melody_tab_path` now starts
  with `https://` for all 49 rows. `song_sheet_path` and `materials_json` were not
  touched (verified unchanged: 190/191 and 193 non-null respectively, same as before).
- Spot-checked 3 migrated URLs with real HTTP requests (not just DB values) — all
  returned `200`, `content-type: application/pdf`, valid `%PDF` file signature.

**Related rendering bug discovered and fixed — not part of the original root cause**

- Verifying the fix surfaced a second, independent bug: `SongDetailModal.js` only
  ever checked `songSheetPath.startsWith('https://')` to decide whether to render the
  R2 link block. When true, it rendered **only** the Song Sheet link and returned —
  the TAB link branches further down were unreachable code. Since 47 of the 49
  now-migrated TAB songs also have an `https://` `song_sheet_path`, the correct R2
  TAB URL existed in the database but was never rendered to the user, even after the
  migration above.
- Fixed by computing `hasR2SongSheet` and `hasR2Tab` independently and rendering each
  link conditionally inside the same block, so a song can show a Song Sheet link, a
  TAB link, both, or neither, based on its own data — no longer an either/or chain.
  Legacy `buildMaterialsUrl()` fallback branch left untouched (dead code, out of
  scope for this pass).
- Verified via the live `/songs` API response (not just source inspection) against 5
  real cases: 3 of the 47 previously-masked songs (now show both links) and the 2
  TAB-only songs with no song sheet (still correctly show only the TAB link — no
  regression).

**Outstanding, not addressed this session**

- `materials_json` still holds local Google Drive `absolutePath` values for all 193
  populated rows — never updated by either R2 migration script. Not read as a source
  of truth by the primary R2 render branch, but it is a second, stale representation
  of "this song's PDFs" that disagrees with `song_sheet_path`/`melody_tab_path`.
- Espresso song sheet still missing from R2 (longstanding, unchanged).

---

### 30 August 2026 — Phase 0 Closeout Complete: System Prompt Rework

**Phase 0 — all 4 items now closed.** Audit trail, STUDIO-01 guard, and seed-catalogue
tool validation were closed in Session 18 (see 20 August 2026 entry above). This entry
closes the fourth and final item, the system-prompt rework, flagged there as the one
remaining open item.

**What was replaced and why:** the blanket rule in `BASE_SYSTEM_PROMPT`
(`materials-server/routes/chat.js`) instructing the AI to always call `searchSongs`
before answering anything that could relate to SOU's song repertoire, and never answer
from general knowledge alone. This over-suppressed legitimate general-knowledge
brainstorming and discovery conversation unrelated to SOU's actual data — confirmed as
a real, inconsistent production behavior problem during the Session 18 investigation
(see above).

**New design — four principles now written into the prompt, in the model's own voice
rather than pasted as a bullet list:**
1. General knowledge is allowed by default — brainstorming, discovery, and general
   music/teaching discussion no longer require a tool call.
2. A real tool call is required before asserting any fact specifically about SOU's own
   data (teaching library or discovery catalogue) — never answered from memory or
   assumption, regardless of how many general-knowledge questions preceded it.
3. Ask for clarification, rather than silently guess, when a request is ambiguous in a
   way that would change the answer, data source, or action (e.g. "do we already teach
   X" — currently active vs. ever taught vs. should we teach it).
4. Tool-capability honesty — the AI must not imply `searchSongs` can reach the
   47,000-song discovery catalogue (that is `searchSeedCatalog`'s job), and must say so
   plainly if no available tool supports what's being asked, rather than fabricating a
   workaround.

**Verification performed before drafting:** full-text search confirmed `BASE_SYSTEM_PROMPT`
in `routes/chat.js` is the only live copy of the restrictive rule — no duplicate or
fallback prompt exists elsewhere in the codebase.

**Reviewed and deliberately left unchanged:**
- `promoteSongFromSeed`'s write-confirmation paragraph (item 3 in the prompt) — still
  requires explicit tutor confirmation before any promote call; untouched.
- The 6-message `REMINDER` block in `buildSystemPrompt()` — only fires for actual
  SOU-repertoire questions (keys, levels, genres, "what do we have"), which is still
  consistent with principle 2 above, not a contradiction of principle 1.
- Tool dispatch logic, `executeToolWithAudit()`, and the STUDIO-01 guard
  (`looksLikeRawToolJson()`) — out of scope for this task, not touched.

**Not addressed this session:** the tool-capability misstatement regression case from
Session 18 (message 26's `searchSongs` hallucination) has not been re-tested live
against production; recommend adding as a regression scenario next time production chat
is exercised.

Drafted and shown to admin for review before editing; wording approved as-is, no
revisions requested.

---

### 30 August 2026 — Phase 1 Built: Song-Sheet Content Extraction + searchSongContent Tool

**Scope:** song-sheet PDFs only (chord/lyric sheets). TAB/melody notation extraction is
explicitly out of scope — a 5-file proof-of-concept run earlier this session confirmed
Guitar Pro TAB exports extract as garbled, unreadable text (fret numbers/scale labels out
of visual order), while Keynote-built song sheets extract cleanly. That PoC also found
`pdf-parse@2.x` hard-crashes on this repo's Node 18 runtime (depends on `pdfjs-dist`,
which needs Node 20+/22+ `DOMMatrix`/`process.getBuiltinModule`) — `pdf-parse@1.1.1`
(classic CJS build) was used instead, confirmed working.

**1. Schema — `extracted_content` table**, added via the existing idempotent
`runMigrations()` pattern in `dbManager.js` (same mechanism that carried `tool_calls` to
production in Phase 0 — see 20 August entry above). **One deviation from the originally
approved schema, flagged and applied:** `song_id` is `TEXT` (not `INTEGER` as first
proposed) — `songs.id` is a TEXT slug primary key (e.g. `craig_david_7_days`), never an
integer; an `INTEGER` FK would never have matched. Verified idempotent (ran twice locally,
second run no-ops). Reaches production the same automatic way as prior migrations — no
separate deploy step, but not yet actually run against the production DB.

**2. Batch extraction — `extractSongSheetText.js`.** Dry-run mode by default (`--apply` for
real), idempotent via `INSERT ... ON CONFLICT(song_id, source_type) DO UPDATE` (preserves
row `id`/`created_at` across re-runs, cleaner than `INSERT OR REPLACE`). Classifies
`success` (≥200 chars, ≥20 words) vs. `low_confidence` (extracted but too short/sparse) vs.
`failed` (real exception, error text preserved verbatim). Read-only w.r.t. `songs` —
`song_sheet_path`/`melody_tab_path`/`materials_json` untouched throughout.

**Real run results — 191 songs with a non-null `song_sheet_path`:**
- `success`: 175
- `low_confidence`: 9
- `failed`: 7 — 1 expected (`sabrina_carpenter_espresso`, the already-logged Espresso
  gap — caught cleanly by an explicit pre-check rather than a raw fetch error), and
  **6 new findings**, added to outstanding items below.

**3. `searchSongContent` Studio tool** — new read-only AI tool, same pattern as
`searchSongs`/`searchSeedCatalog`. Plain `LIKE` substring match against
`extracted_content.extracted_text` (joined to `songs` for title/artist); FTS5 deliberately
deferred, not implemented this pass. Wired into `services/aiProvider.js` (tool
declaration, `TOOL_METADATA`, Gemini tool list, `executeTool()` dispatch) and
`routes/chat.js` (new numbered `BASE_SYSTEM_PROMPT` section, plus the Phase-0-rework
"verify before asserting SOU-specific facts" and tool-honesty paragraphs extended to
explicitly cover lyric/chord-content claims, not just catalogue metadata). Tool
description states plainly: song-sheet text only, no TAB, teaching-library-only (same 218
songs as `searchSongs`), substring match not smart/thematic search.

**`executeToolWithAudit()` checked, not modified:** confirmed it already handles read-only
tools cleanly — the mutation-blocking branch is gated entirely on `TOOL_METADATA[name].isMutation`,
which defaults `false` and only affects the wrapper's behavior when `true`. Flagging
`searchSongContent` as `{ isMutation: false }` was sufficient; no wrapper changes needed.

**Verified before considering this done:** smoke-tested `searchSongContent()` directly
against the real extracted data (not through the AI) — correctly found "7 Days" by Craig
David for `"subway"` and "All About That Bass" by Meghan Trainor for `"no treble"`, with
accurate snippets; a nonsense query correctly returned zero results.

**Not done this session:** the batch extraction has only been run against the local dev
DB — production's `extracted_content` table will exist after the next deploy (automatic
migration), but the 191-song batch job itself has not been run against production data.

**Outstanding — new item, separate from the longstanding Espresso gap:**

6 song-sheet PDFs are genuinely 0-byte empty objects on R2 (HTTP 200, `Content-Type:
application/pdf`, `Content-Length: 0` — confirmed via direct HEAD requests, not just the
extraction script's error message). Pre-existing data-integrity issue, unrelated to Phase 1
and not fixed here:

- `supergrass_alright` — "Alright" by Supergrass
- `radiohead_creep` — "Creep" by Radiohead
- `donna_summer_hot_stuff` — "Hot Stuff" by Donna Summer
- `scott_mckenzie_san_francisco` — "San Francisco" by Scott McKenzie
- `dodgy_staying_out_for_the_summer` — "Staying Out For The Summer" by Dodgy
- `mike_oldfield_tubular_bells_from_the_exorcist` — "Tubular Bells from The Exorcist" by Mike Oldfield

---

**END OF MASTER_ARCHITECTURE.MD**
