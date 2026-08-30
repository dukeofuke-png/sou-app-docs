# Song Search Engine Feature

## Overview
The song search engine allows admins to query multiple external music databases to discover and import songs in bulk. The system uses a **multi-source fallback strategy** for reliability:

1. **Spotify** (rich metadata, audio features) → primary when available
2. **MusicBrainz** (comprehensive coverage, open data) → fallback for artist/recording lookup
3. **Last.fm** (popularity signals, genre tags) → fallback for charts/trends
4. **Soundcharts** (chart positions) → optional for historical chart data

This is ideal for scenarios like:
- Finding all songs by a specific artist (e.g., "Blondie" returns 50+ studio tracks)
- Discovering top hits from a year range (e.g., "Top 10 hits from 1995-2000")
- Advanced searches with genre and year filters
- Natural language queries via AI assistant (e.g., "20 disco hits from late 70s")

## How It Works

### Architecture
```
Frontend (SongSearch.js + AdminAIHelper.js)
    ↓ Search request (manual or AI-generated)
Backend (searchService.js + aiAssistant.js)
    ↓ Try primary source (Spotify)
    ↓ Fallback to MusicBrainz (if Spotify fails)
    ↓ Fallback to Last.fm (if both unavailable)
External Music Databases
    ↓ Song metadata results (with source badges)
Backend formats & returns
    ↓ Display results with source indicators
Frontend shows preview with audio (if available)
    ↓ Admin selects songs
Enrichment API adds full metadata
    ↓ Import to CSV
Song added to database
```

### Multi-Source Fallback Logic

**Artist Search:**
1. Try Spotify artist search + top tracks
2. If fails → Query MusicBrainz for artist MBID + recordings
3. Filter MusicBrainz results (exclude live, intro, <30s tracks)
4. If MusicBrainz empty → Try Last.fm artist top tracks
5. Return results with source badge (SP/MBZ/LFM)

**Year/Chart Search:**
1. Try Soundcharts API (if credentials present)
2. Fallback to Spotify year-filtered search sorted by popularity
3. If Spotify fails → Use MusicBrainz recording search by year
4. If both fail → Return empty array (graceful, no hard error)

**Advanced Search:**
1. Spotify multi-filter query (genre + year + keyword)
2. Fallback to MusicBrainz tag + year search
3. Last.fm tag-based search as final fallback

### Search Types

#### 0. AI Assistant (Natural Language)
**Use Case:** "Get me 20 disco hits from 1975 to 1979"

**How it works:**
1. Admin types natural language prompt in AI Helper panel
2. Backend sends to OpenAI/Anthropic (or rule-based parser if no key)
3. LLM extracts intent + parameters (topN, yearRange, artist, genre)
4. Returns structured suggestions (chartSearch, artistSearch, advancedSearch)
5. Admin clicks suggestion to execute search or pre-populate search modal

**AI Endpoint:** `POST /api/ai/query`

**Request:**
```json
{
  "prompt": "I need 20 upbeat disco tracks from the late 70s for a party playlist"
}
```

**Response:**
```json
{
  "success": true,
  "intent": "genre-period",
  "suggestions": [
    {
      "type": "advancedSearch",
      "description": "Search for upbeat disco tracks from the late 70s.",
      "params": {
        "topN": 20,
        "yearStart": 1975,
        "yearEnd": 1979,
        "genre": "disco"
      }
    }
  ],
  "parsed": {
    "topN": 20,
    "yearStart": 1975,
    "yearEnd": 1979,
    "genre": "disco"
  }
}
```

**Fallback:** If no API keys configured, uses deterministic regex-based parser (still functional, less flexible).

#### 1. By Artist
**Use Case:** "I want all Blondie songs"

**How it works:**
1. **Primary (Spotify):** Searches artist + fetches top tracks + album tracks (if token valid)
2. **Fallback (MusicBrainz):** 
   - Searches artist name (strict match)
   - If no match, tries loose match (e.g., "the beatles" → "Beatles")
   - Fetches recordings, filters out live/intro/short (<30s) tracks
   - Implements retry logic (3 attempts with 1s delay for network issues)
3. **Final Fallback (Last.fm):** Queries artist top tracks if LASTFM_API_KEY present
4. Deduplicates by title + returns up to limit (default 50)

**API Endpoint:** `GET /api/search/artist/:artistName?limit=50`

