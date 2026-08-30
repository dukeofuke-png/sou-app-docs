# Student UI Data Audit - Pre-SQLite Migration
**Date**: 2025-12-01  
**Purpose**: Document all data currently displayed in Student UI before SQLite export

## Current Data Source
- **File**: `sou-song-browser/src/data/songs_app_export_merged.json`
- **Total Songs**: 212
- **Format**: JSON array of song objects

## Fields Currently in Student UI JSON (57 fields)

### Core Identity (5 fields)
- `id` - Unique identifier (e.g., "leonard_cohen_hallelujah")
- `slug` - URL-friendly identifier
- `title` - Song title
- `artist` - Artist name
- `year` - Release year

### Release Information (6 fields)
- `releaseDate` - Full release date
- `releaseDateSource` - Source of release date (e.g., "Spotify", "MusicBrainz")
- `releaseMonth` - Month number
- `season` - Season (Winter, Spring, Summer, Autumn)
- `era` - Era classification
- `spotifyReleaseDate` - Spotify-specific release date

### Musical Properties (10 fields)
- `originalKey` - Original key signature
- `souKeys` - Array of keys taught at SOU
- `mode` - Major/Minor
- `timeSignature` - Time signature (e.g., "4/4", "3/4")
- `bpm` - Beats per minute
- `tempoLabel` - Tempo description (e.g., "Moderate", "Fast")
- `numChords` - Number of chords
- `chords` - Array of chord names
- `chordNumerals` - Roman numeral chord progression
- `strumStyle` - Strumming pattern/style
- `fingerpickingStyle` - Fingerpicking pattern/style

### Teaching/Learning (5 fields)
- `level` - SOU difficulty level (1-5)
- `teachingNotes` - Instructor notes
- `songSheetStatus` - Status of song sheet
- `tabStatus` - Status of TAB
- `notes` - General notes

### Materials/Files (9 fields)
- `songSheetPath` - Path to song sheet PDF
- `songSheetUrl` - URL to song sheet
- `hasSongSheet` - Boolean flag
- `melodyTabPath` - Path to melody TAB PDF
- `tabUrl` - URL to TAB
- `hasTab` - Boolean flag
- `materialsFolder` - Folder containing materials
- `coverArtUrl` - Album/single cover art URL
- `discogsUrl` - Discogs page URL

### Genre/Classification (5 fields)
- `genre` - Primary genre(s)
- `tags` - Additional tags
- `genres_provenance` - Source tracking for genres
- `tags_per_source` - Tags organized by source
- `styles_per_source` - Styles organized by source

### Popularity & Charts (7 fields)
- `spotifyPopularity` - Spotify popularity score (0-100)
- `lastfmPlays` - Last.fm total plays
- `lastfmListeners` - Last.fm unique listeners
- `popularityTier` - Calculated tier (Low/Medium/High/Massive)
- `chartPeak` - Peak chart position
- `top10` - Boolean flag for Top 10 hit
- `top40` - Boolean flag for Top 40 hit

### Credits (4 fields)
- `songwriters` - Songwriter names
- `songwritersSource` - Source of songwriter info
- `wordsAndMusic` - Combined writer credit
- `publisher` - Music publisher
- `hasWriterCredits` - Boolean flag

### External Links (5 fields)
- `spotifyTrackId` - Spotify track ID
- `youtubeUrl` - YouTube video URL
- `youtubeVideoId` - YouTube video ID
- `youtubeViews` - YouTube view count
- `wikipediaUrl` - Wikipedia page URL
- `wikipediaIntro` - Wikipedia introduction text (first paragraph)

---

## Student UI Display Sections

### 1. Song Card (List View)
**Displays:**
- Title
- Artist  
- Year
- Level badge
- Popularity tier badge
- Top 10/40 badge
- Chart peak badge
- Cover art thumbnail

### 2. Song Detail Modal - Header
**Displays:**
- Cover art (large)
- Title
- Artist
- Popularity tier badge
- Top 10 badge
- Top 40 badge
- Chart peak badge (e.g., "Peak #3")
- External links: Discogs, Spotify Track, YouTube

