# Admin Dashboard Testing Plan

**Date:** November 30, 2025  
**Status:** Ready for Testing  
**Backend:** http://localhost:3002 ✅ (218 songs)  
**Frontend:** http://localhost:3000 ✅  
**Admin:** http://localhost:3000/admin

---

## 🎯 Testing Objectives

1. ✅ Verify all CRUD operations work correctly
2. ✅ Test inline, modal, and bulk editing
3. ✅ Test Bulk Add Songs feature (parallel search, Enter key, checkboxes)
4. ✅ Verify CSV writes are happening correctly
5. ✅ Test PDF links work in song details
6. ✅ Find and document any bugs
7. ✅ Verify field validation works

---

## 📋 Test Checklist

### 1. Basic Access & Loading
- [ ] Navigate to http://localhost:3000/admin
- [ ] Admin dashboard loads without errors
- [ ] Dashboard shows correct song count: **218 songs**
- [ ] Dashboard shows "With PDFs" count
- [ ] Dashboard shows "Recently Added" count
- [ ] Dashboard shows "Seed Database" count (~47K songs)
- [ ] No console errors in browser DevTools

**Expected Results:**
- Clean load with all stats displayed
- No 404 or connection errors
- Stats match backend data

---

### 2. Manage SOU Database - View & Navigation
- [ ] Click "Manage SOU Database" in sidebar
- [ ] Page loads with song table
- [ ] All 218 songs display in table
- [ ] Columns visible: #, Title, Artist, BPM, Key, Genre, PDF, Actions
- [ ] Table is scrollable
- [ ] Footer shows "Showing 218 of 218 songs"

**Expected Results:**
- All songs load correctly
- Data displays properly in all columns
- No missing or malformed data

---

### 3. Inline Editing (Double-Click Cell)

**Test Case 3.1: Edit Level Field**
- [ ] Find a song (e.g., first song in table)
- [ ] Note current Level value
- [ ] Double-click the Level cell
- [ ] Cell becomes editable input field
- [ ] Change value (e.g., "Beginner" → "Intermediate")
- [ ] Press Enter
- [ ] Success message appears: "Song updated successfully!"
- [ ] Cell shows new value
- [ ] Page does NOT reload

**Test Case 3.2: Edit Status Field**
- [ ] Double-click a Status cell
- [ ] Change value (e.g., "Active" → "Draft")
- [ ] Press Enter
- [ ] Verify success message
- [ ] Verify new value persists

**Test Case 3.3: Edit Non-Editable Field (Should Fail)**
- [ ] Try double-clicking Title or Artist
- [ ] Cell should NOT become editable
- [ ] No edit input appears

**Expected Results:**
- Only editable fields respond to double-click
- Changes save immediately
- Success toast appears
- Backend logs show update
- CSV file is updated (verify timestamp changed)

**Potential Issues to Check:**
- ❌ "Failed to save changes" error
- ❌ Old numeric IDs being sent (song_0211)
- ❌ Cell not becoming editable
- ❌ Changes not persisting after page refresh

---

### 4. Modal Editing (Click Row)

**Test Case 4.1: Open Modal**
- [ ] Click any song row in the table
- [ ] Modal opens with song details
- [ ] All fields are visible
- [ ] Current values are displayed correctly
- [ ] Modal has "Save" and "Cancel" buttons

**Test Case 4.2: Edit Multiple Fields**
- [ ] Open modal for a song
- [ ] Change 3-5 fields:
  - Level: "Beginner" → "Intermediate"
  - Status: "Active" → "Draft"
  - Key (SOU): "C" → "G"
  - Strum Style: Change to different value
  - Notes: Add some test text
- [ ] Click "Save"
- [ ] Modal closes
- [ ] Success message appears
- [ ] Song table updates with new values

**Test Case 4.3: Cancel Editing**
- [ ] Open modal
- [ ] Change several fields
- [ ] Click "Cancel"
- [ ] Modal closes
- [ ] Changes are NOT saved
- [ ] Original values remain in table

**Test Case 4.4: Validation**
- [ ] Open modal
- [ ] Try to clear required field (Title or Artist)
- [ ] Try to save
- [ ] Should show validation error

**Expected Results:**
- Modal displays all song data
- Multiple field edits save correctly
- Cancel discards changes
- Validation prevents invalid data

---

### 5. Bulk Editing (Select Multiple Songs)

**Test Case 5.1: Select Songs**
- [ ] Click checkboxes for 3-5 songs
- [ ] Selected count updates
- [ ] "Bulk Edit" button appears/becomes enabled

