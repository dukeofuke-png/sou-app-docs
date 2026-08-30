# Materials Management System Documentation

**Last Updated:** 3 December 2025  
**Status:** Complete & Production Ready

## Overview

The Materials Management System automatically detects PDF song sheets and TABs from Google Drive, matches them to songs in the SQLite database, and exposes them via API for the frontend. This system replaces the manual materials tracking and enables students to access multiple versions of each song (different keys, arrangements, etc.).

---

## Architecture

### Components

1. **Google Drive Materials Folder** (Source)
   - Path: `/Song Sheets PDF ONLY - School of Uke/`
   - Structure: One subfolder per song, PDFs inside
   - Example: `(You're the) Devil in Disguise - Elvis Presley (1963)/`
     - Contains: Multiple PDFs with different keys/arrangements

2. **Scanner Script** (`scanMaterialsToDatabase.js`)
   - Scans Google Drive folder recursively (depth 1)
   - Parses PDF filenames to extract metadata
   - Matches PDFs to songs using fuzzy matching
   - Updates SQLite database with all versions

3. **SQLite Database** (`sou_songs.db`)
   - Stores materials data in 3 formats:
     - Legacy fields: `song_sheet_path`, `melody_tab_path` (absolute paths)
     - New fields: `song_sheet_rel_path`, `melody_tab_rel_path` (relative paths)
     - JSON array: `materials_json` (all versions with metadata)

4. **Express API Server** (`server.js`)
   - Endpoint: `GET /songs` (public, no auth)
   - Endpoint: `GET /materials/*` (serves PDFs via relative path)
   - Transforms DB data to frontend format
   - Parses `materials_json` string → array

5. **React Frontend** (`SongDetailModal.js`, `App.js`)
   - Fetches songs from API on mount
   - Displays all material versions in song detail modal
   - Builds URLs: `${materialsBaseUrl}/materials/${relativePath}`

---

