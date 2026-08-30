# Complete Data Pipeline Inventory

**Last Updated**: 23 November 2025

This document catalogs ALL data pipelines (Import, Enrichment, Export) for all three databases.

---

## Pipeline Type Definitions

- **IMPORT** - Getting raw data INTO a database
- **ENRICHMENT** - Querying APIs to ADD metadata to existing records
- **EXPORT** - Transforming/combining data for OUTPUT (JSON, CSV, frontend)

---

# SEED DATABASE PIPELINES

**Database**: `materials-server/data/expanded_seed_base.json` (38,437 songs)

## Import Pipelines

### ✅ 1. Playlist Import
**Script**: `materials-server/importPlaylistsToSeed.js`
**Status**: Production-ready
**Purpose**: Import songs from Spotify playlists (650+ playlists)
**Input**: `materials-server/artist_playlist_links_master.csv`
**Output**: Updates `expanded_seed_base.json` with new songs
**Data Added**:
- title, artist, spotifyId
- genres (from Spotify artist API)
- releaseYear (from album)
- popularity (Spotify track metric)
- source: "artist_playlists"
- discoveredDate: timestamp

**Features**:
- Deduplication by `title|artist` lowercase key
- Automatic backup before destructive operations
- Test mode: `--test --limit=N`
- Progress logging every 1 playlist
- Graceful failure handling (continues on errors)

**Last Run**: Nov 23, 2025 - Added 19,330 songs (69 failed playlists)

---

### ✅ 2. Chart Import (Manual)
**Script**: Various manual imports
**Status**: Ad-hoc
**Purpose**: Import songs from music charts (Rolling Stone 500, Billboard, etc.)
**Input**: Manually curated lists
**Output**: Updates `expanded_seed_base.json`
**Data Added**: Basic track info + source tag

---

### ✅ 3. Artist Seed Expansion (Dual API)
**Script**: `materials-server/expandCatalogDual.js`
**Status**: Production-ready (recently refactored)
**Purpose**: Find similar artists and their top tracks using Spotify + Last.fm
**Input**: `materials-server/artist_seed_list.csv`
**Output**: Updates `expanded_seed_base.json`
**APIs Used**:
- Spotify: Related artists + top tracks
- Last.fm: Similar artists

**Features**:
- Checkpoint system (saves progress every 10 artists)
- Test mode: `--test --limit=N`
- Configurable depth (how many similar artists to explore)
- Structured logging
- Retry logic with exponential backoff

---

### ✅ 4. Artist Seed Expansion (Auto)
**Script**: `materials-server/expandCatalogAuto.js`
**Status**: Production-ready (recently refactored)
**Purpose**: Automated expansion using Spotify only
**Input**: `materials-server/artist_seed_list.csv`
**Output**: Updates `expanded_seed_base.json`
**APIs Used**: Spotify only

**Features**:
- Fully automated (no manual intervention)
- Checkpoint system
- Test mode available
- Structured logging

---

### ⏳ 5. CSV Batch Import
**Script**: `materials-server/expandFromCSV.js`, `materials-server/expandFromCSV_batch.js`
**Status**: Exists but needs audit
**Purpose**: Import songs from external CSV files
**Input**: Custom CSV files
**Output**: Updates `expanded_seed_base.json`

---

## Enrichment Pipelines

### 🔄 1. Genre Enrichment (IN PROGRESS)
**Script**: `materials-server/enrichGenres.js`
**Status**: Under development (rate limiting issues)
**Purpose**: Add genres to songs missing genre data (29,846 songs, 6,945 unique artists)
**APIs**: Last.fm (primary) + Spotify (fallback)
**Strategy**: Artist-level lookup → propagate to all songs by that artist

**Current Issue**: Spotify rate limiting (429 errors)
**Next Step**: Switch to Last.fm primary

**Features**:
- Artist-level enrichment (efficient)
- Test mode: `--test --limit=N`
- Progress logging
- Verbose logging for debugging

---

### ⚠️ 2. Release Date Enrichment
**Script**: DOES NOT EXIST
**Status**: Needed
**Purpose**: Fill missing release years (67 songs = 0.2%)
**API**: Spotify track API
**Priority**: Low (99.8% coverage already)

