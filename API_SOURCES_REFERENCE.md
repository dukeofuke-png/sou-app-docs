# 3rd Party API & Tool Source Reference

Complete reference for all external data sources used in the SOU Song Database enrichment pipeline.

---

## 1. **Spotify (SP)**
- **Best for:** Audio previews, popularity metrics, audio features (BPM, key, energy), genre tags
- **Scripts:** `sou_enrich_spotify.py`, `sou_enrich_spotify_audio_features.py`, `sou_spotify_audio_backfill_v*.py`
- **Limitations:** No songwriter credits, no chart position, requires API credentials

## 2. **MusicBrainz (MBZ)**
- **Best for:** **Songwriter/composer/credits**, release metadata, ISRC, alternate versions
- **Scripts:** `sou_enrich_musicbrainz_ids.py`, `sou_enrich_mb_dates.py`, `sou_enrich_mb_writers.py`
- **Limitations:** No popularity or chart data, no audio previews

## 3. **Last.fm (LFM)**
- **Best for:** Genre/tags (crowdsourced), popularity (playcount, listeners), related artists
- **Scripts:** `sou_enrich_lastfm_popularity.py`, `sou_enrich_lastfm_tags.py`
- **Limitations:** No chart position, no songwriter info, no audio previews

## 4. **Soundcharts (SC)**
- **Best for:** **Chart positions** (Billboard, UK, global), historical chart data, social metrics
- **Scripts:** `sou_enrich_soundcharts.py`, `sou_enrich_soundcharts_charts.py`, `sou_enrich_soundcharts_master.py`
- **Limitations:** No audio previews, no songwriter info, requires paid credentials

## 5. **Wikipedia (WP)**
- **Best for:** **Chart positions** (scraped), release dates, song context, critical reception, **songwriter credits** (text parsing)
- **Scripts:** `sou_enrich_wikipedia_urls.py`, `sou_enrich_wikipedia_content.py`, `sou_enrich_wikipedia_charts.py`
- **Usage:** Heavily used for chart/date backfill
- **Limitations:** Scraping-based (fragile), no audio, inconsistent formatting

## 6. **Wikidata (WD)**
- **Best for:** **Release dates** (P577), **chart positions** (P2291), **songwriter credits** (P676, P86), publisher info, cover art
- **Scripts:** `sou_enrich_wikidata_dates.py`, `sou_wikidata_backfill.py`, `sou_resolve_wikidata_qids.py`
- **Usage:** Significantly used for dates, charts, credits when MusicBrainz insufficient
- **Limitations:** Coverage varies, no audio, no popularity metrics

## 7. **Discogs**
- **Best for:** Release metadata (master/release IDs), label info, format details, genre/style tags, cover art
- **Scripts:** `sou_discogs_enrich_v1.py`, `sou_fix_discogs_columns.py`
- **Limitations:** No chart data, no songwriter credits, no audio, requires API token

## 8. **Deezer**
- **Best for:** **BPM/tempo** (primary source), genre tags, preview URLs
- **Scripts:** `sou_enrich_deezer.py`, `sou_enrich_deezer_genres.py`, `sou_merge_bpm_best.py`
- **Usage:** Primary BPM source alongside GetSongBPM/TuneBat
- **Limitations:** No songwriter credits, no chart positions

## 9. **Genius**
- **Best for:** **Lyrics**, song annotations, songwriter credits (sometimes)
- **Scripts:** `sou_enrich_genius_full.py`, `sou_enrich_songwriters_genius.py`
- **Limitations:** No chart/popularity data, no audio previews

## 10. **YouTube**
- **Best for:** Video IDs for embedding, view counts (popularity proxy)
- **Scripts:** `sou_enrich_youtube.py`
- **Limitations:** No structured metadata, no songwriter credits, no chart data

## 11. **GetSongBPM / TuneBat / Cyanite**
- **Best for:** **BPM/tempo measurements**, key signatures, time signatures
- **Scripts:** `sou_enrich_getsongbpm.py`, `sou_enrich_tunebat.py`, `sou_enrich_cyanite.py`
- **Usage:** Used alongside Deezer for BPM consolidation
- **Limitations:** Single-purpose (tempo/key only)

## 12. **Librosa**
- **Type:** Python audio analysis library (local MP3 analysis, not an API)
- **Best for:**
  - **Comprehensive audio feature extraction from MP3 files**
  - BPM/tempo detection (beat tracking)
  - Key & mode detection (chroma features)
  - Time signature estimation
  - Spectral features (timbre, brightness, rolloff)
  - Rhythm patterns (onset detection)
  - Harmonic/percussive separation
  - Duration, sample rate, channels
  - **No internet connection required**