## Data Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│ 1. Google Drive Materials Folder                                    │
│    /Song Sheets PDF ONLY - School of Uke/                          │
│    └── Song Name - Artist (Year)/                                   │
│        ├── Song Name - Artist - Key C.pdf                          │
│        ├── Song Name - Artist - Key F (Easy).pdf                   │
│        └── Song Name TAB - Key C.pdf                               │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│ 2. Scanner (scanMaterialsToDatabase.js)                            │
│    • Scans subdirectories for PDFs                                  │
│    • Parses filename: title, artist, year, key, isTab              │
│    • Fuzzy matches to database songs                                │
│    • Updates database with paths and metadata                       │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│ 3. SQLite Database (sou_songs.db)                                   │
│    songs table:                                                      │
│    ├── song_sheet_status: "Yes" / NULL                             │
│    ├── tab_status: "Yes" / NULL                                    │
│    ├── song_sheet_path: "/full/path/to/first.pdf"                 │
│    ├── melody_tab_path: "/full/path/to/first_tab.pdf"             │
│    ├── song_sheet_rel_path: "folder/first.pdf"                    │
│    ├── melody_tab_rel_path: "folder/first_tab.pdf"                │
│    └── materials_json: '[{filename, relativePath, ...}, {...}]'   │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│ 4. Express API (server.js)                                          │
│    GET /songs → Returns all songs with materials array              │
│    GET /materials/* → Serves PDF file from Google Drive             │
│                                                                      │
│    Field mapping:                                                    │
│    • materials_json (string) → materials (parsed array)            │
│    • song_sheet_rel_path → songSheetRelPath                        │
│    • melody_tab_rel_path → melodyTabRelPath                        │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│ 5. React Frontend (localhost:3000)                                  │
│    • Fetches songs on mount: fetch('/songs')                        │
│    • Displays materials in SongDetailModal                          │
│    • Builds PDF URLs: /materials/{relativePath}                     │
│    • Shows all versions with keys/arrangements                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## File Naming Conventions

### Standard Format
```
[Song Name] - [Artist] ([Year]) [Key X] [Additional Info].pdf
```

**Examples:**
- `(You're the) Devil in Disguise - Elvis Presley (1963) - Key F.pdf`
- `5 Years Time - Noah and the Whale (2008) Key C.pdf`
- `Fly Me To The Moon - Bart Howard - Key G (Easy version).pdf`

### TAB Files
Must contain "TAB" (case-insensitive) anywhere in filename:
- `Ain't No Sunshine TAB - Bill Withers (1971) Key C.pdf`
- `Don't Speak - No Doubt TABS Key Am.pdf`
- `Fly Me To The Moon (In Other Words) TABS.pdf`

### Sloppy Formatting (Still Works!)
Scanner handles missing artists or inconsistent formatting:
- `Be Thankful TAB Key D Major.pdf` (no artist, no dash)
- `Take On Me - AHA (1984).pdf` (works despite "AHA" vs "A-ha")
- `Stayin Alive - Bee Gees (1977) Key Gm.pdf` (works despite "Stayin" vs "Staying")

---

## Scanner Implementation

### File: `scanMaterialsToDatabase.js`

**Purpose:** Populate database with materials from Google Drive folder

**Run Command:**
```bash
cd materials-server
node scanMaterialsToDatabase.js
```

**What It Does:**

1. **Scan Folders**
   - Reads all subdirectories in materials folder
   - For each folder, finds all `.pdf` files
   - Depth: 1 level (song folders only, not recursive)

2. **Parse Filename**
   ```javascript
   parseFilename(filename)
   // Returns: { title, artist, year, key, isTab }
   ```
   - Extracts title before first " - "
   - Extracts artist between " - " and "(Year)"
   - Identifies TABs via regex: `/\bTAB[Ss]?\b/i`
   - Extracts key: `/Key ([A-G][#b]?m?\s*(?:Major|Minor)?)/i`

3. **Fuzzy Match to Database**
   ```javascript
   findMatchingSong(parsedInfo)
   ```
   - **Pass 1:** Exact title + artist match (normalized)
   - **Pass 2:** Partial artist match (handles "The Bee Gees" vs "Bee Gees")
   - **Pass 3:** Title-only match (for files missing artist)
   - Normalization: lowercase, remove punctuation, collapse whitespace

4. **Update Database**
   ```javascript
   updateSongMaterials(songId, materials)
   ```
   - Stores first sheet in `song_sheet_path` / `song_sheet_rel_path`
   - Stores first TAB in `melody_tab_path` / `melody_tab_rel_path`
   - Stores all versions in `materials_json` as JSON array:
   ```json
   [
     {
       "filename": "Fly Me To The Moon - Key G.pdf",
       "relativePath": "Fly Me.../Fly Me To The Moon - Key G.pdf",
       "absolutePath": "/Users/.../Fly Me To The Moon - Key G.pdf",
       "isTab": false,
       "key": "G"
     }
   ]
   ```
   - Updates status: `song_sheet_status` = "Yes", `tab_status` = "Yes"

**Output:**
```
🔍 Starting materials scan...
📁 Materials folder: /Users/.../Song Sheets PDF ONLY - School of Uke
📂 Found 213 folders
✅ (You're the) Devil in Disguise by Elvis Presley → 3 sheet(s)
✅ 5 Years Time by Noah and The Whale → 1 sheet(s)
...
✅ Fly Me To The Moon (In Other Words) by Bart Howard... → 7 sheet(s), 1 TAB(s)
⚠️  No match found for: Folsom Prison Blues by Johnny Cash
...
✅ Scan complete!
📊 Stats:
   - Folders scanned: 214
   - PDFs processed: 363
   - Songs matched: 197
   - Songs with sheets: 195
   - Songs with TABs: 49
```

---

## Database Schema

### Existing Columns (Before Materials System)
```sql
-- Core song data
id INTEGER PRIMARY KEY
title TEXT
artist TEXT
year INTEGER
genre TEXT
-- ... 180+ other columns
```

### New Columns (Materials System)
```sql
-- Status flags
song_sheet_status TEXT      -- "Yes" / NULL
tab_status TEXT             -- "Yes" / NULL

-- Legacy absolute paths (backward compatibility)
song_sheet_path TEXT        -- Full path to first sheet PDF
melody_tab_path TEXT        -- Full path to first TAB PDF

-- New relative paths (preferred for API)
song_sheet_rel_path TEXT    -- "folder/file.pdf"
melody_tab_rel_path TEXT    -- "folder/tab.pdf"

-- All versions as JSON array (PRIMARY SOURCE)
materials_json TEXT         -- '[{filename, relativePath, absolutePath, isTab, key}, ...]'
```

**Example Data:**
```sql
-- Song: Fly Me To The Moon (In Other Words)
song_sheet_status = "Yes"
tab_status = "Yes"
song_sheet_path = "/Users/.../Fly Me To The Moon - Key G (Easy version).pdf"
melody_tab_path = "/Users/.../Fly Me To The Moon (In Other Words) TABS.pdf"
song_sheet_rel_path = "Fly Me.../Fly Me To The Moon - Key G (Easy version).pdf"
melody_tab_rel_path = "Fly Me.../Fly Me To The Moon (In Other Words) TABS.pdf"
materials_json = '[
  {"filename": "...Key G (Easy version).pdf", "relativePath": "...", "isTab": false, "key": "G"},
  {"filename": "...Key G.pdf", "relativePath": "...", "isTab": false, "key": "G"},
  {"filename": "...Key C (Easy version).pdf", "relativePath": "...", "isTab": false, "key": "C"},
  ... (8 total)
]'
```

---

## API Implementation

### Endpoint: `GET /songs`

**Location:** `materials-server/server.js` (line ~869)

**Purpose:** Public endpoint to fetch all songs with materials

**Implementation:**
```javascript
app.get('/songs', async (req, res) => {
  try {
    const songs = await dbManager.getAllSongs();
    const formatted = songs.map(song => toFrontendFormat(song));
    res.json(formatted);
  } catch (e) {
    console.error('Public /songs error:', e.message);
    res.status(500).json({ error: 'Failed to fetch songs' });
  }
});
```

**Field Mapping:**
```javascript
const fieldMappings = {
  'song_sheet_rel_path': 'songSheetRelPath',
  'melody_tab_rel_path': 'melodyTabRelPath',
  'materials_json': 'materials',  // Parsed from string to array
  // ... 50+ other mappings
};
```

**Special Handling for materials_json:**
```javascript
if (outputKey === 'materials' && typeof value === 'string' && value) {
  try {
    result[outputKey] = JSON.parse(value);
  } catch (e) {
    console.error(`Failed to parse materials_json:`, e);
    result[outputKey] = null;
  }
  return;
}
```

**Response Format:**
```json
[
  {
    "id": 123,
    "title": "Fly Me To The Moon (In Other Words)",
    "artist": "Bart Howard, Astrud Gilberto, Frank Sinatra",
    "songSheetStatus": "Yes",
    "tabStatus": "Yes",
    "songSheetRelPath": "Fly Me.../Fly Me To The Moon - Key G.pdf",
    "melodyTabRelPath": "Fly Me.../Fly Me To The Moon TABS.pdf",
    "materials": [
      {
        "filename": "Fly Me To The Moon - Key G (Easy version).pdf",
        "relativePath": "Fly Me.../...Key G (Easy version).pdf",
        "absolutePath": "/Users/.../...Key G (Easy version).pdf",
        "isTab": false,
        "key": "G"
      },
      ... (7 more)
    ]
  },
  ... (214 more songs)
]
```

### Endpoint: `GET /materials/*`

**Location:** `materials-server/server.js` (line ~885)

**Purpose:** Serve PDF files from Google Drive via relative path

**Implementation:**
```javascript
app.get('/materials/*', (req, res) => {
  const relativePath = req.params[0];
  const fullPath = path.join(MATERIALS_BASE, relativePath);
  
  res.sendFile(fullPath, (err) => {
    if (err) {
      console.error('Error serving file:', err);
      res.status(404).json({ error: 'File not found' });
    }
  });
});
```

**Example Request:**
```
GET /materials/Fly%20Me%20To%20The%20Moon%20(In%20Other%20Words)%20-%20Bart%20Howard%2C%20Astrud%20Gilberto%2C%20Frank%20Sinatra%20et%20al/Fly%20Me%20To%20The%20Moon%20(In%20Other%20Words)%20-%20Bart%20Howard%20-%20Astrud%20Gilberto%20-%20Key%20G.pdf

→ Serves: /Users/.../Song Sheets PDF ONLY - School of Uke/Fly Me.../Fly Me...Key G.pdf
→ Response: PDF file (application/pdf)
```

---

## Frontend Implementation

### File: `sou-song-browser/src/App.js`

**Changes Made:**

1. **Removed Static JSON Imports**
   ```javascript
   // BEFORE
   import sanitizedData from "./data/songs_app_export_sanitized.json";
   import legacyData from "./data/songs_app_export_merged.json";
   
   // AFTER
   // Removed - now fetches from API
   ```

2. **Added API Configuration**
   ```javascript
   const API_URL = process.env.REACT_APP_API_URL || 'http://localhost:3002';
   ```

3. **Added State Management**
   ```javascript
   const [songs, setSongs] = useState([]);
   const [songsLoading, setSongsLoading] = useState(true);
   const [songsError, setSongsError] = useState(null);
   ```

4. **Added Fetch Effect**
   ```javascript
   useEffect(() => {
     const fetchSongs = async () => {
       try {
         setSongsLoading(true);
         const response = await fetch(`${API_URL}/songs`);
         if (!response.ok) {
           throw new Error(`Failed to fetch songs: ${response.statusText}`);
         }
         const data = await response.json();
         setSongs(Array.isArray(data) ? data : []);
         setSongsError(null);
       } catch (err) {
         console.error('Error fetching songs:', err);
         setSongsError(err.message);
         setSongs([]);
       } finally {
         setSongsLoading(false);
       }
     };
     fetchSongs();
   }, []);
   ```

5. **Added Loading/Error UI**
   ```javascript
   {songsLoading ? (
     <div>Loading songs from database...</div>
   ) : songsError ? (
     <div>❌ Error loading songs: {songsError}</div>
   ) : (
     <>{/* Normal UI */}</>
   )}
   ```

### File: `sou-song-browser/src/SongDetailModal.js`

**Changes Made:**

1. **Added materials Array Support**
   ```javascript
   {song.materials && song.materials.length > 0 ? (
     <div className="external-links">
       {song.materials.map((material, index) => {
         const url = buildMaterialsUrl(material.relativePath);
         // Build display name from metadata
         let displayName = material.isTab ? '🎵 Melody TAB' : '📄 Song Sheet';
         if (material.key) displayName += ` - Key ${material.key}`;
         // Add arrangement info from filename
         if (/easy/i.test(material.filename)) displayName += ' (Easy)';
         // ...
         return <a key={index} href={url}>{displayName}</a>;
       })}
     </div>
   ) : (
     // Fallback to legacy single sheet/tab format
   )}
   ```

2. **Enhanced Display Names**
   - Detects key signatures from metadata
   - Identifies arrangement types from filename:
     - `(Easy)` - Easy version
     - `(v2)` - Version number
     - `(Ensemble)` - Ensemble arrangement
     - `(Chords & Lyrics)` - Chords with lyrics
     - `(Chords Only)` - Chords only

**Example Output for "Fly Me To The Moon":**
```
Available Materials
📄 Song Sheet - Key G (Easy)
📄 Song Sheet - Key G
📄 Song Sheet - Key C (Easy)
📄 Song Sheet (Ensemble)
📄 Song Sheet (Chords & Lyrics)
📄 Song Sheet (Chords Only)
📄 Song Sheet
🎵 Melody TAB
```

---

## Current Statistics

**As of 3 December 2025:**

### Materials Coverage
- **Total Songs in Database:** 215
- **Songs with Materials:** 193 (90%)
- **Songs with Sheets:** 191 (89%)
- **Songs with TABs:** 49 (23%)
- **Total PDFs Stored:** 339

### Match Quality
- **PDF Files Found:** 363
- **Folders Scanned:** 213
- **Successful Matches:** 197 songs (92%)
- **Unmatched Folders:** 14 (6.5%)
- **Songs Without Materials:** 22 (10%)

### Multiple Versions
- **Most PDFs per Song:** 8 ("Fly Me To The Moon")
- **Average PDFs per Song:** 1.8
- **Songs with 2+ Versions:** 94 (49%)
- **Songs with 3+ Versions:** 38 (20%)

---

## Common Issues & Solutions

### Issue 1: PDF Not Appearing in Database

**Symptoms:** Scanner processes PDF but song shows no materials

**Causes & Solutions:**

1. **Filename doesn't match database title**
   - Check exact title in database vs folder name
   - Use `findMissingMaterials.js` to identify mismatches
   ```bash
   node findMissingMaterials.js
   ```

2. **Artist mismatch**
   - Folder: "R.E.M" → Database: "R.E.M."
   - Scanner handles partial matches but may miss punctuation
   - Solution: Add artist variations to fuzzy matcher

3. **Special characters**
   - Folder: "Sh-boom" → Scanner splits incorrectly
   - Solution: Use normalization function that preserves hyphens in titles

**Fix:** Re-run scanner after renaming folder or updating database title

### Issue 2: Materials Not Showing in Frontend

**Symptoms:** Database has materials but modal shows "No materials available"

**Diagnostic Steps:**

1. **Check API Response**
   ```bash
   curl -s "http://localhost:3002/songs" | jq '.[] | select(.title == "Song Name") | .materials'
   ```
   Expected: Array of material objects
   Actual: `null` or empty array → Issue is in API

2. **Check Database**
   ```bash
   sqlite3 data/sou_songs.db "SELECT materials_json FROM songs WHERE title='Song Name';"
   ```
   Expected: JSON array string
   Actual: `NULL` → Issue is in scanner

3. **Check Frontend State**
   - Open browser console
   - Check `songs` state: `$r.props.songs[0].materials`
   - Should see array of materials

**Solutions:**
- If API returns null: Check field mapping in `server.js`
- If DB is null: Re-run scanner
- If frontend doesn't receive: Check fetch() call and state updates

### Issue 3: PDF Link Returns 404

**Symptoms:** Click material link, get "File not found"

**Causes:**

1. **Incorrect relative path**
   - Check `relativePath` value in database
   - Should NOT include "Song Sheets PDF ONLY - School of Uke/"
   - Should be: `"folder/file.pdf"`

2. **Materials base path wrong**
   - Check `MATERIALS_PATH` env var in server
   - Default: `/Users/.../Song Sheets PDF ONLY - School of Uke`

3. **File moved/renamed**
   - PDF was moved after scanner ran
   - Solution: Re-run scanner to update paths

**Debug:**
```bash
# Check what the server sees
ls "/Users/matthew/Library/CloudStorage/.../Song Sheets PDF ONLY - School of Uke/folder/"

