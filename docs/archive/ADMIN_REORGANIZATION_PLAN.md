# Admin Frontend Reorganization - Implementation Plan

**Date**: November 24, 2025  
**Status**: Planning Phase  
**Goal**: Reorganize admin dashboard to separate seed database discovery from SOU database maintenance

---

## 1. OVERVIEW

### Current Issues
- Dashboard has unnecessary stats (BPM, Key) and redundant Quick Action buttons
- Search Database page designed for wrong database (SOU instead of seed)
- No clear workflow to promote songs from seed → SOU database
- Popularity Catalog is redundant
- Conceptual separation between seed/SOU databases is unclear

### Proposed Solution
Restructure menu and pages to create logical separation:
- **Song Discovery** section → work with 47K seed database
- **Manage SOU Database** section → work with teaching database
- Remove redundancy and clarify workflows

---

## 2. MENU STRUCTURE CHANGES

### Before (7 menu items):
```
- Dashboard
- Search Database
- Chart Lookup
- Popularity Catalog
- Manage Songs
- Bulk Import
- Settings
```

### After (4 menu items + 2 sub-items):
```
- Dashboard
- Song Discovery
  ├─ Seed Database Search
  └─ Bulk Export
- Manage SOU Database
- Settings
```

### Chart Lookup
- Keep as separate feature (not part of this reorganization)
- Can be added back as needed

---

## 3. COMPONENT-BY-COMPONENT CHANGES

### 3.1 AdminLayout.js
**File**: `sou-song-browser/src/components/AdminLayout.js`

**Current menuItems**:
```javascript
const menuItems = [
  { id: 'dashboard', icon: <FiHome />, label: 'Dashboard' },
  { id: 'search', icon: <FiSearch />, label: 'Search Database' },
  { id: 'charts', icon: <FiTrendingUp />, label: 'Chart Lookup' },
  { id: 'popularity', icon: <FiStar />, label: 'Popularity Catalog' },
  { id: 'songs', icon: <FiMusic />, label: 'Manage Songs' },
  { id: 'bulk', icon: <FiUpload />, label: 'Bulk Import' },
  { id: 'settings', icon: <FiSettings />, label: 'Settings' }
];
```

**New menuItems**:
```javascript
const menuItems = [
  { id: 'dashboard', icon: <FiHome />, label: 'Dashboard' },
  { 
    id: 'discovery', 
    icon: <FiSearch />, 
    label: 'Song Discovery',
    subItems: [
      { id: 'seed-search', label: 'Seed Database Search' },
      { id: 'bulk-export', label: 'Bulk Export' }
    ]
  },
  { id: 'manage-sou', icon: <FiMusic />, label: 'Manage SOU Database' },
  { id: 'settings', icon: <FiSettings />, label: 'Settings' }
];
```

**Changes Required**:
- [ ] Add support for expandable menu items with sub-items
- [ ] Add expand/collapse icon for items with subItems
- [ ] Update CSS for nested menu structure
- [ ] Update active state logic to highlight parent when sub-item is active

---

### 3.2 AdminDashboard.js
**File**: `sou-song-browser/src/components/AdminDashboard.js`

#### Dashboard Page (renderDashboardOverview)

**Current Stats**:
```javascript
- Total Songs
- With PDFs
- With BPM
- With Key
```

**New Stats**:
```javascript
- Total Songs in SOU Database
- Songs with PDFs  
- Recently Added (last 30 days)
- Seed Database Size (read from expanded_seed_base.json)
```

**Quick Actions - REMOVE**:
```javascript
// Delete this entire section
<div className="quick-actions">
  <h2>Quick Actions</h2>
  <div className="action-buttons">
    <button onClick={() => setActivePage('search')}>Search Database</button>
    <button onClick={() => setActivePage('songs')}>Manage Songs</button>
    <button onClick={() => setActivePage('bulk')}>Bulk Import</button>
  </div>
</div>
```

**Changes Required**:
- [ ] Update stat calculations (remove BPM, Key)
- [ ] Add "Recently Added" stat (filter by date added)
- [ ] Add API endpoint to fetch seed database size
- [ ] Remove Quick Actions section entirely
- [ ] Update dashboard CSS for new layout

---

### 3.3 Seed Database Search Page (NEW)
**File**: `sou-song-browser/src/components/SeedDatabaseSearch.js` (NEW FILE)

**Purpose**: Search 47K seed songs and promote to SOU database

**Search Criteria**:
```javascript
- Artist (text input)
- Year Range (two number inputs: start/end)
- Genre (text input or dropdown with common genres)
- Popularity Score (range slider: 0-100)
```