### 3. Song Detail Modal - Basic Info Section
**Displays:**
- Year
- Release Date (with source)
- Written by (songwriters with source)
- Genre (classified from combined genre/tags)
- Tags (including season, era, nationality markers)

### 4. Song Detail Modal - Chart Performance Section
**NEW FIELDS FROM SQLITE** (not currently in JSON):
- `chartPeakUs` - US Billboard Hot 100 peak
- `chartPeakUk` - UK Singles Chart peak
- `chartPeakAus` - Australian Singles Chart peak
- `chartPeakCanada` - Canadian Hot 100 peak
- `chartPeakGermany` - German Singles Chart peak
- `chartPeakFrance` - French Singles Chart peak
- `chartPeakSweden` - Swedish Singles Chart peak
- `chartPeakIreland` - Irish Singles Chart peak
- `chartPeakNetherlands` - Dutch Singles Chart peak
- `chartPeakNewZealand` - New Zealand Singles Chart peak
- `chartPeakSwitzerland` - Swiss Singles Chart peak
- `wikipediaChartsText` - All charts summary text

### 5. Song Detail Modal - Wikipedia Background Section
**NEW FIELDS FROM SQLITE** (not currently in JSON):
- `wikipediaBackground` - Background/history text

### 6. Song Detail Modal - Wikipedia Composition Section
**NEW FIELDS FROM SQLITE** (not currently in JSON):
- `wikipediaComposition` - Composition/musical structure text

### 7. Song Detail Modal - Cover Versions Section
**NEW FIELDS FROM SQLITE** (not currently in JSON):
- `coverVersionsList` - Delimited list of cover versions (|||)
- `coverVersionsCount` - Number of cover versions

### 8. Song Detail Modal - Song Overview Section
**Displays:**
- Original Key
- SOU Keys
- Mode
- SOU Level
- Time Signature
- BPM
- Tempo Label
- Number of Chords
- Chords (comma-separated)
- Strum Style
- Fingerpicking Style

### 9. Song Detail Modal - Teaching Notes Section
**Displays:**
- Teaching Notes (full text)

### 10. Song Detail Modal - Available Materials Section
**Displays:**
- Song Sheet PDF links (with key detection from filename)
- Melody TAB PDF links (with key detection from filename)
- Formatted names: "Song Sheet in Key C", "Melody TAB in Key Am"

### 11. Song Detail Modal - About This Song Section
**Displays:**
- Wikipedia Intro (first paragraph)
- "Read more on Wikipedia →" link

### 12. Song Detail Modal - Popularity & Charts Section
**Displays:**
- Top 10 Hit badge (if applicable)
- Top 40 Hit badge (if applicable)
- Peak Position badge (e.g., "Peak Position: #3")
- Last.fm Plays (with progress bar, max scale: 100,000)
- Last.fm Listeners (with progress bar, max scale: 50,000)
- Spotify Popularity (with progress bar, 0-100 scale)
- Popularity Tier badge (with color coding)

### 13. Song Detail Modal - Credits Section (Optional)
**Displays:**
- Words & Music
- Publisher

### 14. Song Detail Modal - Notes Section (Optional)
**Displays:**
- General Notes (full text)

---

## Fields ADDED by SQLite Migration (Not in Current JSON)

### International Chart Data (11 fields)
- `chart_peak_us` → `chartPeakUs`
- `chart_peak_uk` → `chartPeakUk`
- `chart_peak_aus` → `chartPeakAus`
- `chart_peak_canada` → `chartPeakCanada`
- `chart_peak_germany` → `chartPeakGermany`
- `chart_peak_ireland` → `chartPeakIreland`
- `chart_peak_france` → `chartPeakFrance`
- `chart_peak_netherlands` → `chartPeakNetherlands`
- `chart_peak_new_zealand` → `chartPeakNewZealand`
- `chart_peak_sweden` → `chartPeakSweden`
- `chart_peak_switzerland` → `chartPeakSwitzerland`