# Test direct file access
curl -I "http://localhost:3002/materials/folder/file.pdf"
```

### Issue 4: Scanner Shows Wrong Number of Matches

**Symptoms:** Scanner says "197 matched" but database shows 193

**Explanation:** This is normal!
- Scanner counts folder-level matches (some folders have 0 PDFs after filtering)
- Database counts songs with `materials_json != NULL`
- Difference is usually 2-5 songs

**Verify:**
```bash
# Count materials in DB
sqlite3 data/sou_songs.db "SELECT COUNT(*) FROM songs WHERE materials_json IS NOT NULL;"

# Count total PDFs stored
sqlite3 data/sou_songs.db "SELECT SUM(json_array_length(materials_json)) FROM songs WHERE materials_json IS NOT NULL;"
```

---

## Maintenance Procedures

### Adding New Song Materials

**When:** Tutor uploads new PDF to Google Drive

**Steps:**

1. **Upload PDF to Correct Folder**
   - Navigate to: `/Song Sheets PDF ONLY - School of Uke/`
   - Create folder if needed: `Song Name - Artist (Year)/`
   - Upload PDF with proper naming: `Song Name - Artist - Key X.pdf`

2. **Re-run Scanner**
   ```bash
   cd materials-server
   node scanMaterialsToDatabase.js
   ```
   - Scanner updates only changed files (via timestamp check)
   - Safe to run repeatedly

3. **Verify in Database**
   ```bash
   sqlite3 data/sou_songs.db "SELECT title, json_array_length(materials_json) FROM songs WHERE title LIKE '%Song Name%';"
   ```

4. **Test in Browser**
   - Refresh page: `http://localhost:3000`
   - Search for song
   - Click to open modal
   - Verify new PDF appears