**Test Case 5.2: Bulk Update Single Field**
- [ ] Select 5 songs
- [ ] Click "Bulk Edit" button
- [ ] Bulk edit modal opens
- [ ] Change one field (e.g., Level → "Intermediate")
- [ ] Click "Apply"
- [ ] All 5 songs update
- [ ] Success message shows count: "Updated 5 songs"

**Test Case 5.3: Bulk Update Multiple Fields**
- [ ] Select 3 songs
- [ ] Open bulk edit modal
- [ ] Change:
  - Level: "Advanced"
  - Status: "Active"
  - Tags: "test, bulk-edit"
- [ ] Click "Apply"
- [ ] All 3 songs update with all changes
- [ ] Verify in table

**Test Case 5.4: Validation in Bulk Edit**
- [ ] Try to set invalid data in bulk edit
- [ ] Should show validation error
- [ ] Should not update if validation fails

**Expected Results:**
- Bulk updates work for multiple songs
- All selected songs receive same changes
- Non-editable fields are not available in bulk edit
- Success message shows correct count

**Potential Issues:**
- ❌ Only some songs update (partial failure)
- ❌ Wrong songs get updated
- ❌ Non-editable fields get changed

---

### 6. Bulk Add Songs Feature

**Test Case 6.1: Single-Term Search (Parallel)**
- [ ] Click "Bulk Add Songs" in sidebar
- [ ] Type "Prince" (single term, no separator)
- [ ] Press Enter key
- [ ] Search executes (loading indicator)
- [ ] Results show songs by artist "Prince"
- [ ] Results show songs with "Prince" in title
- [ ] Results have checkboxes
- [ ] Results show: Title, Artist, Year, Popularity, Genres
- [ ] No duplicate results (same spotifyId)

**Test Case 6.2: Multi-Term Search (Artist | Title)**
- [ ] Clear previous search
- [ ] Type:
   ```
   Adele | Rolling in the Deep
   Beatles | Let It Be
   Queen | Bohemian Rhapsody
   ```
- [ ] Click "Find Matches" (or press Enter)
- [ ] Results show 3 songs (or more if multiple matches)
- [ ] Each result has correct artist/title pairing

**Test Case 6.3: Mixed Search Terms**
- [ ] Enter both single terms and artist|title pairs:
   ```
   Madonna
   Fleetwood Mac | Dreams
   Coldplay
   ```
- [ ] Search executes
- [ ] Results combine both search types
- [ ] All results have checkboxes

**Test Case 6.4: Checkbox Selection**
- [ ] Search for "Beatles"
- [ ] Click individual song checkboxes
- [ ] Selected songs highlight
- [ ] Click "Select All" checkbox in header
- [ ] All visible results become selected
- [ ] Click "Select All" again
- [ ] All results deselect

**Test Case 6.5: Add Selected Songs**
- [ ] Search and select 3 songs
- [ ] Button shows "Add 3 to SOU Database"
- [ ] Click button
- [ ] Songs are added
- [ ] Success message appears
- [ ] Can verify songs now appear in Manage SOU Database