### Wikipedia Enrichment (4 fields)
- `wikipedia_background` → `wikipediaBackground`
- `wikipedia_composition` → `wikipediaComposition`
- `cover_versions_list` → `coverVersionsList`
- `cover_versions_count` → `coverVersionsCount`

### Additional Metadata (1 field)
- `wikipedia_charts_text` → `wikipediaChartsText` (all charts summary)

---

## Data Transformation Notes

### Field Name Conversion (snake_case → camelCase)
The server's `toCamelCase()` function converts SQLite snake_case field names to camelCase for frontend compatibility:
- `chart_peak_us` → `chartPeakUs`
- `wikipedia_background` → `wikipediaBackground`
- `cover_versions_list` → `coverVersionsList`

### Arrays vs Strings
**Current JSON format:**
- `chords`: Array of strings
- `souKeys`: Array of strings
- `genre`: Can be string or array
- `tags`: Can be string or array

**SQLite format:**
- Stored as comma-separated strings
- Must be split into arrays during export

### Cover Versions Format
- **Delimiter**: `|||` (triple pipe)
- **Structure**: `"artist name: description|||artist name: description"`
- **Display**: Split by `|||`, then split by `:` to separate artist from description

---

## Critical Data Preservation Checklist

### ✅ Must Preserve (Currently in JSON, must keep in export)
- [ ] All 57 existing fields listed above
- [ ] Array formatting for `chords`, `souKeys`
- [ ] Genre/tag classification logic
- [ ] External link URLs (Spotify, YouTube, Discogs, Wikipedia)
- [ ] Cover art URLs
- [ ] Material file paths (both absolute and relative)
- [ ] Teaching metadata (level, notes, status fields)
- [ ] Popularity metrics (Last.fm, Spotify)

### ✅ Will Add (New fields from SQLite)
- [ ] 11 international chart peak positions
- [ ] Wikipedia background text
- [ ] Wikipedia composition text  
- [ ] Cover versions list with descriptions
- [ ] Cover versions count
- [ ] All charts summary text

### ⚠️ Potential Issues to Watch

1. **Field Name Mapping**
   - Ensure all snake_case → camelCase conversions are correct
   - Verify no fields are lost in translation

2. **Data Type Consistency**
   - Arrays must remain arrays (not converted to strings)
   - Numbers must remain numbers (not converted to strings)
   - Booleans must remain booleans

3. **Special Characters**
   - Ensure proper escaping in JSON
   - Handle quotes, apostrophes in text fields
   - Preserve newlines in long text fields

4. **File Paths**
   - Preserve both absolute paths (`songSheetPath`) and relative paths
   - Maintain materialsFolder structure
   - Keep URL encoding consistent

5. **Genre/Tag Logic**
   - The `classifyTerms()` function in SongDetailModal.js uses heuristics
   - Don't change the structure of genre/tags fields without updating the logic

---

## Export Script Requirements

The SQLite → JSON export script must:

1. **Include all 57 existing fields** from current JSON
2. **Add 16 new Wikipedia/chart fields** from SQLite
3. **Convert field names** from snake_case to camelCase
4. **Preserve data types** (arrays, numbers, booleans)
5. **Split comma-separated strings** into arrays where appropriate
6. **Handle null values** gracefully (don't omit fields)
7. **Maintain sort order** (if any)
8. **Validate output** against current JSON structure

---

## Rollback Plan

If issues arise after migration:

1. **Current JSON is backed up** in this repository
2. **Revert to previous commit** in git if needed
3. **Copilot instructions** document the migration
4. **This audit document** provides complete field inventory

---

## Next Steps

1. ✅ Complete batch enrichment (49/215 done, ~166 remaining)
2. ⏳ Create SQLite → JSON export script
3. ⏳ Test export with sample data (5-10 songs)
4. ⏳ Validate all fields are present and correctly formatted
5. ⏳ Full export of all 215 songs
6. ⏳ Backup current JSON
7. ⏳ Replace with new enriched JSON
8. ⏳ Rebuild React app
9. ⏳ Test student UI thoroughly
10. ⏳ Deploy to production

---

**End of Audit Document**