**No Server Restart Needed!** API reads from database on each request.

### Updating Existing Materials

**When:** PDF needs to be replaced or renamed

**Steps:**

1. **Rename/Replace File in Google Drive**
   - Keep same folder structure
   - Follow naming conventions

2. **Clear Materials Data** (Optional)
   ```bash
   sqlite3 data/sou_songs.db "UPDATE songs SET materials_json=NULL WHERE title='Song Name';"
   ```

3. **Re-run Scanner**
   ```bash
   node scanMaterialsToDatabase.js
   ```

### Bulk Operations

**Re-scan All Materials:**
```bash
# Clear all materials data
sqlite3 data/sou_songs.db "UPDATE songs SET song_sheet_status=NULL, tab_status=NULL, song_sheet_path=NULL, melody_tab_path=NULL, song_sheet_rel_path=NULL, melody_tab_rel_path=NULL, materials_json=NULL;"

# Re-scan everything
node scanMaterialsToDatabase.js
```

**Find Unmatched Materials:**
```bash
node findMissingMaterials.js
```

**Export Materials Report:**
```bash
sqlite3 data/sou_songs.db "SELECT title, artist, song_sheet_status, tab_status, json_array_length(materials_json) as num_files FROM songs WHERE materials_json IS NOT NULL ORDER BY num_files DESC;" > materials_report.csv
```

