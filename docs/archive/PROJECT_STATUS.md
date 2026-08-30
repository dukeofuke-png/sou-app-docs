# School of Uke - Project Status & Roadmap
**Last Updated:** November 19, 2025  
**Status:** Local development complete, ready for admin dashboard phase

---

## 🎯 Project Mission
Build a comprehensive ukulele teaching platform for School of Uke with:
- Public song browser for students
- Admin dashboard for content management  
- Tutor dashboard with teaching tools
- User accounts with progress tracking
- Dynamic song sheet editor

---

## 📊 Current State (Phase 1: COMPLETE ✅)

### What's Working Locally
- ✅ React frontend serving 212 songs (http://localhost:3000)
- ✅ Express materials server (http://localhost:3002)
- ✅ PDF serving from local Google Drive
- ✅ Sanitized data (no exposed file paths)
- ✅ Search, filters, song details modal
- ✅ Enriched metadata (BPM: 61%, genres, chart data, Spotify/YouTube links)

### Architecture
```
┌─────────────────────┐
│ React Frontend      │ → Static import of songs JSON
│ (Create React App)  │ → Filter/search/display
│ Port 3000           │ → Modal with song details
└──────────┬──────────┘
           │ HTTP requests for PDFs
           ↓
┌─────────────────────┐
│ Express Backend     │ → /health endpoint
│ (materials-server)  │ → /materials/* (PDF serving)
│ Port 3002           │ → /api/* (auth/CRUD - dormant)
└──────────┬──────────┘
           │ File system access
           ↓
┌─────────────────────┐
│ Google Drive        │ → Mounted locally
│ Song Sheets PDFs    │ → ~200 PDF files
└─────────────────────┘
```

### Data Pipeline (Offline)
```
Raw CSV (School of Uke Song Sheets Database.csv)
    ↓ Python enrichment scripts
Enriched CSV (songdb_master_v2_enriched.csv) + BPM_Best, metadata
    ↓ scripts/sanitize_paths.js
Sanitized JSON (songs_app_export_sanitized.json) - no absolute paths
    ↓ Static import
React App
```

### Key Files
```
SOU App/
├── materials-server/           # Backend API
│   ├── server.js              # Express app with Lambda support
│   ├── auth.js                # Password auth (ready, not active)
│   ├── csvManager.js          # CRUD for CSV (ready, not active)
│   └── package.json           # Dependencies include serverless-http
├── sou-song-browser/          # Frontend
│   ├── src/
│   │   ├── App.js             # Main component (212 songs)
│   │   ├── SongDetailModal.js # Song popup with PDF links
│   │   └── data/
│   │       ├── songs_app_export_sanitized.json  # Active dataset
│   │       └── songs_app_export_merged.json     # Legacy fallback
│   ├── public/index.html      # Updated title
│   └── package.json
├── scripts/
│   └── sanitize_paths.js      # Removes absolute Google Drive paths
├── Song Database/             # Enrichment scripts & data
│   ├── scripts/               # Python enrichment tools
│   ├── data/                  # CSVs, backups
│   └── *.md                   # Pipeline docs
├── README_AWS.md              # Deployment guide
└── .github/
    └── copilot-instructions.md  # AI agent guidance
```

### Tech Stack
- **Frontend:** React 18, Create React App
- **Backend:** Node 18+, Express, CORS
- **Data:** CSV → JSON pipeline
- **Auth (future):** bcryptjs, express-session
- **AWS (future):** S3, CloudFront, Lambda, API Gateway
- **PDFs (current):** Local Google Drive mount
- **PDFs (future):** S3 bucket

---

## 🎯 Deployment Timeline

### Week 1 (Nov 19-26): Admin Dashboard Priority
**Goal:** Password-protected admin for editing songs locally before AWS deployment

#### Day 1-2: Admin Login ✅ COMPLETE
- [x] Backend auth routes exist (dormant)
- [x] Create AdminLogin component
- [x] Session management
- [x] Single admin password (env var)

#### Day 3-4: Song Management UI ✅ COMPLETE
- [x] AdminDashboard component
- [x] Song list with edit/delete buttons
- [x] SongEditor form (all fields editable)
- [x] Add new song functionality

#### Day 5-6: Bulk Operations ✅ COMPLETE
- [x] Bulk song import (paste list of song names)
- [x] Auto-fetch metadata from APIs (Spotify, GetSongBPM, YouTube)
- [x] Database search engine (query Spotify for songs)
- [x] Unified advanced search (no tabs, all criteria, fuzzy chart parsing)
- [x] Source transparency badges (SP/MBZ/LFM/SC/DZ)
- [x] CSV write-back with versioning

**Advanced Search Features:**
- Single unified form (removed tabs)
- 10 search criteria: artist, tag, genre, year range, chart position, songwriter, season, key, mode
- Fuzzy chart parsing ("No.1", "Top 10", "#1" → precise ranges)
- Source preference routing (Auto, Spotify, MusicBrainz, Last.fm, etc.)
- Color-coded source badges in results
- See [UNIFIED_SEARCH_IMPLEMENTATION.md](./UNIFIED_SEARCH_IMPLEMENTATION.md)

#### Day 7: Testing & Polish
- [ ] Test all CRUD operations
- [ ] Test unified advanced search with all criteria combinations
- [ ] Verify source badges display correctly for all API sources
- [ ] Test fuzzy chart position parsing ("No.1", "Top 10", "#1", etc.)
- [ ] Ensure CSV updates work
- [ ] Test duplicate detection (future integration)
- [ ] Prepare for AWS deployment

### Week 2 (Nov 26-Dec 2): AWS Deployment
**Deadline:** Dec 2, 2025 (9 days from now)

#### Backend (Lambda + API Gateway)
- [ ] Run sanitization script
- [ ] Upload PDFs to S3 bucket
- [ ] Update songs JSON with S3 paths
- [ ] Deploy Lambda function
- [ ] Configure API Gateway
- [ ] Test /health, /materials, /api/songs

#### Frontend (S3 + CloudFront)
- [ ] Build React app
- [ ] Upload build/ to S3
- [ ] Create CloudFront distribution
- [ ] Point to Lambda backend
- [ ] Test end-to-end

#### Cost: ~$2-5/month
- S3: $0.50 (static files)
- CloudFront: $1-2 (CDN)
- Lambda: Free tier (1M requests/month)

---

## 🎨 Medium-Term Features (Month 2-3)

### User Accounts & Progress Tracking
- [ ] User registration/login
- [ ] Save favorite songs
- [ ] Track learned songs
- [ ] Practice history
- [ ] Difficulty recommendations

### Tutor Dashboard
- [ ] Music Theory Sheet Finder
- [ ] PDF archive browser
- [ ] Tutor account management
- [ ] Student progress view

### Data Enrichment (Background)
- [ ] Auto-enrich remaining 39% BPM
- [ ] Bulk fetch songs by criteria (e.g., "Top 10 2010-2020")
- [ ] Add missing Spotify/YouTube links
- [ ] Genre classification improvements

---

## 🚀 Long-Term Vision (Month 4+)

### Dynamic Song Sheet Editor
**Problem:** Currently using Keynote → static PDFs  
**Solution:** Digital, editable song sheets with PDF export

Features:
- Chord diagram builder
- Strumming pattern editor
- Lyrics + chord placement
- Key transposition
- PDF export for printing
- Version control

### Ukulele Sound Simulator
- Simulated uke sounds per strum pattern
- Play-along audio for song sheets
- Interactive practice mode
- Metronome integration

### Advanced Admin Features
- Bulk PDF uploads
- Song sheet template library
- Automated enrichment pipelines
- Analytics dashboard (most popular songs, etc.)

---

## 🔧 Technical Debt & Improvements

### Current Issues (Fixed ✅)
- ✅ React hooks eslint warning (wrapped songs in useMemo)
- ✅ Page title was "React App" (updated to "School of Uke")
- ✅ Duplicate server.js in frontend (removed)

### Future Refactors
- [ ] Move songs JSON to backend endpoint (not bundled in frontend)
- [ ] Add caching layer (Redis or in-memory)
- [ ] Implement proper error boundaries
- [ ] Add loading states
- [ ] Unit tests for components
- [ ] E2E tests for critical flows

---

## 📋 Admin Dashboard Requirements (Detailed)

### Must-Have (Week 1)
1. **Authentication**
   - Single password (you only)
   - Session persistence
   - Logout functionality

2. **Song CRUD**
   - View all 212 songs in table
   - Edit any field (Title, Artist, BPM, Key, Genre, etc.)
   - Add new song manually
   - Delete song (with confirmation)
   - Save changes to CSV

3. **Bulk Add**
   - Paste list of song names
   - Auto-fetch metadata (Spotify, MusicBrainz, etc.)
   - Review before adding to database
   - Add all with one click

4. **PDF Management**
   - Upload PDF for a song
   - Replace existing PDF
   - View current PDF
   - Mark PDF status (Yes/Draft/No)

### Nice-to-Have (Week 2+)
- Search/filter in admin table
- Undo last change
- Audit log (who changed what when)
- Backup/restore CSV
- Import/export JSON

---

## 🔐 Security Considerations

### Current (Local Only)
- No passwords stored in code
- Absolute paths sanitized from JSON
- CORS restricted to localhost

### For AWS Deployment
- [ ] Environment variables for secrets
- [ ] HTTPS only (CloudFront)
- [ ] API Gateway throttling
- [ ] JWT tokens for user sessions
- [ ] S3 bucket private (signed URLs for PDFs)
- [ ] Input validation on all forms
- [ ] SQL injection prevention (if moving to DB)

---

## 📦 Deployment Checklist (Week 2)

### Pre-Deployment
- [ ] Run `node scripts/sanitize_paths.js`
- [ ] Verify all 212 songs load in admin
- [ ] Test CRUD operations
- [ ] Export final songs JSON
- [ ] Collect all PDFs for S3 upload

### Backend Deployment
- [ ] Create S3 bucket for PDFs (private)
- [ ] Upload PDFs with consistent naming
- [ ] Update songs JSON with S3 keys
- [ ] Zip Lambda function
- [ ] Create Lambda in AWS Console
- [ ] Set environment variables (NODE_ENV, FRONTEND_URL)
- [ ] Create API Gateway REST API
- [ ] Map routes (/health, /materials, /api/*)
- [ ] Enable CORS
- [ ] Deploy stage
- [ ] Test invoke URL

### Frontend Deployment
- [ ] Update `.env.production` with API Gateway URL
- [ ] Build: `npm run build`
- [ ] Create S3 bucket for static site
- [ ] Upload build/ folder
- [ ] Enable static website hosting
- [ ] Create CloudFront distribution
- [ ] Point origin to S3
- [ ] Map 404 → index.html (SPA routing)
- [ ] Test CloudFront URL
- [ ] (Optional) Add custom domain + SSL cert

### Post-Deployment
- [ ] Verify song list loads
- [ ] Test PDF links
- [ ] Test admin login
- [ ] Test CRUD operations
- [ ] Monitor CloudWatch logs
- [ ] Set up billing alerts

---

## 🆘 Disaster Recovery

### If Everything Crashes
This document + `.github/copilot-instructions.md` contain full context.

**Quick Recovery Steps:**
1. Read this file top-to-bottom
2. Check `materials-server/server.js` (main backend)
3. Check `sou-song-browser/src/App.js` (main frontend)
4. Check `songs_app_export_sanitized.json` (data)
5. Run local: `materials-server: npm start` + `sou-song-browser: npm start`
6. Admin code is in `/src/components/Admin*` (when built)

**Key Commands:**
```bash
# Sanitize data
node scripts/sanitize_paths.js

# Start backend (port 3002)
cd materials-server && npm start

# Start frontend (port 3000)
cd sou-song-browser && npm start

# Test health
curl http://localhost:3002/health

# Open browser
open http://localhost:3000
```

### Backup Strategy (Future)
- [ ] Daily CSV backup to S3
- [ ] Weekly full database snapshot
- [ ] Git commits for all code changes
- [ ] Separate staging environment

---

## 📞 Questions for Future Development

### Phase 2 (User Accounts)
- Separate tutor vs. student login?
- Social login (Google/Facebook)?
- Payment integration for premium features?
- Age verification for students?

### Phase 3 (Song Sheet Editor)
- Desktop app or web-based?
- Collaborative editing?
- Template marketplace?
- Mobile app?

### Phase 4 (Sound Simulator)
- Real instrument samples vs. synthesized?
- MIDI support?
- Record user playing?
- AI-powered feedback?

---

## 🎓 Learning Resources

### For You (Beginner)
- AWS Free Tier: https://aws.amazon.com/free/
- React Docs: https://react.dev/
- Express Docs: https://expressjs.com/
- AWS Lambda Tutorial: https://aws.amazon.com/lambda/getting-started/

### Recommended Next Steps
1. Complete admin dashboard locally
2. Watch AWS S3 + CloudFront tutorial
3. Practice deploying a simple Lambda function
4. Read README_AWS.md in this repo

---

## 📝 Notes

- **BPM Source:** GetSongBPM API (130/212 songs enriched)
- **PDF Count:** ~200 song sheets in Google Drive
- **User Count:** 0 (not public yet)
- **Admin Count:** 1 (you)
- **Last Deploy:** Never (local only)
- **Git Repo:** dukeofuke-png/sou-song-browser

---

## ✅ Completion Criteria

### Phase 1 (Complete ✅)
- [x] Local frontend working
- [x] Local backend working
- [x] PDFs loading
- [x] Data sanitized

### Phase 2 (In Progress - Week 1)
- [ ] Admin login works
- [ ] Can edit any song
- [ ] Can add new songs
- [ ] Can delete songs
- [ ] Bulk import works

### Phase 3 (Week 2)
- [ ] Live on AWS
- [ ] PDFs on S3
- [ ] Admin accessible online
- [ ] Public song browser online

### Phase 4 (Future)
- [ ] User accounts
- [ ] Progress tracking
- [ ] Tutor dashboard
- [ ] Song sheet editor
- [ ] Sound simulator

---

**END OF DOSSIER**

*Copy this entire file to Copilot if the project crashes. It contains everything needed to understand the current state and continue development.*