---

### ⚠️ 3. Popularity Update
**Script**: DOES NOT EXIST
**Status**: Optional
**Purpose**: Refresh popularity scores (Spotify metric changes over time)
**API**: Spotify track API
**Priority**: Low (not time-critical)

---

## Export Pipelines

### ❌ 1. Seed to SOU Migration
**Script**: DOES NOT EXIST
**Status**: Needed
**Purpose**: Migrate curated songs from seed database → Manual SOU database
**Process**:
1. Admin reviews songs in seed database
2. Selects songs for SOU curriculum
3. Script exports selected songs to SOU format
4. Manual PDF creation + teaching notes added

**Requirements**:
- Admin UI to flag/select songs
- Export to CSV format matching Manual SOU schema
- Preserve basic metadata (title, artist, year, genre)

---

### ❌ 2. Seed Database JSON Export
**Script**: DOES NOT EXIST
**Status**: Optional
**Purpose**: Export seed database for admin dashboard UI
**Output**: JSON format for web interface
**Priority**: Medium (needed for admin features)

---

# SOU DATABASE PIPELINES

**Database**: `Song Database/data/songdb_master_v2_enriched.csv` (212 songs)

## Import Pipelines

### ✅ 1. Manual CSV Import
**Script**: Manual editing
**Status**: Active (tutors manually update CSV)
**Purpose**: Add new songs, update teaching metadata
**Input**: Manual SOU Database CSV
**Output**: Updated CSV with new rows or edited fields

**Fields Added Manually**:
- Songsheet, SOU Keys, Level, TAB
- Teaching Notes, Strum Style, Finger-picking Style
- Major/Minor, Original Key, Chords, Chord Numerals
- Time Signature, BPM, Tempo, No. of Chords
- PDF_ID (Google Drive link)

---

### ⏳ 2. Seed Database Migration (See Seed Export #1)
**Script**: DOES NOT EXIST
**Status**: Needed
**Purpose**: Import curated songs from seed database
**Priority**: High (workflow improvement)

---

## Enrichment Pipelines

### ✅ 1. Last.fm Tags (Track + Artist)
**Script**: `Song Database/tools/sou_enrich_lastfm_tags.py`
**Status**: Production-ready
**Purpose**: Fetch genre tags from Last.fm
**Input**: `songdb_master_v2_genres.csv`
**Output**: `data/songdb_master_v2_lastfm_tags.csv`

**APIs**: Last.fm API
- `track.getInfo` → top tags
- `artist.getInfo` → artist tags

**Columns Written**:
- `Last.fm Tags (Track)`
- `Last.fm Tags (Artist)`
- `Genres (Last.fm)` (cleaned/normalized)
- `Last.fm Source` (track/artist/both)
- `Last.fm Retrieved (UTC)`

**Features**:
- Idempotent (can re-run safely)
- Augments existing data (merge + dedupe)
- TEST_MODE flag
- Result: 197/212 songs enriched (93%)

---

### ✅ 2. Deezer Genres
**Script**: `Song Database/tools/sou_enrich_deezer.py`
**Status**: Production-ready
**Purpose**: Fetch genres from Deezer API
**Input**: Previous enrichment CSV
**Output**: `data/songdb_master_v2_deezer.csv`

**API**: Deezer artist API
**Columns Written**:
- `Genres (Deezer)`
- `Deezer Retrieved (UTC)`

**Features**:
- Idempotent
- No API key required
- Result: 0/212 songs (limited coverage for this dataset)

---

### ✅ 3. MusicBrainz + Last.fm Popularity
**Script**: `Song Database/tools/sou_enrich_mb_lastfm.py`
**Status**: Production-ready
**Purpose**: Fetch MusicBrainz tags and Last.fm popularity metrics
**Input**: Previous enrichment CSV
**Output**: Updated CSV

**APIs**:
- MusicBrainz: Artist tags
- Last.fm: Playcount, listeners

