# Database Terminology Reference

**Last Updated**: 23 November 2025

---

## Database Names & Purposes

### 1. **Seed Database** (aka **Admin Database**, **Expansion Database**)

**Official Name**: `expanded_seed_base.json`

**Location**: `materials-server/data/expanded_seed_base.json`

**Purpose**: 
- Large-scale song discovery and catalog expansion
- Admin/backend tool for building the song catalog
- Source for identifying potential songs to add to SOU curriculum

**Size**: 38,437 songs (as of Nov 23, 2025)

**Characteristics**:
- Continuously growing through playlist imports and API searches
- Less curated, more comprehensive
- Used by admins to discover and evaluate songs
- Feeds into the SOU database after manual curation

**Key Fields**:
```json
{
  "title": "Song Title",
  "artist": "Artist Name",
  "spotifyId": "track_id",
  "genres": ["genre1", "genre2"],
  "releaseYear": 2025,
  "popularity": { "spotify": 75 },
  "source": "artist_playlists",
  "discoveredDate": "2025-11-23"
}
```

**When to use these terms**:
- "Seed database"
- "Admin database" 
- "Expansion database"
- "Admin song expansion database"
- Files: `expanded_seed_base.json`, `artist_seed_list.csv`, `artist_playlist_links_master.csv`

---

### 2. **SOU Database** (aka **Tutor Database**, **Production Database**)

**Official Name**: `School of Uke Song Sheets Database.csv` → exported as `songs_app_export_merged.json`

**Location**: 
- CSV: `Song Database/School of Uke Song Sheets Database.csv`
- Intermediate: `Song Database/data/songdb_master_v2_enriched.csv`
- JSON Export: `sou-song-browser/src/data/songs_app_export_merged.json`

**Purpose**:
- Curated song sheets for School of Uke tutors
- Production-ready data for tutor frontend (and later student/end-user frontend)
- High-quality, manually verified songs with PDF links

**Size**: 212 songs (as of Nov 16, 2025)

**Characteristics**:
- Highly curated and manually verified
- Each song has a PDF sheet music file
- Rich metadata from multiple API sources
- Quality over quantity
- Used in production by tutors and students

**Key Fields**:
```csv
ID, Title, Artist, Genre, Key, BPM_Best, Date, ReleaseDate, 
Genres (Best), Genres (Best Sources), Last.fm Tags (Track),
Spotify Popularity, YouTube Link, PDF_ID, etc.
```

**When to use these terms**:
- "SOU database"
- "Tutor database"
- "Production database"
- "Song sheets database"
- Files: `School of Uke Song Sheets Database.csv`, `songdb_master_v2_enriched.csv`, `songs_app_export_merged.json`

---

## Quick Reference

| Term | Refers To | Size | Format | Purpose |
|------|-----------|------|--------|---------|
| **Seed Database** | `expanded_seed_base.json` | 38,437 songs | JSON | Admin catalog expansion |
| **Admin Database** | `expanded_seed_base.json` | 38,437 songs | JSON | Admin catalog expansion |
| **Expansion Database** | `expanded_seed_base.json` | 38,437 songs | JSON | Admin catalog expansion |
| **SOU Database** | `songdb_master_v2_enriched.csv` | 212 songs | CSV | Production tutor/student use |
| **Tutor Database** | `songdb_master_v2_enriched.csv` | 212 songs | CSV | Production tutor/student use |
| **Production Database** | `songdb_master_v2_enriched.csv` | 212 songs | CSV | Production tutor/student use |

---

## Data Flow

```
┌─────────────────────────────────────────┐
│   SEED/ADMIN/EXPANSION DATABASE         │
│   expanded_seed_base.json               │
│   38,437 songs (growing)                │
│   - Spotify playlists                   │
│   - Chart aggregation                   │
│   - Artist discovery                    │
└───────────────┬─────────────────────────┘
                │
                │ Manual curation
                │ Quality review
                │ PDF creation
                ↓
┌─────────────────────────────────────────┐
│   SOU/TUTOR/PRODUCTION DATABASE         │
│   songdb_master_v2_enriched.csv         │
│   212 songs (curated)                   │
│   - Rich metadata                       │
│   - PDF sheet music                     │
│   - Multi-API enrichment                │
└───────────────┬─────────────────────────┘
                │
                │ Export to JSON
                ↓
┌─────────────────────────────────────────┐
│   FRONTEND (sou-song-browser)           │
│   songs_app_export_merged.json          │
│   Used by tutors & students             │
└─────────────────────────────────────────┘
```

