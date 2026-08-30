# API Enrichment Strategy Reference

**Last Updated**: 23 November 2025

This document defines which APIs to use for each data field, in priority order, for both databases.

---

## Database-Specific Strategies

### Seed/Admin Database (`expanded_seed_base.json`)
- **Focus**: Speed and coverage
- **Volume**: 38,437 songs (growing)
- **Rate limits**: Critical constraint
- **Quality**: Good enough for discovery

### SOU/Tutor Database (`songdb_master_v2_enriched.csv`)
- **Focus**: Accuracy and depth
- **Volume**: 212 songs (curated)
- **Rate limits**: Less critical
- **Quality**: Production-grade

---

## API Priority by Data Field

### 🎵 **GENRES**

#### For **Seed Database** (38,437 songs, 6,945 unique artists)
**Strategy**: Artist-level lookup (efficient for bulk enrichment)

1. **Last.fm** - `artist.getinfo` → `tags` array (top 5)
   - ✅ Best coverage for diverse/indie artists
   - ✅ Better rate limits (200ms delay sufficient)
   - ✅ Artist-level tags very reliable
   - ✅ Free API, no OAuth needed
   - ⚠️ Sometimes returns music-related tags (e.g., "seen live", "favorites")

2. **Spotify** - `/v1/artists/{id}` → `genres` array
   - ✅ Official genres, well-curated
   - ✅ Excellent for mainstream artists
   - ❌ Aggressive rate limiting (429 errors even at 300ms)
   - ❌ Many artists have empty genre arrays
   - ⚠️ Requires OAuth token refresh

3. **MusicBrainz** - `/ws/2/artist/{id}` → tags (fallback)
   - ✅ Community-curated, reliable
   - ✅ Good for classical, jazz, world music
   - ❌ Very slow rate limit (1 request/second)
   - ❌ Not practical for 6,945 artists (~2 hours minimum)

**Recommended Order for Seed Database**: Last.fm → Spotify fallback → MusicBrainz (manual for gaps)

#### For **SOU Database** (212 songs)
**Strategy**: Multi-source with provenance tracking

1. **Last.fm** - Track + Artist tags
   - Track: `track.getinfo` → `toptags`
   - Artist: `artist.getinfo` → `tags`
   - Merge and dedupe

2. **MusicBrainz** - Artist tags
   - `/ws/2/artist/{mbid}` → tags
   - Good for genre depth

3. **Deezer** - Artist genres
   - `/artist/{id}` → `genres`
   - Currently low coverage (0/212)

4. **Manual** - Original SOU Genre field
   - Fallback if APIs have no data

**Merge into**: `Genres (Best)` with source tracking in `Genres (Best Sources)`

---

### 📅 **RELEASE DATE / YEAR**

#### For **Seed Database**
**Strategy**: Single source (speed priority)

1. **Spotify** - `/v1/tracks/{id}` → `album.release_date`
   - ✅ Best coverage (99.9% of commercial music)
   - ✅ Accurate down to day precision
   - ✅ Already have track IDs from playlist import
   - ⚠️ Rate limiting (use with caution)

2. **MusicBrainz** - `/ws/2/recording/{id}` → release dates
   - ✅ Very accurate
   - ❌ Slow (1 req/sec)
   - ❌ Requires MBID lookup first

**Recommended**: Spotify only (fast, sufficient accuracy)

#### For **SOU Database**
**Strategy**: Multi-source verification

1. **Spotify** - Primary
2. **MusicBrainz** - Verification for classical/obscure
3. **Manual** - From original CSV `Date` field

**Current Status**: 99.8% coverage via Spotify enrichment scripts

---

### 📊 **POPULARITY / ENGAGEMENT**

#### For **Seed Database**
**Strategy**: Simple single-source metric

1. **Spotify** - `/v1/tracks/{id}` → `popularity` (0-100)
   - ✅ Real-time metric
   - ✅ Already in track object (no extra call)
   - ✅ Good proxy for mainstream appeal

2. **Last.fm** - `track.getinfo` → `playcount` + `listeners`
   - ✅ Better for indie/niche artists
   - ❌ Extra API call required
   - ⚠️ Numbers can be misleading (bots, old data)

**Recommended**: Spotify popularity only (efficient)

#### For **SOU Database**
**Strategy**: Multi-metric aggregate

1. **Spotify** - `popularity` (0-100)
2. **Last.fm** - `playcount` and `listeners`
3. **YouTube** - View counts (from `sou_enrich_youtube.py`)

**Aggregate into**: Popularity tier (High/Medium/Emerging) based on thresholds:
```python
High:     plays >= 50,000 OR listeners >= 10,000 OR spotify >= 70
Medium:   plays >= 10,000 OR listeners >= 3,000  OR spotify >= 50
Emerging: any non-zero plays/listeners OR spotify > 0
```

---

### 🎹 **BPM / TEMPO**

#### For Both Databases
**Strategy**: Multiple sources with "Best" field

1. **Getsongbpm.com** - Web scraping
   - ✅ Community-verified BPMs
   - ✅ Often most accurate
   - ⚠️ Manual/slow (not API-based)

2. **Tunebat** - Web scraping
   - ✅ Good coverage
   - ⚠️ Manual/slow

3. **Spotify** - `/v1/audio-features/{id}` → `tempo`
   - ✅ API-based (fast)
   - ⚠️ Can be off by 2x or halved
   - ⚠️ Sometimes weird values

4. **Manual** - Original BPM field

**Merge into**: `BPM_Best` (most trusted value, with source tracking)

**Current Scripts**:
- `tools/sou_enrich_getsongbpm.py`
- `tools/sou_enrich_tunebat.py`
- `tools/sou_enrich_spotify_features.py`

---

