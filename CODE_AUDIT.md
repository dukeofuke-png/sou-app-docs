# SOU App Code Audit & Cleanup Plan
**Date**: 22 November 2025  
**Purpose**: Comprehensive audit of all scripts with cleanup recommendations

---

## CATEGORY 1: CORE ACTIVE SERVICES (Keep & Maintain)

### 1.1 Main Server
**File**: `server.js`  
**Purpose**: Express backend serving API endpoints, PDF proxy, health checks  
**Status**: ✅ ACTIVE - Core runtime service  
**Quality**: Review needed for error handling and auth middleware

### 1.2 Core Services (Used by Server)
| File | Purpose | Status |
|------|---------|--------|
| `spotifyService.js` | Spotify API integration (search, popularity) | ✅ ACTIVE |
| `lastfmService.js` | Last.fm API integration (listener counts) | ✅ ACTIVE |
| `searchService.js` | Song search logic | ✅ ACTIVE |
| `enrichmentService.js` | Metadata enrichment | ✅ ACTIVE |
| `chartService.js` | Chart position aggregation | ✅ ACTIVE |
| `wikidataChartService.js` | Wikidata chart queries | ✅ ACTIVE |
| `wikipediaChartScraper.js` | Wikipedia chart scraping | ✅ ACTIVE |
| `auth.js` | Admin authentication | ✅ ACTIVE |
| `advancedSearchHelpers.js` | Advanced search utilities | ✅ ACTIVE |
| `searchSessionStore.js` | Session management for searches | ✅ ACTIVE |
| `popularityAggregator.js` | Popularity metric aggregation | ✅ ACTIVE |
| `popularityScore.js` | Popularity scoring algorithm | ✅ ACTIVE |
| `csvManager.js` | CSV read/write utilities | ✅ ACTIVE |
| `importService.js` | Bulk import with preview/commit | ✅ ACTIVE |
| `admissionPolicy.js` | Rules for promoting songs to main DB | ✅ ACTIVE |
| `seedCatalogue.js` | In-memory seed catalogue management | ✅ ACTIVE |
| `aiAssistant.js` | AI assistant integration | ✅ ACTIVE (?) |

**Action**: Review each for error handling, hardcoded paths, consistent patterns

---

## CATEGORY 2: SEED EXPANSION SCRIPTS (Active Utilities)

### 2.1 Artist/Track Discovery
| File | Purpose | Status |
|------|---------|--------|
| `addFollowedArtists.js` | Add Spotify followed artists to seed | ✅ UTILITY |
| `expandCatalogAuto.js` | Auto expand catalog from seed | ✅ UTILITY |
| `expandCatalogDual.js` | Dual-source expansion (Spotify + Last.fm) | ✅ UTILITY |
| `expandFromCSV.js` | Expand seed from CSV artist list | ✅ UTILITY |
| `expandFromCSV_batch.js` | Batch version of CSV expansion | ✅ UTILITY |
| `generateExpandedSeed.js` | Generate expanded seed | ✅ UTILITY |
| `mergeSeeds.js` | Merge multiple seed files | ✅ UTILITY |

**Action**: Standardize error handling, rate limiting, checkpoint/backup patterns

### 2.2 Playlist Import & Discovery
| File | Purpose | Status |
|------|---------|--------|
| `importSOUPlaylists.js` | Import SOU course playlists | ✅ UTILITY |
| `importPublicPlaylists.js` | Import public playlists by URL | ✅ UTILITY |
| `importArtistPlaylistsFromCSV.js` | Import artist playlists from CSV | ✅ UTILITY |
| `autoFindThisIsPlaylists.js` | Auto-discover "This Is" playlists | ✅ UTILITY |
| `buildPlaylistReports.js` | Generate found/missing playlist reports | ✅ UTILITY |
| `extractArtistLists.js` | Extract artist lists from various sources | ✅ UTILITY |

**Action**: Review for redundancy; some may overlap with today's fillMissingPlaylistURLs.js

### 2.3 Enrichment & Quality
| File | Purpose | Status |
|------|---------|--------|
| `enrichGenres.js` | Genre enrichment for seed entries | ✅ UTILITY |
| `duplicateDetector.js` | Detect duplicate songs (fuzzy matching) | ✅ UTILITY |
| `flagAlbumLinks.js` | Flag playlist URLs that might be albums | ✅ UTILITY |

**Action**: Ensure deduplication logic is consistent across all scripts

---

## CATEGORY 3: TODAY'S SESSION SCRIPTS (New - Need Review)