---

## Performance Considerations

### Scanner Performance
- **Initial Scan:** ~15-20 seconds (363 PDFs)
- **Incremental Scan:** ~5-10 seconds (only changed files)
- **Database Updates:** Batched via prepared statements

### API Performance
- **Response Time:** ~50-100ms for 215 songs
- **Payload Size:** ~1.5MB (includes all metadata)
- **Caching:** None currently (reads DB on each request)
- **Optimization:** Consider caching with mtime check

### Frontend Performance
- **Initial Load:** ~2-3 seconds (fetch + render)
- **Song Modal:** Instant (data already loaded)
- **PDF Loading:** Depends on Google Drive sync

**Future Optimizations:**
1. Add Redis caching layer for `/songs` endpoint
2. Implement pagination (currently loads all 215)
3. Add service worker for offline PDF access
4. Compress materials_json in database (gzip)

---

## Environment Variables

### Backend (materials-server)

```bash
# .env file
MATERIALS_PATH="/Users/matthew/Library/CloudStorage/GoogleDrive-info.schoolofuke@gmail.com/My Drive/School of Uke Lesson Content/Lesson Content Tutor Access Only - School of Uke /Song Sheets PDF ONLY - School of Uke"

PORT=3002
NODE_ENV=production

# Disable watcher (use scanner script instead)
ENABLE_MATERIALS_WATCHER=false
```

