# SOU App Database & File Inventory
**Generated**: 22 November 2025, 21:12  
**Backup Created**: `/Users/matthew/Documents/SOU_App_Full_Backup_20251122_211240.tar.gz` (12MB)

---

## 1. PRIMARY DATABASES

### 1.1 Song Sheets Database (Main Product Database)
**File**: `Song Database/School of Uke Song Sheets Database.csv`  
**Rows**: 212 (211 songs + header)  
**Purpose**: Core database of songs with existing PDF sheet music  
**Headers**: Song Name, Artist, Release Date, Year, Major/Minor, Song Sheet, Original Key, SOU Keys, Level, TAB, Words & Music, Publisher, Genre, Tags, Era, Teaching Notes, Strum Style, Finger-picking Style, No. of Chords, Chords, Chord Numerals, Time SIgnature, BPM, Tempo, Month Released, Season, Notes  
**Status**: ✅ ACTIVE - Primary product database  
**Enrichment Flow**: Offline Python scripts → enriched CSV → exported JSON → consumed by frontend

### 1.2 Artist Seed List (Discovery Catalog)
**File**: `artist_seed_list.csv`  
**Rows**: 3,601 (3,600 artists + header)  
**Purpose**: Master artist catalog with track counts and tier classifications  
**Headers**: Artist, TrackCount, Add 5 Songs, Add 10 songs, Add 15 songs, Add 20 songs  
**Status**: ✅ ACTIVE - Primary artist discovery source  
**Backup**: `artist_seed_list_backup_1763757869706.csv` (1,018 rows - older snapshot)

### 1.3 Artist Playlist Links Master (Playlist URLs)
**File**: `materials-server/artist_playlist_links_master.csv`  
**Rows**: 650 (artists with playlist URLs)  
**Purpose**: Authoritative list of Spotify playlist URLs for artist seed expansion  
**Headers**: Artist, URL, Tier  
**Status**: ✅ ACTIVE & COMPLETE - Ready for track extraction  
**Validation**: 0 blank URLs, 0 invalid URLs, 1 intentional duplicate (Bronski Beat/Jimmy Somerville)  
**Created**: Today (22 Nov 2025) - Result of playlist discovery + manual curation

---

## 2. SEED DATABASES (Artist/Track Discovery Pool)

### 2.1 Primary Active Seed (19,137 Songs)
**File**: `materials-server/expanded_seed_base.json`  
**Size**: 5.6MB  
**Entries**: 19,137 songs  
**Modified**: 22 Nov 2025, 00:26  
**Purpose**: Main song discovery database with enriched metadata  
**Fields**: title, artist, spotifyId, genres (array), releaseYear, popularity (spotify score), source, discoveredDate  
**CSV Export**: `expanded_seed_base.csv` (19,137 rows with full metadata)  
**Status**: ✅ ACTIVE - This is your ~17k+ enriched song seed database  

**Sources breakdown**:
- Spotify followed artists: 1,051 songs
- Spotify catalog seed: 3,021 songs
- Spotify compilation: 1,906 songs
- SOU playlists (various courses): ~1,400 songs
- Curated lists (Rolling Stone 500, Billboard, etc.): ~50 songs
- Other sources: ~12,700 songs

**Enrichment status**: Most entries from Spotify sources have full metadata (genres, year, popularity). Curated list entries have basic fields only (title, artist, source).

### 2.2 Previous Seed Version (7,558 Songs)
**File**: `materials-server/mega_seed_discovered.json`  
**Size**: 2.3MB  
**Entries**: 7,558 songs  
**Modified**: 21 Nov 2025, 19:29  
**Status**: ⚠️ SUPERSEDED by expanded_seed_base.json - Earlier discovery snapshot

### 2.3 Seed Backups (Timestamped)
- `expanded_seed_base_backup_1763812203257.json` (710KB) - 22 Nov 11:50
- `expanded_seed_base_backup_1763771205466.json` (4.9MB) - 22 Nov 00:26
- `expanded_seed_base_backup_1763764432275.json` (3.4MB) - 21 Nov 22:33
- `expanded_seed_base_backup_1763755738142.json` (2.3MB) - 21 Nov 20:08

**Status**: ✅ ARCHIVE - Automatic backups from expansion operations

### 2.4 Seed Variants & Checkpoints
- `curated_seed.json` (991B) - Small curated starter seed
- `expanded_seed_10k.json` (7.9KB) - Expanded 10k variant
- `mega_seed_10k.json` (1.3KB) - Mega seed 10k variant
- `mega_seed_discovered_checkpoint.json` (2.6MB) - Discovery checkpoint

**Status**: ⚠️ PURPOSE UNCLEAR - May be obsolete or experimental variants; recommend archiving