### 3.1 Playlist URL Discovery (Created Today)
| File | Purpose | Quality | Action |
|------|---------|---------|--------|
| `fillMissingPlaylistURLs.js` | Auto-fill missing playlist URLs via Spotify search | ⚠️ POOR | **Rewrite or archive** - Fuzzy matching failed; over-complicated |
| `validatePlaylistURLs.js` | Validate CSV for blank/invalid/duplicate URLs | ✅ GOOD | Keep - Simple, focused utility |
| `mergeMissingIntoMaster.js` | Safe merge of missing into master CSV | ⚠️ UNUSED | Archive or test - Created but not used |
| `rebuildMasterPlaylistSheet.js` | Aggregate tier CSVs into master | ⚠️ UNUSED | Archive - Workflow changed; not used |
| `runFinalPlaylistSearch.js` | Run final playlist search | ⚠️ UNCLEAR | Review - May be duplicate of fillMissing |

**Issues with fillMissingPlaylistURLs.js**:
- Over-complicated pattern matching (7+ patterns)
- Failed to find 98% of "This Is" playlists that actually existed
- Too many name variants without testing subset first
- No incremental validation

**Recommendation**: 
- Archive `fillMissingPlaylistURLs.js` as example of what NOT to do
- Keep `validatePlaylistURLs.js` - it's clean and useful
- Archive unused merge scripts

---

## CATEGORY 4: TEST & DEBUG SCRIPTS (Keep Separate)

| File | Purpose | Status |
|------|---------|--------|
| `chartService.test.js` | Unit tests for chartService | ✅ TEST |
| `testExpansion.js` | Test expansion logic | ✅ TEST |
| `testImportPreview.js` | Test import service | ✅ TEST |
| `testSpotifySearch.js` | Test Spotify search | ✅ TEST |
| `verifySpotifyCredentials.js` | Verify API credentials | ✅ TEST |

**Action**: Move to `materials-server/test/` subdirectory for organization

---

## CATEGORY 5: OBSOLETE/UNCLEAR FILES (Review or Archive)

### 5.1 Potential Duplicates
- `expandCatalogAuto.js` vs `expandCatalogDual.js` - Which is primary?
- `expandFromCSV.js` vs `expandFromCSV_batch.js` - Batch just a wrapper?
- `autoFindThisIsPlaylists.js` vs `fillMissingPlaylistURLs.js` - Overlap?

### 5.2 Possibly Obsolete (Based on Today's Work)
- `buildPlaylistReports.js` - May have been for earlier playlist discovery workflow
- `runFinalPlaylistSearch.js` - Purpose unclear; may duplicate fillMissing

**Action**: Review each pair; keep one, archive the other OR clarify distinct use cases

---

## CODE QUALITY ISSUES IDENTIFIED

### Issue 1: CSV Parsing Inconsistency
**Problem**: Multiple custom CSV parsers across different scripts  
**Files**: `fillMissingPlaylistURLs.js`, `mergeMissingIntoMaster.js`, `validatePlaylistURLs.js`, `rebuildMasterPlaylistSheet.js`, `csvManager.js`  
**Fix**: Centralize in `csvManager.js`; all scripts should use same parser

### Issue 2: No Consistent Error Handling
**Problem**: Scripts mix `process.exit(1)`, thrown errors, silent failures  
**Fix**: Standardize error handling pattern:
```javascript
try {
  // operation
} catch (err) {
  console.error('❌ Error in [operation]:', err.message);
  process.exit(1);
}
```

### Issue 3: Hardcoded Paths
**Problem**: Some scripts have hardcoded file paths instead of using `__dirname` or config  
**Fix**: Audit all file I/O; ensure paths are configurable or relative to `__dirname`

### Issue 4: No Input Validation
**Problem**: Scripts don't validate environment variables or file existence before proceeding  
**Fix**: Add validation at start of main() function

### Issue 5: Checkpoint/Backup Inconsistency
**Problem**: Some scripts create timestamped backups, others don't; naming inconsistent  
**Fix**: Standardize backup naming: `[filename]_backup_[timestamp].[ext]`

### Issue 6: Rate Limiting Varies
**Problem**: Different scripts use different delay values (100ms, 200ms, etc.)  
**Fix**: Define SPOTIFY_RATE_LIMIT_MS in .env; use consistently

### Issue 7: Logging Inconsistency
**Problem**: Mix of console.log, console.error, emoji prefixes, plain text  
**Fix**: Standardize logging:
- `console.log('✅ Success')` for completion
- `console.log('ℹ️ Info')` for progress
- `console.error('❌ Error')` for failures
- `console.log('⚠️ Warning')` for issues

---

## CLEANUP PLAN

### Phase 1: Archive Obsolete Files
Create `materials-server/archive/` and move:
- `fillMissingPlaylistURLs.js` (failed experiment)
- `rebuildMasterPlaylistSheet.js` (unused)
- `mergeMissingIntoMaster.js` (unused)
- Any duplicate/unclear scripts after review