- **Use Cases:**
  - Backfill missing BPM/key data from local MP3 collection
  - Validate/cross-reference API-provided tempo/key
  - Generate audio fingerprints for duplicate detection
  - Admin drag-and-drop MP3 analysis tool
- **Scripts:** (from early iterations; not currently active in pipeline)
- **Future Admin Feature:**
  - Drag-and-drop MP3 upload interface
  - Real-time analysis display (BPM, key, duration, spectral preview)
  - Batch processing for local MP3 library
  - Write results to measurements table or enrichment CSV
- **Limitations:**
  - Requires local MP3 files (cannot query remote databases)
  - Python dependency (not Node.js; requires separate service or subprocess)
  - Computationally intensive for large batches
  - No metadata like songwriter, genre, or popularity

---

## Multi-Source Chart Aggregation Pipeline

The materials server implements a sophisticated **multi-source chart aggregation** system that combines authoritative chart data from multiple sources with intelligent fallback logic.

### Aggregation Order (Priority Sequence)

When a chart lookup is requested via `/api/search/charts/aggregate`, the system queries sources in this order:

1. **Soundcharts** (primary authoritative source)
   - Real-time API for Billboard, UK Singles, Spotify Global, Apple Music charts
   - Provides peak positions, chart names, country codes, historical data
   - Requires API credentials (`SOUNDCHARTS_APP_ID`, `SOUNDCHARTS_API_KEY`)

2. **Wikidata** (structured knowledge base)
   - SPARQL queries for P1352 (ranking), P2291 (charted in), P585 (point in time)
   - Two-phase strategy: Entity search for QID → Chart statement retrieval
   - Returns chart name, peak position, date

3. **Wikipedia** (scraped chart tables)
   - HTML parsing of song pages targeting "Weekly charts" / "Charts" sections
   - Normalizes footnote markers, excludes certification tables
   - Extracts chart name and peak from table cells

4. **Internal Dataset** (songs_app_export_merged.json)
   - Fallback to enriched CSV export with `chartPeak` field
   - Pre-populated from historical enrichment pipeline
   - Used when external sources lack data

5. **Popularity Approximation** (last resort heuristic)
   - When NO authoritative chart data exists across all sources
   - Uses Spotify `popularity` score or Last.fm `playcount` as proxy
   - Returned with source `popularity-fallback` and `chartName: 'approx-popularity'`
   - **Does not represent actual chart performance**

### Fallback Logic

- **Cascading:** Each source is tried sequentially; aggregation stops if sufficient data found
- **Deduplication:** Charts are deduplicated by `chartName|chartPeak|source` key
- **Similarity Collapse:** Near-duplicate chart names (e.g., "US Billboard Hot 100" vs "Billboard Hot 100") with same peak are merged
- **Caching:** Results cached for 1-2 hours using LRU cache to reduce external API calls

### Response Metadata

All chart aggregation responses include provenance metadata:

```json
{
  "charts": [ /* array of chart position objects */ ],
  "overallPeak": 1,
  "sourcesUsed": ["wikipedia", "internal"],
  "title": "Video Killed The Radio Star",
  "artist": "Buggles",
  "year": 1979
}
```

### Interpreting `sourcesUsed`

- **`soundcharts`**: Real-time authoritative chart data
- **`wikidata`**: Structured statements from knowledge graph
- **`wikipedia`**: Scraped from chart tables (historical)
- **`internal`**: Pre-enriched CSV data (offline pipeline)
- **`popularity-fallback`**: ⚠️ **Approximation only** – not actual chart position

### When Popularity Fallback Occurs

The popularity fallback is **only invoked** when:
1. No charts found in Soundcharts, Wikidata, Wikipedia, or internal dataset
2. Artist/title search returns results from Spotify/Last.fm/MusicBrainz
3. System needs to return *something* to indicate song exists but lacks chart history

**Important:** Songs with `popularity-fallback` source should be marked with a warning in the UI (e.g., "⚠️ Approximate – no chart data available").

### Configuration

Set environment variables to enable external sources:
```bash
# Soundcharts (required for primary chart source)
SOUNDCHARTS_APP_ID=your_app_id
SOUNDCHARTS_API_KEY=your_api_key

# Internal dataset path
SONGS_EXPORT_PATH=/path/to/songs_app_export_merged.json
```

### API Endpoints

- **`GET /api/search/charts/aggregate`**
  - Query: `?title=Song&artist=Artist&year=1980&limit=50`
  - Returns: Aggregated charts with provenance
  - Auth: Required

- **`GET /api/search/charts/years`**
  - Returns: List of available years from internal dataset
  - Used for validation and autocomplete
  - Auth: Required

### Testing

