# Enrichment Service Refactoring Plan

**Date:** 30 November 2025  
**Backup Location:** `/Users/matthew/Documents/SOU App/materials-server/archive/enrichment-refactor-backup-20251130/`

## Current State Analysis

### Existing Service Modules (Already Working)
1. ✅ `spotifyService.js` - Spotify track search + audio features
2. ✅ `lastfmService.js` - Last.fm track info + tags
3. ✅ `enrichmentService.js` - **Currently NOT using above services** ❌

### Problem
The `enrichmentService.js` is making raw fetch() calls instead of delegating to the proper service modules. It also only populates `_Best` columns instead of source-specific columns.

## APIs Available (All Configured in .env)

| API | Service Module | Status | Purpose |
|-----|---------------|--------|---------|
| Spotify | ✅ spotifyService.js | Ready | Track info, audio features (BPM, key, mode), popularity |
| Last.fm | ✅ lastfmService.js | Ready | Track tags/genres, playcount, listeners |
| MusicBrainz | ❌ Need to create | Ready | Genres, metadata (no auth needed) |
| YouTube | ❌ Need to create | Ready | Video ID for player |
| GetSongBPM | ❌ Need to create | Ready | BPM + key (fallback) |
| Deezer | ❌ Need to create | Ready | BPM, genres (public API) |

## Refactoring Steps

### Step 1: Create Missing Service Modules
Create these new service files following the pattern of existing services:

#### A. `musicbrainzService.js`
```javascript
/**
 * MusicBrainz API integration for metadata and genre data
 * No auth required, uses user agent from config
 */
- getRecordingInfo(title, artist) -> mbid, genres, year, etc.
- searchRecording(title, artist) -> array of matches
- Uses retry logic and rate limiting (1 req/sec)
```

#### B. `youtubeService.js`
```javascript
/**
 * YouTube Data API v3 integration for video search
 * Requires API key from .env
 */
- searchVideo(title, artist) -> videoId, title, channel, views
- Scoring logic for best match (official, vevo, etc.)
```

#### C. `getsongbpmService.js`
```javascript
/**
 * GetSongBPM API integration for BPM and key data
 * Requires API key from .env
 */
- searchTrack(title, artist) -> bpm, key, tempo
```

#### D. `deezerService.js`
```javascript
/**
 * Deezer public API for BPM and genre data
 * No auth required
 */
- searchTrack(title, artist) -> bpm, genres, duration
```

### Step 2: Refactor enrichmentService.js

#### Current Structure (BROKEN):
```javascript
enrichSong(title, artist) {
  // Makes raw fetch() calls to Spotify
  // Only populates _Best columns
  // Missing most APIs
}
```

#### New Structure (FIXED):
```javascript
async enrichSong(title, artist) {
  const enrichedData = {
    // Core fields
    title, artist,
    
    // Spotify source-specific fields
    spotifyTrackId: null,
    bpm_spotify: null,
    key_spotify: null,
    mode_spotify: null,
    timeSignature_spotify: null,
    genres_spotify: null,
    spotifyPopularity: null,
    
    // Last.fm source-specific fields  
    genres_lastfm: null,
    lastfmPlays: null,
    lastfmListeners: null,
    
    // MusicBrainz source-specific fields
    genres_musicbrainz: null,
    musicbrainzRecordingId: null,
    
    // YouTube fields
    youtubeVideoId: null,
    youtubeViews: null,
    
    // GetSongBPM fields
    bpm_getsongbpm: null,
    key_getsongbpm: null,
    
    // Deezer fields
    bpm_deezer: null,
    genres_deezer: null,
    
    // BEST fields (calculated from sources)
    bpm: null,              // BPM_Best
    originalKey: null,      // Key_Best  
    mode: null,             // Major/Minor
    timeSignature: null,    // Time_Signature_Best
    genre: null,            // Genres (Best)
    genres_provenance: null,// Genres (Best Sources)
    year: null              // Year
  };
  
  // Call ALL service modules
  const [spotify, lastfm, musicbrainz, youtube, getsongbpm, deezer] = await Promise.allSettled([
    spotifyService.searchTrack(title, artist),
    lastfmService.getTrackInfo(title, artist),
    musicbrainzService.getRecordingInfo(title, artist),
    youtubeService.searchVideo(title, artist),
    getsongbpmService.searchTrack(title, artist),
    deezerService.searchTrack(title, artist)
  ]);
  
  // Populate source-specific fields
  if (spotify.status === 'fulfilled' && spotify.value) {
    // ... populate spotify fields
  }
  
  // Calculate BEST values using priority logic
  enrichedData.bpm = this.determineBestBPM({
    spotify: enrichedData.bpm_spotify,
    deezer: enrichedData.bpm_deezer,
    getsongbpm: enrichedData.bpm_getsongbpm
  });
  
  enrichedData.genre = this.determineBestGenre({
    spotify: enrichedData.genres_spotify,
    lastfm: enrichedData.genres_lastfm,
    musicbrainz: enrichedData.genres_musicbrainz,
    deezer: enrichedData.genres_deezer
  });
  
  return enrichedData;
}
```