### 2.4 Temporary/Working Files
- `expanded_seed_base_import_temp.json` (1.6MB) - Import staging
- `genre_enrichment_checkpoint.json` - Genre enrichment progress
- `expansion_checkpoint.json` - Expansion progress
- `artists_add_5_songs_playlist_checkpoint.json` - Playlist discovery checkpoint

**Status**: ⚠️ WORKING FILES - Should be cleaned after operations complete

---

## 3. TIER CSV FILES (Artist Classification by Catalog Size)

### 3.1 Active Tier Lists
**Location**: `materials-server/`

| File | Rows | Purpose | Status |
|------|------|---------|--------|
| `artists_add_20_songs.csv` | 56 | Artists with 20+ song catalogs | ✅ SOURCE |
| `artists_add_15_songs.csv` | 88 | Artists with 15+ song catalogs | ✅ SOURCE |
| `artists_add_10_songs.csv` | 192 | Artists with 10+ song catalogs | ✅ SOURCE |
| `artists_add_5_songs.csv` | 319 | Artists with 5+ song catalogs | ✅ SOURCE |
| `artists_final_search_results.csv` | 498 | Auto-discovered artists | ✅ SOURCE |

**Total Unique Artists**: ~650 (after deduplication)  
**Purpose**: Used to generate `artist_playlist_links_master.csv`  
**Status**: ✅ REFERENCE - Keep for audit trail

### 3.2 Flagged Variants (Album Detection)
- `artists_add_20_songs_flagged.csv` (56 rows)
- `artists_add_15_songs_flagged.csv` (88 rows)

**Purpose**: Artists where playlist URL might be an album link (flagged for review)  
**Status**: ⚠️ REDUNDANT - Already consolidated into master; can archive

---

## 4. OBSOLETE/DEPRECATED FILES

### 4.1 Playlist Discovery Artifacts
**Files**:
- `playlists_master_found.csv` (1 row - nearly empty)
- `playlists_missing.csv` (498 rows)
- `artist_playlist_links_master_missing.csv` (54 rows)

**Status**: ❌ OBSOLETE - These were intermediate working files during playlist discovery. All data is now in `artist_playlist_links_master.csv`. Can be archived or deleted.

### 4.2 Sample/Test Files
- `sample_import.csv` (4 rows)

**Status**: ❌ TEST FILE - Can be deleted

---

## 5. EXPORTED JSON (Frontend Consumption)

### 5.1 Song Database Exports
**Location**: `Song Database/data/`
- `songs_app_export_merged.json` - Primary export (merged/enriched)
- `songs_app_export.json` - Base export
- `genre_synonyms.json` - Genre normalization map

**Duplicates in**:
- `sou-song-browser/src/data/`
- `sou-song-browser/public/`

**Purpose**: JSON exports of song database consumed by React frontend  
**Status**: ✅ ACTIVE - Regenerated after CSV enrichment

---

## 6. WORKFLOW SUMMARY

### Current Data Flow
```
1. Artist Discovery:
   artist_seed_list.csv (3,600 artists)
   ↓
   Tier classification → artists_add_[X]_songs.csv
   ↓
   Playlist URL discovery (manual + auto)
   ↓
   artist_playlist_links_master.csv (650 artists with URLs) ✅ COMPLETE

2. Next Step (NOT YET DONE):
   artist_playlist_links_master.csv
   ↓
   Extract tracks from playlists
   ↓
   Merge into expanded_seed_base.json
   ↓
   Enrich & deduplicate
   ↓
   Eventually flow into Song Sheets Database

3. Song Database:
   School of Uke Song Sheets Database.csv (211 songs)
   ↓
   Offline Python enrichment (BPM, metadata, etc.)
   ↓
   Export to songs_app_export_merged.json
   ↓
   React frontend displays
```

---

## 7. FILES CREATED IN THIS SESSION (22 Nov 2025)

### Scripts Created Today
1. **`fillMissingPlaylistURLs.js`** - Spotify playlist search & fuzzy matching (enhanced multiple times)
2. **`validatePlaylistURLs.js`** - CSV validation for blank/invalid/duplicate URLs
3. **`mergeMissingIntoMaster.js`** - Safe merge utility (created but not used; manual merge done instead)
4. **`rebuildMasterPlaylistSheet.js`** - Tier aggregation script (created but not used; user chose different workflow)

### CSVs Created/Modified Today
1. **`artist_playlist_links_master.csv`** - Created from scratch; manually filled; now complete (650 rows)
2. **`artist_playlist_links_master_missing.csv`** - Working file for manual curation (54 rows, now obsolete)

### Backups Created
- `artist_playlist_links_master.csv.bak` - Created by sed when removing duplicate 2Pac entry

---

## 8. WHAT I DID (COMPLETE SESSION SUMMARY)