**Test Case 6.6: Enter Key Functionality**
- [ ] Clear search box
- [ ] Type search term
- [ ] Press Enter (don't click button)
- [ ] Search executes
- [ ] Results appear

**Test Case 6.7: Metadata Display**
- [ ] Search for any artist
- [ ] Verify results show:
  - ✅ Title (bold)
  - ✅ Artist
  - ✅ Release Year (or "—")
  - ✅ Popularity badge (orange, shows %)
  - ✅ Genre tags (max 3 visible, "+N more")

**Expected Results:**
- Single terms search BOTH artist AND title
- Enter key triggers search
- Results have all metadata
- Checkboxes work for selection
- Add button shows correct count
- Songs successfully import to database

**Potential Issues:**
- ❌ Single terms only search title (not artist)
- ❌ Enter key doesn't trigger search
- ❌ Checkboxes missing or not working
- ❌ Metadata missing (year, popularity, genres)
- ❌ Duplicate songs in results

---

### 7. Search & Filtering (Manage SOU Database)

**Test Case 7.1: Search Box**
- [ ] Type in search box: "beatles"
- [ ] Table filters to matching songs
- [ ] Shows songs with "beatles" in title, artist, or genre
- [ ] Footer updates: "Showing X of 218 songs"

**Test Case 7.2: Clear Search**
- [ ] Clear search box
- [ ] All 218 songs return
- [ ] Footer shows "Showing 218 of 218 songs"

**Test Case 7.3: No Results**
- [ ] Search for nonsense: "zzzzzz"
- [ ] Table shows no results
- [ ] Footer shows "Showing 0 of 218 songs"
- [ ] No errors

**Expected Results:**
- Search filters correctly
- Case-insensitive
- Searches multiple fields
- Performance is good (instant filtering)

---

### 8. Sorting

**Test Case 8.1: Sort by Title**
- [ ] Click "Title" column header
- [ ] Songs sort A→Z
- [ ] Arrow indicator shows ↑
- [ ] Click again
- [ ] Songs sort Z→A
- [ ] Arrow indicator shows ↓

**Test Case 8.2: Sort Other Columns**
- [ ] Sort by Artist
- [ ] Sort by BPM
- [ ] Sort by Genre
- [ ] Each sorts correctly

**Expected Results:**
- Sorting works for all columns
- Direction indicator shows
- Multiple clicks toggle direction

---

### 9. PDF Links (Song Detail Modal)

**Test Case 9.1: Open Song with PDFs**
- [ ] Navigate to main app: http://localhost:3000
- [ ] Click any song in the catalog
- [ ] Song detail modal opens
- [ ] Scroll to "Available Materials" section
- [ ] PDF links are visible
- [ ] Links have format: "📄 Song Sheet in Key X" or "🎵 Melody TAB in Key X"

**Test Case 9.2: Click PDF Link**
- [ ] Click a song sheet PDF link
- [ ] New tab/window opens
- [ ] PDF loads and displays
- [ ] URL format: http://localhost:3002/materials/[folder]/[file].pdf

**Test Case 9.3: Click TAB Link**
- [ ] Click a melody TAB link
- [ ] PDF loads correctly
- [ ] Shows tablature notation

**Test Case 9.4: Song Without PDFs**
- [ ] Find a song without PDF materials
- [ ] Open song detail
- [ ] "Available Materials" section shows: "📋 No learning materials available yet"

**Expected Results:**
- PDF links display with correct names
- PDFs load in browser
- No 404 errors
- Correct files open for each song

**Potential Issues:**
- ❌ PDF links return 404
- ❌ Wrong PDF opens
- ❌ Link format wrong (port 3001 instead of 3002)

---

### 10. CSV Data Persistence

**Test Case 10.1: Verify Edits Write to CSV**
- [ ] Note current timestamp of CSV file:
  ```bash
  ls -la "/Users/matthew/Documents/SOU App/Song Database/data/songdb_master_v2_enriched.csv"
  ```
- [ ] Make an edit (inline or modal)
- [ ] Check CSV timestamp again
- [ ] Timestamp should be newer
- [ ] Check CSV backup created:
  ```bash
  ls -la "/Users/matthew/Documents/SOU App/Song Database/data/backups/"
  ```
- [ ] Latest backup should exist

**Test Case 10.2: Verify Edit Persists After Restart**
- [ ] Edit a song (change a unique value)
- [ ] Note the change
- [ ] Restart backend server:
  ```bash
  lsof -ti:3002 | xargs kill -9
  cd materials-server && npm start &
  ```
- [ ] Refresh admin page
- [ ] Verify edit is still there

**Test Case 10.3: Verify CSV Field Mapping**
- [ ] Open CSV file in editor
- [ ] Find the edited song row
- [ ] Verify fields match:
  - Frontend "level" → CSV "Level"
  - Frontend "status" → CSV "Status"
  - Frontend "souKeys" → CSV "Key (SOU)"
  - etc.

**Expected Results:**
- CSV updates on every edit
- Backup created automatically
- Changes persist after restart
- Field mapping is correct
- All 100+ CSV columns preserved (not just mapped ones)

---

### 11. Error Handling

**Test Case 11.1: Backend Down**
- [ ] Stop backend server:
  ```bash
  lsof -ti:3002 | xargs kill -9
  ```
- [ ] Try to edit a song in admin
- [ ] Should show error message
- [ ] Restart backend:
  ```bash
  cd materials-server && npm start &
  ```
- [ ] Verify admin recovers

**Test Case 11.2: Invalid Data**
- [ ] Try to save song with missing Title
- [ ] Should show validation error
- [ ] Try to set BPM to "abc"
- [ ] Should show validation error or reject input

**Test Case 11.3: Network Errors**
- [ ] Edit a field
- [ ] Check browser console for errors
- [ ] Check backend terminal for errors
- [ ] Should see appropriate error messages if any

**Expected Results:**
- Graceful error messages (not crashes)
- User-friendly error text
- Errors logged in console/backend
- App recovers after errors

---

### 12. Performance

**Test Case 12.1: Load Time**
- [ ] Measure page load time (Network tab in DevTools)
- [ ] Admin dashboard should load in < 2 seconds
- [ ] Song table should load in < 3 seconds

**Test Case 12.2: Search Performance**
- [ ] Type quickly in search box
- [ ] Filtering should be instant (< 100ms)
- [ ] No lag or freezing

**Test Case 12.3: Large Operations**
- [ ] Select 50 songs
- [ ] Bulk edit them
- [ ] Should complete in < 5 seconds

**Expected Results:**
- Fast load times
- Responsive interactions
- No freezing or lag

---

## 🐛 Bug Tracking

### Known Issues (Fixed)
✅ Admin dashboard showing 0 songs → Fixed with .env port configuration  
✅ PDF buttons not working → Fixed with correct materials URL  
✅ "Song not found" errors → Fixed by loading from API with slug IDs  
✅ Port conflicts → Fixed with auto-cleanup in start scripts

### Bugs Found During Testing

**Bug #1: [Title]**
- **Severity:** High/Medium/Low
- **Steps to Reproduce:**
  1. 
  2. 
  3. 
- **Expected:** 
- **Actual:** 
- **Error Message:** 
- **Status:** Open/Fixed

**Bug #2: [Title]**
- **Severity:** 
- **Steps to Reproduce:**
- **Expected:** 
- **Actual:** 
- **Status:** 

---

## ✅ Test Results Summary

### Completed Tests: 0/12 sections

**Status by Feature:**
- [ ] Basic Access & Loading
- [ ] View & Navigation
- [ ] Inline Editing
- [ ] Modal Editing
- [ ] Bulk Editing
- [ ] Bulk Add Songs
- [ ] Search & Filtering
- [ ] Sorting
- [ ] PDF Links
- [ ] CSV Persistence
- [ ] Error Handling
- [ ] Performance

### Critical Issues: 0
### Major Issues: 0
### Minor Issues: 0

---

## 📊 Test Coverage

### Backend Endpoints Tested
- [ ] GET /api/songs (load all songs)
- [ ] PUT /api/songs/:id (update song)
- [ ] POST /api/songs (create song)
- [ ] DELETE /api/songs/:id (delete song)
- [ ] POST /api/songs/bulk-update (bulk edit)
- [ ] POST /api/seed/search (search seed database)
- [ ] GET /materials/* (PDF serving)

### Frontend Components Tested
- [ ] AdminDashboard (main page)
- [ ] ManageSOUDatabase (song table)
- [ ] BulkAddSongs (search & import)
- [ ] SongDetailModal (PDF links)
- [ ] AdminLayout (navigation)

---

## 🔧 Manual Test Commands

**Check Backend Health:**
```bash
curl http://localhost:3002/health
```

**Get Song Count:**
```bash
curl -s http://localhost:3002/api/songs | jq 'length'
```

**Test PDF Link:**
```bash
# Replace with actual path from a song
curl -I "http://localhost:3002/materials/[folder]/[file].pdf"
```

**Check CSV Timestamp:**
```bash
ls -la "/Users/matthew/Documents/SOU App/Song Database/data/songdb_master_v2_enriched.csv"
```

**Check Backups:**
```bash
ls -lt "/Users/matthew/Documents/SOU App/Song Database/data/backups/" | head -5
```

**View Backend Logs:**
```bash
tail -f logs/backend.log
```

**View Frontend Logs:**
```bash
tail -f logs/frontend.log
```

---

## 📝 Testing Notes

**Start Here:**
1. Open http://localhost:3000/admin in browser
2. Open browser DevTools (F12)
3. Open backend terminal to watch logs
4. Work through each test section systematically
5. Document any bugs found
6. Mark sections complete with ✅

**Tips:**
- Test one feature at a time
- Note any console errors
- Watch backend logs during tests
- Save screenshots of any bugs
- Test edge cases (empty values, very long text, etc.)
- Try to break things!

**After Testing:**
- Update this document with results
- Create issues for bugs found
- Prioritize fixes
- Retest after fixes applied

---

## 🎯 Next Steps After Testing

1. **If All Tests Pass:**
   - Update PROJECT_STATUS.md (Week 1 complete ✅)
   - Prepare for AWS deployment
   - Document any UX improvements needed

2. **If Bugs Found:**
   - Prioritize by severity
   - Fix critical bugs first
   - Retest fixed issues
   - Update this document

3. **Future Testing:**
   - Add automated tests (Jest, React Testing Library)
   - Create E2E tests (Playwright, Cypress)
   - Set up CI/CD pipeline
   - Add test coverage metrics

---

**Happy Testing! 🧪**