### Phase 2: Organize Test Files
Create `materials-server/test/` and move:
- `chartService.test.js`
- `testExpansion.js`
- `testImportPreview.js`
- `testSpotifySearch.js`
- `verifySpotifyCredentials.js`

### Phase 3: Standardize Core Utilities
For each script in Category 2 (Seed Expansion):
1. Add descriptive header comment with purpose, usage, example
2. Validate inputs (env vars, file paths) at start
3. Use centralized CSV parser from csvManager
4. Standardize error handling
5. Ensure consistent backup/checkpoint naming
6. Use .env for rate limits and config

### Phase 4: Review Core Services
For each script in Category 1:
1. Audit error handling in API calls
2. Remove any hardcoded paths or credentials
3. Ensure consistent logging
4. Add JSDoc comments for exported functions
5. Test critical paths

### Phase 5: Clean Up Working Files
Delete or archive:
- Empty/placeholder CSV files (e.g., `sample_import.csv`)
- Obsolete intermediate CSVs (e.g., `playlists_master_found.csv`, `playlists_missing.csv`)
- Checkpoint JSON files if operations complete
- Temporary files (e.g., `expanded_seed_base_import_temp.json`)

---

## RECOMMENDED DIRECTORY STRUCTURE

```
materials-server/
├── server.js                 # Main Express server
├── package.json
├── .env
├── services/                 # Core reusable services
│   ├── spotifyService.js
│   ├── lastfmService.js
│   ├── chartService.js
│   ├── enrichmentService.js
│   ├── searchService.js
│   └── ...
├── utilities/                # Standalone utility scripts
│   ├── csvManager.js
│   ├── duplicateDetector.js
│   ├── validatePlaylistURLs.js
│   └── ...
├── scripts/                  # Runnable scripts for operations
│   ├── seed-expansion/
│   │   ├── addFollowedArtists.js
│   │   ├── expandCatalogAuto.js
│   │   └── ...
│   ├── playlist-import/
│   │   ├── importSOUPlaylists.js
│   │   ├── autoFindThisIsPlaylists.js
│   │   └── ...
│   └── enrichment/
│       ├── enrichGenres.js
│       └── ...
├── test/                     # Test scripts
│   ├── chartService.test.js
│   ├── testExpansion.js
│   └── ...
├── archive/                  # Obsolete/unused scripts
│   ├── fillMissingPlaylistURLs.js
│   └── ...
├── data/                     # Data files
│   ├── expanded_seed_base.json
│   ├── expanded_seed_base.csv
│   ├── artist_playlist_links_master.csv
│   └── ...
└── backups/                  # Timestamped backups
    └── [auto-generated]
```

---

## IMMEDIATE ACTIONS (Priority Order)

### 🔴 Priority 1: Safety & Stability
1. ✅ **DONE**: Full project backup created
2. ✅ **DONE**: Database inventory documented
3. **TODO**: Identify and archive truly obsolete scripts
4. **TODO**: Remove empty/placeholder files

### 🟡 Priority 2: Code Quality
5. **TODO**: Centralize CSV parsing in csvManager.js
6. **TODO**: Standardize error handling across all scripts
7. **TODO**: Add input validation to all runnable scripts
8. **TODO**: Document purpose/usage at top of each script

### 🟢 Priority 3: Organization
9. **TODO**: Create directory structure (services/, scripts/, test/, archive/)
10. **TODO**: Move files to appropriate directories
11. **TODO**: Update any import paths after reorganization
12. **TODO**: Create README.md for each subdirectory

---

## NEXT DEVELOPMENT PHASE

Once cleanup is complete, ready to proceed with:
1. **Playlist Track Extraction**: Build script to read `artist_playlist_links_master.csv`, fetch tracks from 650 playlists, merge into `expanded_seed_base.json`
2. **Quality Assurance**: Ensure new script follows standardized patterns from cleanup
3. **Testing**: Validate on subset before full run
4. **Documentation**: Clear usage instructions and error handling

---

## SUMMARY

- **Total Scripts**: 44 JavaScript files
- **Core Services**: ~17 files (active, need quality review)
- **Utilities**: ~15 files (active, need standardization)
- **Test Scripts**: 5 files (move to test/)
- **Today's Scripts**: 5 files (1 keep, 4 archive/review)
- **Unclear/Duplicate**: 3-5 files (need clarification)

**Estimated Cleanup Time**: 2-3 hours of focused work
**Risk**: Low (backup exists, changes are organizational)
**Benefit**: Maintainable codebase, clear structure, fewer mistakes going forward