**Search API**:
```javascript
POST ${API_URL}/api/seed/search
Body: {
  artist?: string,
  yearStart?: number,
  yearEnd?: number,
  genre?: string,
  popularityMin?: number,
  popularityMax?: number,
  limit: 100
}

Response: {
  songs: [
    {
      title: string,
      artist: string,
      spotifyId: string,
      releaseYear?: number,
      genres?: string[],
      popularity?: { spotify: number },
      source: string,
      discoveredDate: string
    }
  ],
  total: number
}
```

**Results Display**:
- Table with columns: ☑️ | Title | Artist | Year | Popularity | Genres | Source
- Checkbox for each row
- "Select All" / "Deselect All" buttons in header
- "Promote to SOU Database" button (disabled if none selected)

**Promotion Workflow**:
```javascript
1. User selects songs via checkboxes
2. Clicks "Promote to SOU Database"
3. Frontend calls: POST ${API_URL}/api/seed/promote
   Body: { spotifyIds: string[] }
4. Backend:
   a. Fetches full metadata for each song from seed database
   b. Runs enrichment pipeline (BPM, Key, additional APIs)
   c. Adds to songdb_master_v2_enriched.csv
   d. Regenerates songs_app_export_merged.json
5. Frontend shows success message with count
6. Selected songs cleared
```

**Changes Required**:
- [ ] Create SeedDatabaseSearch.js component
- [ ] Create SeedDatabaseSearch.css
- [ ] Create backend endpoint: `/api/seed/search`
- [ ] Create backend endpoint: `/api/seed/promote`
- [ ] Implement promotion/enrichment workflow in backend
- [ ] Add loading states and error handling

**Data Source**:
- Backend reads from: `materials-server/data/expanded_seed_base.json`
- Filter logic matches: title, artist, releaseYear, genres[], popularity.spotify

---

### 3.4 Bulk Export Page
**File**: `sou-song-browser/src/components/BulkExport.js` (Rename from BulkImport)

**Current**: BulkImport.js
**New**: BulkExport.js

**Purpose**: Export songs from seed database by artist/title

**Interface**:
```javascript
- Export By:
  - [ ] Artist Name(s) (textarea, comma-separated)
  - [ ] Song Title(s) (textarea, comma-separated)  
  - [ ] Artist + Title (two textareas)
  
- Format:
  - ( ) CSV
  - ( ) JSON
  
- [Export] button
```

**Export API**:
```javascript
POST ${API_URL}/api/seed/export
Body: {
  artists?: string[],
  titles?: string[],
  format: 'csv' | 'json'
}

Response: File download (CSV or JSON)
```

**Changes Required**:
- [ ] Rename BulkImport.js → BulkExport.js
- [ ] Update UI to focus on export from seed database
- [ ] Create backend endpoint: `/api/seed/export`
- [ ] Implement CSV/JSON export logic
- [ ] Update AdminDashboard.js to use BulkExport component

---

### 3.5 Manage SOU Database Page
**File**: `sou-song-browser/src/components/ManageSOUDatabase.js` (Rename from songs list)

**Current**: Embedded in AdminDashboard.js (renderSongsList)
**New**: Separate component ManageSOUDatabase.js

**Purpose**: Search, view, and edit SOU teaching database

**Interface** (Keep mostly as-is):
- Search input (filters by title, artist, genre)
- Sortable table columns
- Edit button → opens SongEditor modal
- Delete button (with confirmation)

**Field Edit Rules** (Document clearly in UI):

| Field | Source | Editable | Notes |
|-------|--------|----------|-------|
| Song Name | API | ❌ No | From Spotify/MusicBrainz |
| Artist | API | ❌ No | From Spotify/MusicBrainz |
| Year | API | ❌ No | From Spotify/MusicBrainz |
| Release Date | API | ✅ Yes | Can override API value |
| Season | API | ❌ No | Derived from release date |
| Duration | API | ❌ No | From Spotify |
| Popularity | API | ❌ No | From Spotify/Last.fm |
| YouTube Link | API | ❌ No | From YouTube API |
| Spotify Link | API | ❌ No | From Spotify |
| **Songsheet** | Primary | ✅ Yes | PDF link/path |
| **Major/Minor** | Override | ✅ Yes | Manual music theory |
| **Original Key** | Override | ✅ Yes | Manual music theory |
| **SOU Keys** | Primary | ✅ Yes | Teaching keys (array) |
| **Level** | Primary | ✅ Yes | Difficulty level |
| **TAB** | Primary | ✅ Yes | Has tablature? |
| **Teaching Notes** | Primary | ✅ Yes | Instructor notes |
| **Strum Style** | Primary | ✅ Yes | Strumming pattern |
| **Fingerpicking Style** | Primary | ✅ Yes | Fingerpicking pattern |
| **Time Signature** | Override | ✅ Yes | e.g., 4/4, 3/4 |
| **BPM** | Override | ✅ Yes | Beats per minute |
| **Tempo** | Override | ✅ Yes | Slow/Medium/Fast |
| **No. of Chords** | Override | ✅ Yes | Count |
| **Chords** | Override | ✅ Yes | Chord list |
| **Chord Numerals** | Override | ✅ Yes | Nashville numbers |
| **Genre** | Augment | ✅ Yes | Can add to API data |
| **Tags** | Augment | ✅ Yes | Can add to API data |
| **Songwriter** | Augment | ✅ Yes | Can add to API data |
| **PDF** | Primary | ❌ No | From archive, not editable |