### Frontend (sou-song-browser)

```bash
# .env file
REACT_APP_API_URL=http://localhost:3002
REACT_APP_MATERIALS_URL=http://localhost:3002
```

**Production:**
```bash
REACT_APP_API_URL=https://api.schoolofuke.com
REACT_APP_MATERIALS_URL=https://api.schoolofuke.com
```

---

## Testing Checklist

### Backend Tests

- [ ] Scanner runs without errors
  ```bash
  node scanMaterialsToDatabase.js
  # Should show: "✅ Scan complete!"
  ```

- [ ] Database populated correctly
  ```bash
  sqlite3 data/sou_songs.db "SELECT COUNT(*) FROM songs WHERE materials_json IS NOT NULL;"
  # Should show: 193
  ```

- [ ] API returns materials array
  ```bash
  curl -s "http://localhost:3002/songs" | jq '.[0].materials | type'
  # Should show: "array"
  ```

- [ ] PDF endpoint serves files
  ```bash
  curl -I "http://localhost:3002/materials/[any-valid-path].pdf"
  # Should show: 200 OK, Content-Type: application/pdf
  ```

### Frontend Tests

- [ ] Page loads without errors
  - Open: `http://localhost:3000`
  - Check console: No errors
  - Should show: "Total songs in database: 215"