**Example:**
```bash
curl "http://localhost:3002/api/search/artist/Blondie?limit=50" \
  --cookie "sessionId=..."
```

**Response (Multi-Source):**
```json
{
  "success": true,
  "artist": "Blondie",
  "count": 47,
  "songs": [
    {
      "title": "Heart of Glass",
      "artist": "Blondie",
      "album": "Parallel Lines",
      "releaseYear": 1978,
      "popularity": 78,
      "duration": 267,
      "source": "SP",
      "spotifyId": "...",
      "previewUrl": "https://...",
      "externalUrl": "https://open.spotify.com/track/..."
    },
    {
      "title": "Call Me",
      "artist": "Blondie",
      "album": "American Gigolo OST",
      "releaseYear": 1980,
      "source": "MBZ",
      "mbid": "...",
      "externalUrl": "https://musicbrainz.org/recording/..."
    }
  ]
}
```

**Source Badges:**
- `SP` = Spotify (includes audio preview, popularity score)
- `MBZ` = MusicBrainz (comprehensive, no preview)
- `LFM` = Last.fm (popularity + play count)

#### 2. By Charts/Year Range
**Use Case:** "I want top hits from 1995-2000"

**How it works:**
1. **Soundcharts Primary (if credentials):** Queries chart API for historical positions
2. **Spotify Fallback:** Year-filtered search (`year:YYYY`), popularity ≥60, top 10 per year
3. **MusicBrainz Fallback:** Recording search filtered by year, sorted by popularity proxy
4. **Graceful Empty:** Returns empty array if all sources unavailable (no hard error)

**API Endpoint:** `GET /api/search/charts/top?yearStart=1995&yearEnd=2000&limit=10`

**Example:**
```bash
curl "http://localhost:3002/api/search/charts/top?yearStart=1995&yearEnd=1996&limit=5" \
  --cookie "sessionId=..."
```

**Response:**
```json
{
  "success": true,
  "yearStart": 1995,
  "yearEnd": 1996,
  "chart": "billboard-hot-100",
  "count": 5,
  "songs": [
    {
      "title": "Gangsta's Paradise",
      "artist": "Coolio",
      "releaseYear": 1995,
      "source": "SC",
      "peak": 1,
      "chartName": "Billboard Hot 100"
    }
  ]
}
```

#### 3. Advanced Search
**Use Case:** "I want rock ballads from the 90s"

**How it works:**
1. **Spotify:** Combined query with genre/year/keyword filters
2. **MusicBrainz:** Tag + year search (e.g., tag:rock + date:1990-1999)
3. **Last.fm:** Tag-based top tracks if API key configured
4. Returns up to limit (default 50) with source badges

**API Endpoint:** `POST /api/search/advanced`

**Request Body:**
```json
{
  "query": "rock ballads",
  "genre": "rock",
  "yearStart": 1990,
  "yearEnd": 1999,
  "limit": 50
}
```

**Example:**
```bash
curl -X POST "http://localhost:3002/api/search/advanced" \
  -H "Content-Type: application/json" \
  --cookie "sessionId=..." \
  -d '{
    "query": "love songs",
    "genre": "pop",
    "yearStart": 2000,
    "yearEnd": 2010
  }'
```

### Frontend Workflow

1. **Access Search:** Click "🔍 Search Database" button in admin dashboard
2. **Choose Search Type:** Select "By Artist", "By Charts/Year", or "Advanced"
3. **Enter Criteria:**
   - Artist: Enter artist name (e.g., "Blondie")
   - Charts: Enter year range (e.g., 1995-2000)
   - Advanced: Enter query + optional genre/years
4. **View Results:** See preview list with:
   - Song title, artist, album
   - Release year, popularity %, duration
   - Audio preview player (30-second clips)
5. **Select Songs:** Click checkboxes or use "Select All"
6. **Import:** Click "Import X Selected" button
7. **Enrichment:** Backend automatically enriches with BPM, key, genre, etc.
8. **Completion:** Songs appear in main dashboard table

## Technical Details

### Data Flow for Import