**Columns Written**:
- `Genres (MB Enrich)`
- `Last.fm Playcount`
- `Last.fm Listeners`

**Features**:
- 1-second rate limit (MusicBrainz requirement)
- Idempotent

---

### ✅ 4. Spotify Release Dates
**Script**: `Song Database/tools/sou_enrich_spotify_dates.py`
**Status**: Production-ready
**Purpose**: Fill missing release dates from Spotify
**Input**: Songs missing `ReleaseDate` field
**Output**: Updated CSV with dates

**API**: Spotify track search + track details
**Result**: 45 songs enriched initially

---

### ✅ 5. Spotify Audio Features
**Script**: `Song Database/tools/sou_enrich_spotify_features.py`
**Status**: Production-ready
**Purpose**: Get audio analysis (tempo, key, danceability, etc.)
**Input**: Songs with Spotify IDs
**Output**: Updated CSV

**API**: Spotify `/audio-features/{id}`
**Columns Written**:
- `Spotify BPM` (tempo)
- `Spotify Key`
- `Spotify Danceability`
- Other audio features

**Note**: BPM often inaccurate (can be 2x or halved)

---

### ✅ 6. Getsongbpm.com Scraping
**Script**: `Song Database/tools/sou_enrich_getsongbpm.py`
**Status**: Production-ready
**Purpose**: Scrape community-verified BPMs
**Input**: Songs missing BPM
**Output**: Updated CSV

**Method**: Web scraping (not API)
**Column Written**: `BPM (Getsongbpm)`
**Priority**: Override (most accurate BPM source)

---

### ✅ 7. Tunebat Scraping
**Script**: `Song Database/tools/sou_enrich_tunebat.py`
**Status**: Production-ready
**Purpose**: Scrape BPM, key, and other musical data
**Input**: Songs missing data
**Output**: Updated CSV

**Method**: Web scraping
**Columns Written**:
- `BPM (Tunebat)`
- `Key (Tunebat)`
- Other musical features

---

### ✅ 8. YouTube Link Enrichment
**Script**: `Song Database/tools/sou_enrich_youtube.py`
**Status**: Production-ready
**Purpose**: Find official YouTube videos
**Input**: Songs missing YouTube links
**Output**: Updated CSV

**API**: YouTube Data API v3
**Search**: `"{title}" {artist}` with fallbacks
**Columns Written**:
- `YouTube Link`
- `YouTube Video ID`
- `YouTube View Count`
- `YouTube Upload Date`
- `YouTube Channel Name`
- `YouTube Retrieved (UTC)`

**Rate Limit**: 10,000 quota units/day

---

### ⚠️ 9. Genius Lyrics
**Script**: DOES NOT EXIST (mentioned in docs)
**Status**: Planned
**Purpose**: Fetch song lyrics
**API**: Genius API
**Priority**: Override (manual lyrics take precedence)

---

### ⚠️ 10. Songwriter Enrichment
**Script**: DOES NOT EXIST
**Status**: Needed
**Purpose**: Fetch songwriter/composer credits
**APIs**: MusicBrainz (primary), Spotify (fallback)
**Priority**: Augment (merge with manual data)

---

## Export Pipelines

### ✅ 1. Genre Consolidation
**Script**: `Song Database/tools/sou_consolidate_enrichment.py`
**Status**: Production-ready
**Purpose**: Merge all enrichment CSVs into master database
**Input Files**:
- `data/songdb_master_v2_genres.csv` (base)
- `data/songdb_master_v2_lastfm_tags.csv`
- `data/songdb_master_v2_deezer.csv`

**Output**: `data/songdb_master_v2_enriched.csv` (90 columns, 214 rows)

**Process**:
1. Merge all per-source columns
2. Create `Genres (Best)` (normalized, deduplicated)
3. Create `Genres (Best Sources)` (provenance tracking)
4. Preserve all original columns

---

### ✅ 2. JSON Export for Frontend
**Script**: `Song Database/scripts/export_songs_merged.py`
**Status**: Production-ready
**Purpose**: Export enriched CSV → JSON for React frontend
**Input**: `data/songdb_master_v2_enriched.csv`
**Output**: `sou-song-browser/src/data/songs_app_export_merged.json`