Run test suite:
```bash
cd materials-server
node chartService.test.js
```

Tests validate:
- Internal fallback behavior
- Wikipedia + Internal merge
- Wikidata peak override
- Popularity fallback invocation

---

## Source Routing for Admin Search Criteria

| Search Criterion    | Primary Source(s)                  | Fallback(s)              |
|---------------------|-----------------------------------|--------------------------|
| **Artist**          | Spotify → MusicBrainz → Last.fm   | Discogs, Deezer          |
| **Tag/Query**       | Last.fm → Spotify                 | Genius (lyrics search)   |
| **Genre**           | Spotify, Last.fm, Soundcharts     | MusicBrainz, Discogs, Deezer |
| **Year Range**      | Spotify, MusicBrainz              | Wikipedia, Wikidata, Discogs |
| **Chart Position**  | **Soundcharts** → Wikipedia → Wikidata | (none)              |
| **Songwriter**      | **MusicBrainz** → Wikidata        | Wikipedia (text parse), Genius |
| **Key**             | Spotify (audio features)          | TuneBat, GetSongBPM, Deezer, **Librosa** |
| **Major/Minor**     | Spotify (audio features: mode)    | **Librosa** (chroma analysis) |
| **BPM**             | **Deezer** → Spotify              | TuneBat, GetSongBPM, Cyanite, **Librosa** |

---

## Summary Table

| Source       | Audio | Popularity | Chart | Songwriter | Genre | Lyrics | BPM/Key | Release Date | Cover Art | Social | Local Files |
|--------------|:-----:|:----------:|:-----:|:----------:|:-----:|:------:|:-------:|:------------:|:---------:|:------:|:-----------:|
| Spotify      |  ✔️   |     ✔️     |  ❌   |     ❌     |  ✔️   |   ❌   |   ✔️    |      ✔️      |    ✔️     |   ❌   |     ❌      |
| MusicBrainz  |  ❌   |     ❌     |  ❌   |   **✔️**   |  ✔️   |   ❌   |   ❌    |      ✔️      |    ❌     |   ❌   |     ❌      |
| Last.fm      |  ❌   |     ✔️     |  ❌   |     ❌     |  ✔️   |   ❌   |   ❌    |      ❌      |    ❌     |   ❌   |     ❌      |
| Soundcharts  |  ❌   |     ❌     | **✔️**|     ❌     |  ✔️   |   ❌   |   ❌    |      ❌      |    ❌     |  ✔️    |     ❌      |
| Wikipedia    |  ❌   |     ❌     | **✔️**|     ✔️     |  ❌   |   ❌   |   ❌    |    **✔️**    |    ❌     |   ❌   |     ❌      |
| Wikidata     |  ❌   |     ❌     | **✔️**|   **✔️**   |  ❌   |   ❌   |   ❌    |    **✔️**    |    ✔️     |   ❌   |     ❌      |
| Discogs      |  ❌   |     ❌     |  ❌   |     ❌     |  ✔️   |   ❌   |   ❌    |      ✔️      |    ✔️     |   ❌   |     ❌      |
| Deezer       |  ✔️   |     ❌     |  ❌   |     ❌     |  ✔️   |   ❌   | **✔️**  |      ✔️      |    ❌     |   ❌   |     ❌      |
| Genius       |  ❌   |     ❌     |  ❌   |     ✔️     |  ❌   | **✔️** |   ❌    |      ❌      |    ❌     |   ❌   |     ❌      |
| YouTube      |  ✔️   |     ✔️     |  ❌   |     ❌     |  ❌   |   ❌   |   ❌    |      ❌      |    ✔️     |   ❌   |     ❌      |
| BPM Services |  ❌   |     ❌     |  ❌   |     ❌     |  ❌   |   ❌   | **✔️**  |      ❌      |    ❌     |   ❌   |     ❌      |
| **Librosa**  |  ❌   |     ❌     |  ❌   |     ❌     |  ❌   |   ❌   | **✔️**  |      ❌      |    ❌     |   ❌   |   **✔️**    |

---

## Librosa Integration Roadmap

### Phase 1: Backend Service
- Create `/api/analyze/mp3` endpoint (multipart/form-data upload)
- Python subprocess or microservice to run Librosa analysis
- Return JSON: `{ bpm, key, mode, duration, spectral_features, confidence }`

### Phase 2: Admin UI
- Drag-and-drop upload zone in admin dashboard
- Real-time analysis progress indicator
- Display results table with option to merge into database
- Batch upload support (analyze entire folder)

### Phase 3: Backfill Pipeline
- Script to process local MP3 directory
- Write results to `measurements.csv` with source='librosa'
- Consolidation with existing BPM/key sources (Deezer, Spotify, TuneBat)