```javascript
// 1. User selects songs from search results
selectedSongs = [
  { title: "Heart of Glass", artist: "Blondie", ... },
  { title: "Call Me", artist: "Blondie", ... }
]

// 2. Frontend calls enrichment API
POST /api/enrich
Body: { songs: [{ Title: "Heart of Glass", Artist: "Blondie" }, ...] }

// 3. Backend enriches each song (Spotify audio features + YouTube)
Response: {
  songs: [
    {
      Title: "Heart of Glass",
      Artist: "Blondie",
      BPM_Best: 108,
      Key_Best: "E major",
      Genre: "disco, new wave",
      releaseYear: 1978,
      ...
    }
  ]
}

// 4. Frontend imports each enriched song
for each song:
  POST /api/songs
  Body: { Title: "...", Artist: "...", BPM_Best: 108, ... }

// 5. Success notification shows import count
alert("Successfully imported 2 of 2 songs!")
```

### Rate Limiting

To avoid API throttling:
- Artist search: 100ms delay between album track fetches
- Chart search: 100ms delay between year queries
- Enrichment: 100ms delay between song enrichments (from enrichmentService.js)

### Spotify Token Caching

The search service reuses the Spotify OAuth token:
- Token cached in memory for 50 minutes (valid for 60 minutes)
- Automatic refresh when expired
- Shared with enrichmentService.js logic

### Error Handling

**No Results:**
```
"No results found. Try a different search."
```

**API Key Missing:**
```
"Spotify credentials not configured"
```
→ Add `SPOTIFY_CLIENT_ID` and `SPOTIFY_CLIENT_SECRET` to `.env`

**Search Failed:**
```
"Search failed. Please try again."
```
→ Check network, API limits, or credential validity

### Preview Audio

Spotify provides 30-second preview URLs for most tracks:
```javascript
{
  "previewUrl": "https://p.scdn.co/mp3-preview/..."
}
```

The frontend renders HTML5 `<audio>` controls for each result. If no preview available, audio player not shown.

## API Requirements

### Multi-Source Configuration

**Spotify API (Optional - Primary Source)**
1. Go to https://developer.spotify.com/dashboard
2. Create an app
3. Add to `.env`:
```bash
SPOTIFY_CLIENT_ID=your_client_id_here
SPOTIFY_CLIENT_SECRET=your_client_secret_here
```
**Status:** Optional as of Nov 2024 due to tighter restrictions. System falls back gracefully to MusicBrainz.

**MusicBrainz (No Credentials Required - Fallback)**
- Public API: https://musicbrainz.org/doc/MusicBrainz_API
- Rate limit: 1 request/second (enforced by our 1s retry delay)
- No authentication needed
- Comprehensive coverage, open data
- Our implementation includes filtering (excludes live/intro/short tracks)

**Last.fm (Optional - Secondary Fallback)**
1. Create API account: https://www.last.fm/api/account/create
2. Add to `.env`:
```bash
LASTFM_API_KEY=your_lastfm_api_key
LASTFM_SHARED_SECRET=your_shared_secret
```

**Soundcharts (Optional - Chart Data)**
1. Get credentials from Soundcharts team
2. Add to `.env`:
```bash
SOUNDCHARTS_APP_ID=your_app_id
SOUNDCHARTS_API_KEY=your_api_token
```

**OpenAI/Anthropic (Optional - AI Assistant)**
1. OpenAI: https://platform.openai.com/api-keys
2. Anthropic: https://console.anthropic.com/
3. Add to `.env`:
```bash
# Choose one or both (will try OpenAI first, then Anthropic)
OPENAI_API_KEY=sk-proj-...
OPENAI_MODEL=gpt-4o-mini
# OR
ANTHROPIC_API_KEY=key-...
ANTHROPIC_MODEL=claude-3-haiku-20240307
```

**Minimum Setup:** None required. MusicBrainz is free and needs no credentials. System fully operational without any keys.

## Use Cases & Examples

### Example 1: Building a Blondie Setlist
```
1. Click "🔍 Search Database"
2. Select "By Artist" tab
3. Enter "Blondie"
4. Click "Search"
5. Results show 47 songs
6. Preview audio for "Heart of Glass", "Call Me", "One Way or Another"
7. Select desired songs (e.g., 10 popular hits)
8. Click "Import 10 Selected"
9. Songs enriched with BPM/key and added to database
```

### Example 2: Finding 90s Hits
```
1. Click "🔍 Search Database"
2. Select "By Charts/Year" tab
3. Enter Start Year: 1990, End Year: 1999
4. Click "Search"
5. Results show ~100 popular songs from that decade
6. Sort by popularity or year
7. Select top 20 most popular
8. Click "Import 20 Selected"
```