### Step 3: Add Field Mapper Entries

Add all source-specific fields to `fieldMapper.js`:

```javascript
JSON_TO_CSV_MAP = {
  // ... existing fields ...
  
  // Spotify source fields
  bpm_spotify: 'BPM (Spotify)',
  key_spotify: 'Key (Spotify)',
  mode_spotify: 'Mode (Spotify)',
  timeSignature_spotify: 'Time Signature (Spotify)',
  genres_spotify: 'Genres (Spotify)',
  
  // Last.fm source fields
  genres_lastfm: 'Genres (Last.fm)',
  
  // MusicBrainz source fields
  genres_musicbrainz: 'Genres (MB)',
  
  // GetSongBPM source fields
  bpm_getsongbpm: 'BPM (Getsongbpm)',
  key_getsongbpm: 'Key (Getsongbpm)',
  
  // Deezer source fields
  bpm_deezer: 'BPM (Deezer)',
  genres_deezer: 'Genres (Deezer)',
  
  // ... etc
};
```

### Step 4: Implement "Best" Determination Logic

Port Python logic from `sou_merge_bpm_best.py` and `sou_enrich_genres_best.py`:

```javascript
determineBestBPM(sources) {
  // Priority: Deezer (0.9) > Spotify (0.9) > Tunebat (0.85) > GetSongBPM (0.8)
  // Return first valid value following priority
}

determineBestGenre(sources) {
  // Combine all sources, deduplicate, sort
  // Track provenance (which sources contributed)
}

determineBestKey(sources) {
  // Similar priority logic for keys
}
```

## Implementation Order

1. ✅ Create backup (DONE)
2. ⏳ Create new service modules (musicbrainz, youtube, getsongbpm, deezer)
3. ⏳ Refactor enrichmentService.js to use services
4. ⏳ Add field mapper entries for source-specific fields
5. ⏳ Implement "Best" determination logic
6. ⏳ Test with a single song
7. ⏳ Restart backend and test promotion

## Testing Plan

**Test Song:** "Dreaming" by Blondie (currently has no enrichment data)

**Expected After Enrichment:**
- ✅ Title: "Dreaming"
- ✅ Artist: "Blondie"
- ✅ Year: 1979
- ✅ BPM_Best: ~140 (from Spotify or other source)
- ✅ Key_Best: E or F (from Spotify)
- ✅ Mode: Major/Minor
- ✅ Time_Signature_Best: 4/4
- ✅ Genres (Best): "New Wave, Pop, Rock" (merged from all sources)
- ✅ Genres (Spotify): Spotify's genres
- ✅ Genres (Last.fm): Last.fm's tags
- ✅ BPM (Spotify): Spotify's tempo value
- ✅ YouTube Video ID: Found video ID

## Risk Mitigation

- ✅ Backup created before any changes
- ⏳ Test with single song before batch operation
- ⏳ Keep existing service modules unchanged
- ⏳ Add error handling for each API call
- ⏳ Log all enrichment steps for debugging

## Questions/Decisions Needed

1. Should we also re-enrich existing songs missing data (like "Call Me" by Blondie)?
2. Rate limiting strategy - sequential or parallel API calls?
3. What to do if ALL API sources fail for a field?

---

**Ready to proceed?** This refactoring will make enrichment much more robust and complete!
