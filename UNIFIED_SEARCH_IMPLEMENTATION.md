# Unified Advanced Search Implementation

## Overview
Rebuilt the admin search interface to support comprehensive multi-criteria queries without tabs, with fuzzy chart position parsing and source transparency badges.

## Frontend Changes (`sou-song-browser/src/components/SongSearch.js`)

### State Variables (Unified Form)
- `artistName` - Artist name search
- `tag` - Tag/keyword search  
- `genre` - Genre filter
- `yearStart` / `yearEnd` - Year range
- `chartPosition` - Fuzzy chart text ("No.1", "Top 10", "#1", etc.)
- `songwriter` - Songwriter name
- `season` - Internal season field (SOU-specific)
- `key` - Musical key (C, D, etc.)
- `mode` - Major/Minor
- `sourcePreference` - Preferred API source (Auto, SP, MBZ, LFM, SC, DZ)

### UI Changes
- **Removed**: Tab-based interface (Artist/Charts/Advanced)
- **Added**: Single unified form with all fields visible
- **Added**: Source preference dropdown
- **Added**: Source badges in results (color-coded SP/MBZ/LFM/SC/DZ/WP/WD)

### API Integration
- Builds `filters` object with only non-empty fields
- POSTs to `/api/search/advanced`
- Validates at least one search criterion before submitting

## Backend Changes

### New File: `materials-server/advancedSearchHelpers.js`

#### `parseChartPosition(text)`
Fuzzy chart position parser supporting:
- **"No.1"** / **"#1"** / **"number 1"** → `{min:1, max:1}`
- **"Top 10"** / **"Top-40"** → `{min:1, max:N}`
- **"5"** (plain numbers) → `{min:5, max:5}`
- **"invalid"** → `null`

#### `applyFilters(songs, filters)`
Post-fetch filtering for:
- Year range (`yearStart`, `yearEnd`)
- Genre (partial case-insensitive match)
- Tag (searches title/artist/album)
- Chart position range (requires `chartPeak` field)
- Songwriter (partial match, requires enrichment field)
- Season (exact match, SOU internal)
- Key (exact match from audio features)
- Mode (major/minor from audio features)

#### `detectSource(song)`
Determines source badge based on ID fields present:
- `spotifyId` → SP
- `musicbrainzId` → MBZ
- `lastfmUrl` → LFM
- `soundchartsId` → SC
- `deezerId` → DZ
- `wikipediaUrl` → WP
- `wikidataId` → WD

### Updated: `materials-server/searchService.js`

#### New Method: `advancedSearch(criteria)`
Unified search supporting all criteria:

**Search Strategy Priority:**
1. **If artist specified** → `searchByArtist()` + filter results
2. **If year/chart specified** → `searchByYearAndCharts()` + filter
3. **If tag/genre specified** → Spotify/Last.fm tag search + filter

**Features:**
- Fuzzy chart position parsing via `parseChartPosition()`
- Post-fetch filtering via `applyFilters()`
- Source preference routing (prioritize specific API)
- Returns up to `limit` results after filtering

#### Updated Format Methods
Added `source` field to all formatters:
- `formatSpotifyTrack()` → `source: 'SP'`
- `formatMBZRecording()` → `source: 'MBZ'`
- `formatLastfmTrack()` → `source: 'LFM'`

### Updated: `materials-server/server.js`

#### `/api/search/advanced` Endpoint
- Detects new fields vs legacy query format
- Routes to `advancedSearch()` if new fields present
- Falls back to `searchWithFilters()` for backward compatibility
- Returns `{ success, songs, count, criteria }`

## CSS Styling (`sou-song-browser/src/components/SongSearch.css`)

### Source Badge Colors
```css
.source-badge.source-sp { background: #1db954; } /* Spotify green */
.source-badge.source-mbz { background: #ba478f; } /* MusicBrainz purple */
.source-badge.source-lfm { background: #d51007; } /* Last.fm red */
.source-badge.source-sc { background: #ff6b35; } /* Soundcharts orange */
.source-badge.source-dz { background: #00c7f2; } /* Deezer cyan */
.source-badge.source-wp { background: #000; } /* Wikipedia black */
.source-badge.source-wd { background: #006699; } /* Wikidata blue */
```

### Unified Search Form
- Select inputs styled with padding, borders, matching existing inputs
- All fields in `.unified-search` container with consistent spacing

## Testing Scenarios

### Chart Position Fuzzy Parsing
- "No.1" → Returns #1 hits only
- "Top 10" → Returns positions 1-10
- "#5" → Returns exact #5
- "Top 40" → Returns positions 1-40

### Multi-Criteria Combinations
1. **Artist + Year**: "Beatles" + 1965-1970
2. **Genre + Chart**: "rock" + "Top 40"
3. **Tag + Year**: "acoustic" + 2010-2020
4. **Songwriter**: "Paul McCartney"
5. **Key + Mode**: "C" + "Major"

### Source Preference Routing
- **Auto**: Try sources in fallback order (SP → MBZ → LFM)
- **SP**: Prioritize Spotify
- **MBZ**: Prioritize MusicBrainz
- **LFM**: Prioritize Last.fm (requires API key)

## Known Limitations

### Field Dependencies on Enrichment
Some filters require enriched data fields:
- **chartPeak**: Requires Wikipedia/Soundcharts enrichment
- **songwriter**: Requires MusicBrainz/Genius enrichment
- **key/mode**: Requires Spotify audio features enrichment

### Tag Search Approximation
Tag filter currently searches title/artist/album text. For proper tag search:
- Last.fm provides actual tags
- Spotify genre metadata is more coarse

### Chart Position Field
`chartPeak` must be populated during enrichment from:
- Wikipedia chart parsing
- Soundcharts API data
- Manual CSV entries

## Future Enhancements

### Priority 1: Enrichment Improvements
- Ensure all songs have `chartPeak` populated
- Add songwriter credits to import flow
- Bulk fetch audio features for key/mode

### Priority 2: UI Polish
- Add field help text ("e.g., Top 10, No.1")
- Show active filters as removable chips
- Display source stats (X from Spotify, Y from MusicBrainz)

### Priority 3: Advanced Features
- Saved search presets
- Export search results to CSV
- Duplicate detection integration (pre-import warning)

## Backward Compatibility

The `/api/search/advanced` endpoint maintains backward compatibility:
- Old queries with `{query, genre, yearStart, yearEnd}` → routes to `searchWithFilters()`
- New queries with artist/tag/chartPosition/etc → routes to `advancedSearch()`

No breaking changes to existing admin features.

## Files Modified
1. `sou-song-browser/src/components/SongSearch.js` - Unified form UI
2. `sou-song-browser/src/components/SongSearch.css` - Source badge styles
3. `materials-server/advancedSearchHelpers.js` - NEW utility functions
4. `materials-server/searchService.js` - advancedSearch method + source badges
5. `materials-server/server.js` - Updated /api/search/advanced endpoint

---

**Status**: Implementation complete, ready for testing
**Next Steps**: Start backend server, test various search combinations, verify source badges