**Changes Required**:
- [ ] Extract renderSongsList() to ManageSOUDatabase.js
- [ ] Update SongEditor.js to enforce edit rules (disable non-editable fields)
- [ ] Add tooltips/help text explaining edit rules
- [ ] Update AdminDashboard.js to import/use ManageSOUDatabase

---

### 3.6 SearchPage.js (REMOVE)
**File**: `sou-song-browser/src/components/SearchPage.js`

**Status**: Replaced by SeedDatabaseSearch.js

**Changes Required**:
- [ ] Delete SearchPage.js (after migrating any useful code)
- [ ] Delete SearchPage.css
- [ ] Remove import from AdminDashboard.js

---

### 3.7 PopularityCatalog.js (REMOVE)
**File**: `sou-song-browser/src/components/PopularityCatalog.js`

**Status**: Functionality merged into SeedDatabaseSearch.js

**Changes Required**:
- [ ] Move to archive folder (don't delete yet, may need code reference)
- [ ] Remove import from AdminDashboard.js
- [ ] Remove 'popularity' case from renderPage() switch

---

### 3.8 SongEditor.js
**File**: `sou-song-browser/src/components/SongEditor.js`

**Changes Required**:
- [ ] Add disabled state for non-editable fields
- [ ] Add visual indication (grayed out, lock icon) for non-editable fields
- [ ] Add tooltips explaining why certain fields can't be edited
- [ ] Update field groupings to match edit rules (API / Override / Augment / Primary)
- [ ] Validate that API fields aren't sent in save requests

---

## 4. BACKEND CHANGES

### 4.1 New Endpoints Required

#### GET /api/seed/size
**Purpose**: Return seed database statistics for dashboard

**Response**:
```json
{
  "total": 47272,
  "withGenres": 43500,
  "averagePopularity": 65,
  "lastUpdated": "2025-11-24T12:00:00Z"
}
```

#### POST /api/seed/search
**Purpose**: Search seed database by criteria

**Request**:
```json
{
  "artist": "Adele",
  "yearStart": 2010,
  "yearEnd": 2020,
  "genre": "pop",
  "popularityMin": 70,
  "popularityMax": 100,
  "limit": 100
}
```

**Response**:
```json
{
  "songs": [...],
  "total": 45,
  "took": 23
}
```

#### POST /api/seed/promote
**Purpose**: Promote songs from seed to SOU database with enrichment

**Request**:
```json
{
  "spotifyIds": ["3n3Ppam7vgaVa1iaRUc9Lp", "..."]
}
```

**Response**:
```json
{
  "success": true,
  "promoted": 5,
  "failed": 0,
  "errors": []
}
```

**Backend Process**:
1. Validate spotifyIds exist in seed database
2. For each song:
   - Fetch full metadata from seed JSON
   - Run enrichment scripts:
     - BPM detection
     - Key detection
     - Additional API calls (if needed)
   - Generate enriched record
   - Append to `Song Database/songdb_master_v2_enriched.csv`
3. Regenerate `songs_app_export_merged.json`
4. Return summary

#### POST /api/seed/export
**Purpose**: Export songs from seed database

**Request**:
```json
{
  "artists": ["The Beatles", "Adele"],
  "titles": ["Hello", "Let It Be"],
  "format": "csv"
}
```

**Response**: File download (CSV or JSON)

---

### 4.2 Backend File Structure

```
materials-server/
├── server.js (main app)
├── routes/
│   ├── seed.js (NEW - seed database routes)
│   ├── songs.js (existing - SOU database routes)
│   └── ...
├── services/
│   ├── seedService.js (NEW - seed database operations)
│   ├── enrichmentService.js (existing - enrichment pipeline)
│   └── ...
└── data/
    └── expanded_seed_base.json (seed database)
```

**New Files**:
- [ ] `routes/seed.js` - Express routes for seed database
- [ ] `services/seedService.js` - Seed database search/export logic
- [ ] Update `server.js` - Register seed routes

---

## 5. IMPLEMENTATION PHASES

### Phase 1: Backend Preparation (1-2 days)
- [ ] Create seed database endpoints (search, export, promote, size)
- [ ] Test seed search with various filters
- [ ] Implement promotion/enrichment workflow
- [ ] Test export (CSV/JSON)

### Phase 2: Component Restructuring (2-3 days)
- [ ] Update AdminLayout.js with new menu structure
- [ ] Create SeedDatabaseSearch.js component
- [ ] Rename/update BulkExport.js
- [ ] Extract ManageSOUDatabase.js from AdminDashboard
- [ ] Update Dashboard stats

### Phase 3: Integration & Testing (1-2 days)
- [ ] Connect components to new backend endpoints
- [ ] Test promotion workflow end-to-end
- [ ] Test export functionality
- [ ] Update SongEditor.js with edit rules
- [ ] Handle loading states and errors

### Phase 4: Cleanup & Documentation (1 day)
- [ ] Remove/archive old components (SearchPage, PopularityCatalog)
- [ ] Update CSS for new layouts
- [ ] Add tooltips and help text
- [ ] Update README with new structure
- [ ] Document API endpoints

**Total Estimated Time**: 5-8 days

---

## 6. TESTING CHECKLIST

### Seed Database Search
- [ ] Search by artist returns correct results
- [ ] Search by year range filters properly
- [ ] Search by genre matches correctly
- [ ] Popularity range filter works
- [ ] Combined filters work (AND logic)
- [ ] Select all / deselect all works
- [ ] Promotion workflow completes successfully
- [ ] Error handling for failed promotions

### Bulk Export
- [ ] Export by artist names (comma-separated)
- [ ] Export by titles (comma-separated)
- [ ] CSV export downloads correctly
- [ ] JSON export downloads correctly
- [ ] File contents are accurate

### Manage SOU Database
- [ ] Search filters work correctly
- [ ] Sorting works on all columns
- [ ] Edit opens SongEditor with correct data
- [ ] Non-editable fields are disabled
- [ ] Editable fields can be modified and saved
- [ ] Delete confirmation works

### Dashboard
- [ ] All stats display correct counts
- [ ] Seed database size loads from API
- [ ] Recently added calculation is correct
- [ ] No quick actions buttons present

### Navigation
- [ ] New menu structure displays correctly
- [ ] Sub-items expand/collapse properly
- [ ] Active state highlights correctly
- [ ] All pages load without errors

---

## 7. ROLLBACK PLAN

If issues arise, rollback strategy:

1. **Keep backup of old components**:
   - Move to `src/components/__admin_backup_YYYYMMDD/`
   - Already have: `__admin_backup_20251124/`

2. **Git branches**:
   - Create `feature/admin-reorganization` branch
   - Keep `main` branch stable
   - Merge only after thorough testing

3. **Feature flags**:
   - Add `REACT_APP_NEW_ADMIN_UI=true/false` env var
   - Toggle between old/new UI during transition

4. **Database backups**:
   - Auto-backup CSV before any writes
   - Keep last 7 days of backups

---

## 8. SUCCESS CRITERIA

✅ Reorganization is successful when:

1. **Clarity**: Users can clearly distinguish seed database (discovery) from SOU database (teaching)
2. **Workflow**: Song promotion from seed → SOU is intuitive and works correctly
3. **Performance**: All searches and operations complete in < 3 seconds
4. **Reliability**: No data loss or corruption during promotion/export
5. **Maintainability**: Code is well-organized, documented, and testable
6. **User Feedback**: Admin user confirms improved usability

---

## 9. OPEN QUESTIONS

1. **Chart Lookup**: Keep as separate menu item or integrate somewhere?
   - *Decision Pending*

2. **Authentication**: Any changes to auth flow needed?
   - *Keep as-is for now*

3. **Seed Database Updates**: How often is expanded_seed_base.json updated?
   - *Current: Manual imports from playlists*
   - *Future: Could add auto-refresh feature*

4. **Enrichment Pipeline**: Which enrichment scripts to run during promotion?
   - *Priority: BPM, Key detection*
   - *Optional: Additional API calls*

5. **CSV vs Database**: When to migrate from CSV to proper database?
   - *Future consideration, not part of this reorganization*

---

## 10. NOTES

- Genre enrichment for seed database running in background (separate process)
- React app location: `sou-song-browser/` subdirectory
- Backend location: `materials-server/`
- Two databases: 
  - Seed: `materials-server/data/expanded_seed_base.json` (47K songs)
  - SOU Teaching: `Song Database/songdb_master_v2_enriched.csv` → `songs_app_export_merged.json`