**Process**:
1. Read enriched CSV (214 rows)
2. Filter to valid songs (212 exported)
3. Apply data transformations:
   - Parse genre arrays
   - Format dates
   - Clean text fields
4. Export to JSON with structured format
5. Include per-source tags/styles for developer use
6. Frontend displays `Genres (Best)` to tutors (clean, simple)

---

# PIPELINE GAPS & PRIORITIES

## High Priority (Needed Soon)

1. **Seed → SOU Migration Tool**
   - Purpose: Move curated songs from seed to SOU database
   - Blocker: Manual workflow inefficient
   - Effort: Medium (admin UI needed)

2. **Seed Genre Enrichment Fix**
   - Purpose: Complete genre enrichment (29,846 songs)
   - Blocker: Rate limiting issues
   - Effort: Small (switch to Last.fm)

3. **Songwriter Enrichment (SOU)**
   - Purpose: Add songwriter credits
   - Blocker: Missing data
   - Effort: Small (similar to existing enrichment scripts)

## Medium Priority (Nice to Have)

4. **Genius Lyrics (SOU)**
   - Purpose: Fetch lyrics for teaching
   - Blocker: Not critical for initial launch
   - Effort: Small (API well-documented)

5. **Seed Admin Dashboard Export**
   - Purpose: JSON export for admin UI
   - Blocker: Admin dashboard not built yet
   - Effort: Small (similar to SOU export)

6. **Popularity Refresh (Seed)**
   - Purpose: Update Spotify popularity scores
   - Blocker: Not time-critical
   - Effort: Small (batch update)

## Low Priority (Future)

7. **Automated PDF Matching**
   - Purpose: Auto-link PDF sheets to songs
   - Blocker: PDFs manually curated
   - Effort: Large (filename matching, fuzzy search)

8. **Release Date Backfill (Seed)**
   - Purpose: Fill 67 missing release years (0.2%)
   - Blocker: Already 99.8% coverage
   - Effort: Small (batch Spotify query)

---

# PIPELINE STANDARDS

## Code Quality Requirements

All pipelines must follow these standards:

### 1. Imports & Dependencies
```javascript
// Node.js
require('dotenv').config();
const config = require('./config');
const { logInfo, logSuccess, logProgress, logError, logWarning, retryWithBackoff } = require('./errorHandler');
const { parseCSVSync, writeCSV } = require('./csvUtils');
```

```python
# Python
import os
from dotenv import load_dotenv
import csv
import logging
# Use centralized utility modules
```

### 2. Configuration
- All file paths from `config.js` (Node) or centralized config (Python)
- All rate limits from `config.rateLimits`
- All API keys from `.env` file
- NO hardcoded paths, credentials, or delays

### 3. Logging
- Use structured logging (logInfo, logSuccess, logProgress, logError)
- Progress updates every N items (N=50 for production, N=5 for test mode)
- Error context included in all error logs
- Summary statistics at completion

### 4. Error Handling
- Use `retryWithBackoff` for all API calls
- Graceful degradation (continue on single failures)
- Catch and log errors with context
- Never fail silently

### 5. Features
- **Test mode**: `--test` flag with `--limit=N`
- **Idempotent**: Can re-run without duplicating data
- **Backups**: Automatic backup before destructive operations
- **Checkpoints**: Save progress for long-running operations
- **Validation**: Validate env vars at startup

### 6. Documentation
- JSDoc comments for all functions
- Script header with purpose, usage, example
- Dependencies listed
- API endpoints documented

---

# NEXT STEPS

1. ✅ Update DATABASE_TERMINOLOGY.md with Manual SOU database
2. ✅ Update API_ENRICHMENT_STRATEGY.md with field priorities
3. ✅ Create this pipeline inventory
4. ⏳ Audit existing pipelines for code quality
5. ⏳ Refactor non-compliant pipelines
6. ⏳ Create missing high-priority pipelines
7. ⏳ Test all pipelines end-to-end
8. ⏳ Document operational procedures (when to run each pipeline)
