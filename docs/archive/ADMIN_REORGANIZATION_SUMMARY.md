# Admin Frontend Reorganization - Quick Reference

## 🎯 Goal
Separate seed database (discovery) from SOU database (teaching) with clear workflows

---

## 📊 Menu Structure

### BEFORE → AFTER

```
BEFORE (7 items):                    AFTER (4 items):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🏠 Dashboard                          🏠 Dashboard
🔍 Search Database                    🔍 Song Discovery
📈 Chart Lookup                            ├─ Seed Database Search  
⭐ Popularity Catalog                      └─ Bulk Export
🎵 Manage Songs                       🎵 Manage SOU Database
📤 Bulk Import                        ⚙️  Settings
⚙️  Settings                          
```

---

## 📋 Page Changes

### 1. Dashboard
**Remove**: BPM stat, Key stat, Quick Action buttons (redundant)  
**Add**: Recently Added stat, Seed Database Size stat  
**Keep**: Total Songs, With PDFs

### 2. Seed Database Search (NEW)
**Purpose**: Search 47K seed songs → promote to SOU database  
**Search**: Artist, Year Range, Genre, Popularity  
**Result**: Table with checkboxes → "Promote to SOU Database" button  
**Workflow**: Selected songs → auto-enriched → added to teaching database

### 3. Bulk Export (Renamed from Bulk Import)
**Purpose**: Export songs from seed database  
**Export By**: Artist names, Titles, or combination  
**Format**: CSV or JSON download

### 4. Manage SOU Database (Extracted from AdminDashboard)
**Purpose**: Search and edit SOU teaching database  
**Keep**: Current functionality (search, sort, edit, delete)  
**Update**: Enforce field edit rules (API/Override/Augment/Primary)

### 5. SearchPage (REMOVE)
**Replaced by**: SeedDatabaseSearch.js

### 6. PopularityCatalog (REMOVE)
**Merged into**: SeedDatabaseSearch.js

---

## 🔧 Technical Changes

### New Components
```
src/components/
├── SeedDatabaseSearch.js       (NEW - search 47K seed songs)
├── BulkExport.js              (RENAME from BulkImport.js)
└── ManageSOUDatabase.js       (EXTRACT from AdminDashboard.js)
```

### Updated Components
```
src/components/
├── AdminLayout.js             (UPDATE - new menu with sub-items)
├── AdminDashboard.js          (UPDATE - new dashboard stats)
└── SongEditor.js              (UPDATE - enforce edit rules)
```

### Removed Components
```
src/components/
├── SearchPage.js              (DELETE - replaced by SeedDatabaseSearch)
└── PopularityCatalog.js       (ARCHIVE - merged into SeedDatabaseSearch)
```

### New Backend Endpoints
```
GET  /api/seed/size            - Seed database statistics
POST /api/seed/search          - Search seed database
POST /api/seed/promote         - Promote songs to SOU + enrich
POST /api/seed/export          - Export seed songs to CSV/JSON
```

---

## 🔐 Field Edit Rules (Manage SOU Database)

| Field Type | Examples | Editable? |
|-----------|----------|-----------|
| **API** | Song Name, Artist, Year, Duration, Popularity | ❌ No |
| **API (Override)** | Release Date | ✅ Yes |
| **Override** | Key, BPM, Tempo, Chords, Time Signature | ✅ Yes |
| **Augment** | Genre, Tags, Songwriter | ✅ Yes (add to API data) |
| **Primary** | Songsheet, SOU Keys, Level, Teaching Notes | ✅ Yes |

---

## 📅 Implementation Phases

| Phase | Tasks | Duration |
|-------|-------|----------|
| **1. Backend** | Create seed endpoints, test search/export/promote | 1-2 days |
| **2. Components** | Update menu, create new components, extract pages | 2-3 days |
| **3. Integration** | Connect to APIs, test workflows, handle errors | 1-2 days |
| **4. Cleanup** | Remove old code, update docs, polish UI | 1 day |
| **Total** | | **5-8 days** |

---

## ✅ Success Criteria

- [x] Clear separation between seed (discovery) and SOU (teaching) databases
- [x] Intuitive promotion workflow from seed → SOU
- [x] Search and export work correctly for both databases
- [x] Field edit rules enforced in UI
- [x] No data loss or corruption
- [x] Improved admin usability

---

## 🗂️ Documentation

- **Vision**: `Song Database/ADMIN_INTERFACE_VISION.md`
- **Implementation Plan**: `ADMIN_REORGANIZATION_PLAN.md` (detailed)
- **This Summary**: `ADMIN_REORGANIZATION_SUMMARY.md` (quick reference)

---

## 📌 Key Decisions

| Question | Decision |
|----------|----------|
| Chart Lookup? | Keep separate (not part of reorganization) |
| Authentication? | No changes needed |
| Seed Database Updates? | Manual imports (current), auto-refresh (future) |
| Enrichment During Promotion? | Priority: BPM, Key detection |
| CSV vs Database Migration? | Future consideration (not now) |

---

## 🔄 Two Databases Explained

### Seed Database (47K songs)
- **File**: `materials-server/data/expanded_seed_base.json`
- **Purpose**: Discovery - curated playlists from Spotify
- **Fields**: title, artist, spotifyId, releaseYear, genres, popularity.spotify
- **Use Case**: Search and promote candidates to teaching database

### SOU Teaching Database (~3K songs)
- **File**: `Song Database/songdb_master_v2_enriched.csv` → `songs_app_export_merged.json`
- **Purpose**: Teaching materials with PDFs, keys, chords, BPM
- **Fields**: All seed fields PLUS BPM, Key, Chords, Teaching Notes, PDF links, etc.
- **Use Case**: Actual lessons and student materials

---

## 🚀 Next Steps

1. Review this plan with stakeholders
2. Create feature branch: `feature/admin-reorganization`
3. Start Phase 1: Backend endpoints
4. Iterative testing throughout implementation
5. Final review and merge to main

---

**Last Updated**: November 24, 2025  
**Status**: Planning Complete, Ready for Implementation