### Initial Request
- User wanted to automate playlist URL collection for ~650 artists across 4 tiers + auto-discovery
- Goal: Build a master sheet without overwriting manual entries

### Early Attempts
1. Built `rebuildMasterPlaylistSheet.js` to aggregate tier CSVs
2. Accidentally overwrote `playlists_master_found.csv` (was recoverable; tier CSVs intact)
3. Created full project backup

### New Workflow (User Decision)
- User created two new files as source of truth:
  - `artist_playlist_links_master.csv` (combined master)
  - `artist_playlist_links_master_missing.csv` (empty URLs only)
- Approach: Only write to "missing" file; never touch master

### Playlist Discovery Automation
1. Built `fillMissingPlaylistURLs.js` with Spotify API search
2. Iterative improvements:
   - Fuzzy matching ("This Is...", "Best of...", etc.)
   - Normalization (diacritics, punctuation, "&" → "and")
   - Market parameter (GB)
   - Increased search limit (10 → 50)
   - Owner preference (prefer Spotify for "This Is" playlists)
   - Name variants (U.S./US, L.A./LA, drop "feat.", drop "The")
   - Candidate logging for manual review
3. Multiple test runs on subsets, then full runs
4. Final automated run: Found 32/58 remaining artists

### Manual Completion
- Script failed to find "This Is..." playlists that actually existed (fuzzy matching too strict or API result ordering issues)
- User manually filled remaining 26 artists (98% had "This Is" playlists)
- User deleted automated results and redid manually

### Final Validation & Cleanup
1. Ran validator: Found 1 blank (2Pac duplicate entry)
2. Removed duplicate blank entry
3. Final validation: 650 artists, 0 blanks, 0 invalid, 1 intentional duplicate
4. **Status: COMPLETE**

---

## 9. TRUST ISSUES & CODE QUALITY PROBLEMS

### What Went Wrong
1. **Fuzzy matching too strict**: Script couldn't find obvious "This Is [Artist]" playlists
2. **API result ordering**: Spotify search results didn't prioritize official playlists reliably
3. **Overcomplicated heuristics**: Too many pattern variants; should have been simpler
4. **No incremental validation**: Ran long operations without confirming subset quality first
5. **File proliferation**: Created intermediate working files that became confusing

### Lessons for Clean Code Going Forward
1. **Simpler is better**: Fewer patterns, clearer matching logic
2. **Validate early**: Test on 5-10 rows before processing hundreds
3. **Single source of truth**: One master file, clear versioning
4. **Descriptive filenames**: Make purpose obvious (`_working`, `_final`, `_backup_YYYYMMDD`)
5. **Clean up artifacts**: Archive/delete intermediate files after completion

---

## 10. RECOMMENDED CLEANUP ACTIONS

### Safe to Archive/Delete
- `playlists_master_found.csv` (obsolete, 1 row)
- `playlists_missing.csv` (obsolete, consolidated into master)
- `artist_playlist_links_master_missing.csv` (obsolete, work complete)
- `artists_add_*_flagged.csv` (2 files, redundant with master)
- `sample_import.csv` (test file)
- `expanded_seed_base_import_temp.json` (temp file if import complete)
- Checkpoint JSON files if operations complete

### Keep for Reference
- All tier CSVs (`artists_add_[X]_songs.csv`) - audit trail
- `artists_final_search_results.csv` - audit trail
- `expanded_seed_base_backup_*.json` - version history
- `artist_seed_list_backup_*.csv` - version history

### Critical Active Files
- `artist_seed_list.csv` (3,600 artists)
- `artist_playlist_links_master.csv` (650 with URLs) ✅
- `expanded_seed_base.json` (5.6MB seed)
- `Song Database/School of Uke Song Sheets Database.csv` (211 songs)
- `songs_app_export_merged.json` (frontend consumption)

---

## 11. NEXT STEPS (USER APPROVED)

### Immediate Next Action
**Extract tracks from 650 playlists** and merge into seed database:

1. Read `artist_playlist_links_master.csv`
2. For each playlist URL, fetch all tracks via Spotify API
3. Deduplicate against `expanded_seed_base.json`
4. Write consolidated seed (e.g., `expanded_seed_from_playlists.json`)
5. Produce summary report: tracks added, duplicates skipped, errors

### After Seed Expansion
- Use enriched seed for discovery/recommendation features
- Eventually flow high-quality candidates → Song Sheets Database
- Create PDF sheet music for new additions

---

## 12. BACKUP CONFIRMATION

**Full Project Backup Created**:
- File: `/Users/matthew/Documents/SOU_App_Full_Backup_20251122_211240.tar.gz`
- Size: 12MB (compressed)
- Excludes: node_modules, .git
- Contains: All source code, databases, scripts, configs

✅ Safe to proceed with cleanup and next development phase.