### 🎤 **ARTIST INFO**

#### For **Seed Database**
**Strategy**: Minimal (just name and ID)

- Store: `artist` (name), Spotify artist ID (if available)
- No biography, images, or extra data

#### For **SOU Database**
**Strategy**: Rich metadata

1. **Spotify** - Bio, images, popularity
2. **Last.fm** - Bio, tags, similar artists
3. **MusicBrainz** - Canonical data, relationships

---

### 🔗 **EXTERNAL LINKS**

#### For **Seed Database**
**Strategy**: Store IDs only (reconstruct URLs as needed)

- `spotifyId` → `https://open.spotify.com/track/{id}`
- Generate URLs on-demand in frontend
- No YouTube or PDF links (not applicable for discovery database)

#### For **SOU Database**
**Strategy**: Full URL storage + metadata from multiple sources

1. **YouTube** - `sou_enrich_youtube.py` (API)
   - Source: YouTube Data API v3
   - Search: `"{title}" {artist}`
   - Store: URL, video ID, view count, upload date, channel name
   - Columns: `YouTube Link`, `YouTube Video ID`, `YouTube View Count`, `YouTube Upload Date`, `YouTube Retrieved (UTC)`
   - Priority: N/A (API only)

2. **Spotify** - Track URLs (API)
   - Source: Spotify Web API
   - Store: Full track URL in `Spotify URL` column
   - Format: `https://open.spotify.com/track/{spotifyId}`
   - Priority: N/A (API only)

3. **PDF** - Google Drive links (Primary source from SOU archive)
   - Source: Manual SOU Database (`PDF_ID` field)
   - Contains: Google Drive file IDs for sheet music, TAB, and music theory PDFs
   - Store: Drive file ID in `PDF_ID` column
   - Proxy: Materials-server endpoint `/pdf/{id}` serves PDFs with proper headers
   - Priority: **Primary** (from actual SOU song, tab, and music theory archive)
   - Note: This is teaching material, not API-sourced data

**Key Distinction**: PDF links come from SOU's internal archive of teaching materials, not from external APIs. YouTube and Spotify links are discovered via APIs for student engagement.

---

## Rate Limit Settings

### Current Configuration (`materials-server/config.js`)

```javascript
rateLimits: {
  spotify: 300,  // ms - recently increased from 100 due to 429 errors
  lastfm: 200    // ms - reliable, rarely hits limits
}
```

### Recommendations

| API | Seed Database | SOU Database | Notes |
|-----|---------------|--------------|-------|
| **Spotify** | 300-500ms | 100-200ms | 429 errors common in bulk |
| **Last.fm** | 200ms | 200ms | Very stable |
| **MusicBrainz** | 1000ms | 1000ms | Enforced server-side |
| **YouTube** | 1000ms | 1000ms | Quota-based (10,000/day) |

---

## Field Priority Matrix

| Field | Seed Database Priority | SOU Database Priority |
|-------|------------------------|----------------------|
| **Genres** | 1. Last.fm<br>2. Spotify | 1. Last.fm<br>2. MusicBrainz<br>3. Deezer<br>4. Manual |
| **Release Date** | 1. Spotify | 1. Spotify<br>2. MusicBrainz<br>3. Manual |
| **Popularity** | 1. Spotify | 1. Multi-metric aggregate |
| **BPM** | Not enriched | 1. Getsongbpm<br>2. Tunebat<br>3. Spotify<br>4. Manual |
| **Artist Info** | Name + ID only | 1. Spotify<br>2. Last.fm<br>3. MusicBrainz |
| **External Links** | IDs only | Full URLs + metadata |

---

## Current Issues & Solutions

### Issue: Spotify Rate Limiting (429 errors)
**Problem**: Even with 300ms delays, bulk enrichment fails  
**Solution**: 
1. Switch to Last.fm for genres (better rate limits)
2. Use Spotify only for tracks with existing IDs (no search needed)
3. Consider batch API endpoints where available

### Issue: Last.fm Tag Quality
**Problem**: Sometimes returns non-genre tags ("seen live", "favorites")  
**Solution**: 
1. Filter common non-genre tags
2. Use artist-level tags (more reliable than track-level)
3. Cross-reference with other sources in SOU database

### Issue: MusicBrainz Speed
**Problem**: 1 req/sec = ~2 hours for 6,945 artists  
**Solution**: 
1. Use only for SOU database (212 songs manageable)
2. Background processing with checkpoint system
3. Cache results aggressively

---

## Script Mapping

### Seed Database Scripts
- `materials-server/enrichGenres.js` - Genre enrichment (Last.fm + Spotify)
- `materials-server/importPlaylistsToSeed.js` - Import with metadata
- `materials-server/expandCatalogAuto.js` - Auto expansion
- `materials-server/expandCatalogDual.js` - Dual API expansion

### SOU Database Scripts  
- `Song Database/tools/sou_enrich_lastfm_tags.py` - Last.fm enrichment
- `Song Database/tools/sou_enrich_deezer.py` - Deezer genres
- `Song Database/tools/sou_enrich_mb_lastfm.py` - MusicBrainz + Last.fm
- `Song Database/tools/sou_enrich_spotify_dates.py` - Release dates
- `Song Database/tools/sou_enrich_getsongbpm.py` - BPM from getsongbpm.com
- `Song Database/tools/sou_enrich_youtube.py` - YouTube links + metadata
- `Song Database/tools/sou_consolidate_enrichment.py` - Merge all sources

---

## Next Steps

1. ✅ Switch seed database genre enrichment to Last.fm primary
2. ⏳ Test Last.fm enrichment on 6,945 artists (~30 minutes)
3. ⏳ Achieve >95% genre coverage for seed database
4. ⏳ Document learnings for SOU database enrichment improvements