---

---

## 3. **Manual SOU Database** (aka **Original SOU CSV**, **Teaching Database**)

**Official Name**: `School of Uke Song Sheets Database.csv` (original, before enrichment)

**Location**: `Song Database/School of Uke Song Sheets Database.csv`

**Purpose**:
- Original curated song list with teaching-specific metadata
- Source of truth for SOU pedagogical data
- Contains fields that override or augment API data

**Size**: 212 songs (matches SOU database)

**Characteristics**:
- Manually maintained by SOU tutors
- Contains teaching-specific fields not available from APIs
- Authority for musical theory annotations (keys, chords, time signatures)
- Source for lesson planning metadata (levels, teaching notes)

**Key Fields** (Teaching-specific):
```csv
Songsheet, Major/Minor, Original Key, SOU Keys, Level, TAB,
Teaching Notes, Strum Style, Finger-picking Style, 
No. of Chords, Chords, Chord Numerals, Time Signature, BPM, Tempo,
PDF_ID (Google Drive file ID for sheet music PDFs)
```

**When to use these terms**:
- "Manual SOU database"
- "Original SOU CSV"
- "Teaching database"
- Files: `School of Uke Song Sheets Database.csv` (before any API enrichment)

---

## Data Source Priority Rules

### Field Override Hierarchy

When the SOU database is built, field values come from different sources with priority rules:

| Field | Priority | Rule | Manual SOU | APIs |
|-------|----------|------|------------|------|
| **Song Name** | N/A | API only | - | ✓ |
| **Artist** | N/A | API only | - | ✓ |
| **Year** | N/A | API only | - | ✓ |
| **Release Date** | N/A | API only | - | ✓ |
| **Season** | N/A | API only | - | ✓ |
| **Songwriter** | Augment | Merge both | ✓ | ✓ |
| **Duration** | N/A | API only | - | ✓ |
| **Popularity** | N/A | API only | - | ✓ |
| **Spotify Link** | N/A | API only | - | Spotify API |
| **YouTube Link** | N/A | API only | - | YouTube API |
| **PDF Link** | Primary | From archive | ✓ | - |
| **Lyrics** | Override | Manual first | ✓ | Genius API |
| | | | | |
| **Songsheet** | Primary | Manual only | ✓ | - |
| **Major/Minor** | Override | Manual first | ✓ | ✓ |
| **Original Key** | Override | Manual first | ✓ | ✓ |
| **SOU Keys** | Primary | Manual only | ✓ | - |
| **Level** | Primary | Manual only | ✓ | - |
| **TAB** | Primary | Manual only | ✓ | - |
| **Genre** | Augment | Merge both | ✓ | ✓ |
| **Tags** | Augment | Merge both | ✓ | ✓ |
| **Teaching Notes** | Primary | Manual only | ✓ | - |
| **Strum Style** | Primary | Manual only | ✓ | - |
| **Finger-picking Style** | Primary | Manual only | ✓ | - |
| **No. of Chords** | Override | Manual first | ✓ | ✓ |
| **Chords** | Override | Manual first | ✓ | ✓ |
| **Chord Numerals** | Override | Manual first | ✓ | ✓ |
| **Time Signature** | Override | Manual first | ✓ | ✓ |
| **BPM** | Override | Manual first | ✓ | ✓ |
| **Tempo** | Override | Manual first | ✓ | ✓ |

**Priority Definitions**:
- **Primary** - Manual SOU is the only/authoritative source
- **Override** - If Manual SOU has a value, use it; otherwise use API
- **Augment** - Merge Manual SOU + API data (dedupe, combine)
- **N/A** - API only (Manual SOU doesn't have this field)

---

## Important Notes

1. **Three databases, not two** - Manual SOU → SOU (enriched) → Frontend JSON
2. **Seed database** = Discovery tool for admins (quantity focus)
3. **SOU database** = Production data for end users (quality focus)
4. **Manual SOU database** = Teaching-specific source of truth
5. The seed database feeds into the SOU database after manual curation
6. Manual SOU fields override API data for teaching-specific metadata
7. Enrichment strategies differ between databases (see API_ENRICHMENT_STRATEGY.md)