### Example 3: Genre-Specific Search
```
1. Click "🔍 Search Database"
2. Select "Advanced" tab
3. Query: "acoustic"
4. Genre: "folk"
5. Year range: 2010-2020
6. Click "Search"
7. Results show acoustic folk songs from 2010s
8. Select desired tracks
9. Import
```

## Troubleshooting

### "Search failed"
**Likely cause:** All sources failed (network issue or rate limit hit).

**Debug:**
1. Check backend logs for detailed errors
2. Verify internet connection
3. Check MusicBrainz status: https://musicbrainz.org/doc/MusicBrainz_API
4. If Spotify-specific: validate credentials or wait if rate-limited

**Note:** System gracefully degrades—if Spotify fails, MusicBrainz takes over automatically.

### No audio previews (MusicBrainz/Last.fm results)
**This is expected.** Only Spotify provides 30-second preview URLs. MusicBrainz and Last.fm return metadata only. You can still import and enrich these songs.

### "No results found"
**Possible reasons:**
1. Artist name misspelled (try variations: "The Beatles" vs "Beatles")
2. Very obscure artist not in any database
3. Year range has no popular tracks

**Solution:** Try broader search or different source via manual API endpoint test.

### Duplicates after import
**Current behavior:** Search doesn't check existing database. Duplicate detection is planned (see Task E).

**Workaround:** Use dashboard search box to verify song doesn't exist before importing.

### AI Assistant returns generic suggestions
**Cause:** No OpenAI/Anthropic key configured, falling back to rule-based parser.

**Solution:** Add API key to `.env` (see Multi-Source Configuration above). Rule-based fallback still works for simple queries.

## Future Enhancements

### Planned Features
1. **Duplicate Detection (Task E - In Progress):** Fuzzy matching (title+artist+BPM) before import with merge options
2. **Source Badges in UI (Task B):** Visual indicators (SP/MBZ/LFM/SC) + filter by source
3. **Direct Import from AI Suggestions:** One-click bulk import from AI assistant recommendations
4. **Batch Select by Criteria:** "Select all songs with popularity >70"
5. **Additional Data Partners:**
   - Discogs (release metadata, catalog numbers)
   - AllMusic (editorial reviews, genre taxonomy)
   - AcousticBrainz (audio features without Spotify dependency)
6. **Save Search Queries:** Bookmark frequent searches
7. **Export Search Results:** Download CSV before importing
8. **Smart Recommendations:** "Songs similar to X" via collaborative filtering

### Technical Improvements
1. **Token Caching:** Persist Spotify tokens across restarts (currently memory-only)
2. **Retry Strategies:** Exponential backoff for transient failures
3. **Pagination:** Handle >100 result sets (currently capped)
4. **Confidence Scores:** Display match quality from each source
5. **Progress Indicators:** Real-time enrichment status per song
6. **Source Priority Config:** Let admin prefer MusicBrainz over Spotify (env var)

## Code References

**Backend:**
- `materials-server/searchService.js` - Multi-source search logic with fallbacks
- `materials-server/aiAssistant.js` - AI query parser (OpenAI/Anthropic + rule-based)
- `materials-server/chartService.js` - Soundcharts integration + fallback
- `materials-server/server.js` - Search & AI endpoints

**Frontend:**
- `sou-song-browser/src/components/SongSearch.js` - Search UI
- `sou-song-browser/src/components/AdminAIHelper.js` - AI assistant panel
- `sou-song-browser/src/components/SongSearch.css` - Styling
- `sou-song-browser/src/components/AdminDashboard.js` - Integration

**API Endpoints:**
- `GET /api/search/artist/:artistName?limit=50` - Artist search with fallback
- `GET /api/search/charts/top?yearStart=YYYY&yearEnd=YYYY&limit=10` - Chart/year with Soundcharts priority
- `POST /api/search/advanced` - Multi-filter search (body: {query, genre, yearStart, yearEnd, limit})
- `POST /api/ai/query` - Natural language query parser (body: {prompt})

All endpoints require authentication (session cookie).

## Summary

The search engine now provides **resilient, multi-source discovery** without reliance on any single API. Spotify restrictions are mitigated by automatic fallback to MusicBrainz (free, comprehensive) and Last.fm. The AI assistant allows non-technical admins to express complex queries in natural language, with graceful degradation to a rule-based parser when LLM keys aren't configured. This architecture future-proofs the system against vendor changes while maintaining rich metadata when available.