- [ ] Song list displays correctly
  - Songs visible in table
  - Filters work
  - Search works

- [ ] Song modal shows materials
  - Click any song with materials
  - "Available Materials" section visible
  - Links are clickable

- [ ] Multiple materials display
  - Click "Fly Me To The Moon"
  - Should show 8 material links
  - Each with key/arrangement info

- [ ] PDF opens correctly
  - Click any material link
  - PDF opens in new tab
  - File loads correctly

### Integration Tests

- [ ] End-to-end flow
  1. Upload new PDF to Google Drive
  2. Run scanner
  3. Refresh frontend
  4. Find song in list
  5. Open modal
  6. Click material link
  7. PDF opens

---

## Future Enhancements

### Phase 2: Real-time Watcher

**Goal:** Auto-detect new PDFs without manual scanner runs

**Implementation:**
- Fix `materialsWatcher.js` with improved matching logic
- Use `chokidar` to watch Google Drive folder
- Update database on file add/change/delete
- Emit WebSocket event to frontend for live updates

**Status:** Currently disabled (broken SQL query)

### Phase 3: Admin Materials Manager

**Goal:** Let admins upload/organize PDFs via web UI

**Features:**
- Drag-and-drop PDF upload
- Auto-parse filename or manual metadata entry
- Preview PDFs in browser
- Bulk operations (rename, delete, reassign)

### Phase 4: Student Download Packs

**Goal:** Allow students to download all materials for a song as ZIP

**Features:**
- "Download All" button in modal
- Server generates ZIP on-the-fly
- Include song metadata in ZIP
- Track download statistics

### Phase 5: Version Management

**Goal:** Track PDF versions and update history

**Features:**
- Store version timestamps
- Show "Updated X days ago"
- Notify students of new versions
- Archive old versions

---

## Troubleshooting Guide

### Problem: Scanner finds 0 PDFs

**Check:**
1. Materials folder path correct?
   ```bash
   echo $MATERIALS_PATH
   ls "$MATERIALS_PATH"
   ```
2. PDFs are in subfolders (not root)?
3. File extensions are lowercase `.pdf`?

### Problem: API returns empty materials array

**Check:**
1. Database has data?
   ```bash
   sqlite3 data/sou_songs.db "SELECT COUNT(*) FROM songs WHERE materials_json IS NOT NULL;"
   ```
2. Field mapping includes `materials_json`?
   ```javascript
   'materials_json': 'materials'
   ```
3. JSON parsing working?
   - Check server logs for parse errors

### Problem: Frontend shows loading forever

**Check:**
1. API server running?
   ```bash
   curl http://localhost:3002/health
   ```
2. CORS enabled?
   - Check Network tab in browser console
3. Fetch call correct?
   - Should be: `fetch('http://localhost:3002/songs')`

### Problem: Some songs missing materials

**Expected!** Not all songs have PDF materials yet.

**Verify it's not a bug:**
1. Check if folder exists in Google Drive
2. Run `findMissingMaterials.js` to see unmatched
3. Check filename format matches conventions

---

## Related Documentation

- **API Reference:** `API_DATAFIELDS_REFERENCE.md`
- **Database Schema:** `DATABASE_INVENTORY.md`
- **Field Mapping:** `FIELD_MAPPING_ARCHITECTURE.md`
- **Admin Dashboard:** `ADMIN_DASHBOARD.md`
- **Project Status:** `PROJECT_STATUS.md`

---

## Support

**Issues:** Report to development team  
**Questions:** Check `QUICK_REFERENCE.txt`  
**Updates:** See git commit history

