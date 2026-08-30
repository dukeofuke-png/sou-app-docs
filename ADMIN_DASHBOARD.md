# Admin Dashboard Guide

## Overview
The School of Uke admin dashboard provides a password-protected interface for managing the song database. It includes full CRUD operations, bulk import, and direct CSV editing.

## Accessing the Admin Dashboard

### Local Development
1. Start the backend server: `cd materials-server && PORT=3002 node server.js`
2. Start the frontend: `cd sou-song-browser && npm start`
3. Navigate to: **http://localhost:3000/admin**

### Production
Navigate to: **https://your-domain.com/admin**

## Default Login

**Password:** `admin123`

⚠️ **IMPORTANT:** Change this password before deploying to production by setting the `ADMIN_PASSWORD` environment variable in `materials-server/.env`

## Features

### 1. Song Management
- **View All Songs**: Sortable table with 212 songs
- **Search**: Filter by title, artist, or genre
- **Sort**: Click any column header to sort
- **Edit**: Click the ✏️ icon to edit a song
- **Delete**: Click the 🗑️ icon to delete (with confirmation)

### 2. Add New Song
Click **"+ Add New Song"** button to:
- Enter song details manually
- Set BPM, Key, Genre, etc.
- Mark PDF availability status
- Add Spotify/YouTube IDs

### 3. Bulk Import
Click **"📋 Bulk Import"** to:
1. Paste a list of songs (one per line)
2. Format: "Title - Artist" or just "Title"
3. Review parsed results
4. Select which songs to import
5. Click "Import X Songs"

**Example input:**
```
Ain't No Sunshine - Bill Withers
Blue Hawaii - Elvis Presley
Somewhere Over The Rainbow
```

### 4. Song Editor
Edit any field:
- **Basic Info**: Title, Artist, Release Year, Level
- **Musical Properties**: BPM, Key, Mode, Time Signature
- **Classification**: Genre, Tags, Era, Season
- **Chart Data**: Position, Year
- **External Links**: Spotify ID, YouTube ID
- **Sheet Music**: Song Sheet Status, Melody Tab Status

### 5. Data Persistence
All changes are saved directly to:
`Song Database/data/songdb_master_v2_enriched.csv`

After editing, regenerate the JSON:
```bash
node scripts/sanitize_paths.js
```

## Field Reference

### Required Fields
- **Title** - Song title
- **Artist** - Primary artist

### Optional Fields
- **BPM_Best** - Tempo (numeric)
- **Key_Best** - Musical key (e.g., C, Am, F#)
- **Mode** - Major or Minor
- **TimeSignature** - e.g., 4/4, 3/4, 6/8
- **Genre** - Comma-separated genres
- **Tags** - Comma-separated tags
- **Level** - Beginner, Intermediate, or Advanced
- **Era** - 1950s, 1960s, ..., 2020s
- **Season** - Spring, Summer, Fall, Winter, Christmas
- **releaseYear** - Year of release
- **chartPosition** - Peak chart position
- **chartYear** - Year of chart peak
- **spotifyId** - Spotify track ID
- **youtubeId** - YouTube video ID
- **songSheetStatus** - Yes, Draft, or No
- **melodyTabStatus** - Yes, Draft, or No

## Security

### Session Management
- Sessions last 24 hours
- Secure cookies in production
- SameSite: lax

### CORS
Development: `http://localhost:3000`
Production: Set `FRONTEND_URL` in `.env`

### Password Storage
- Passwords are hashed with bcrypt (10 rounds)
- Never stored in plaintext
- Change default password immediately

## Troubleshooting

### "Failed to connect to server"
1. Check backend is running: `lsof -i :3002`
2. Check health endpoint: `curl http://localhost:3002/health`
3. Verify CORS settings in `materials-server/server.js`

### "Invalid password"
1. Check `ADMIN_PASSWORD` in `materials-server/.env`
2. Restart backend after changing password
3. Clear browser cookies

### "Failed to load songs"
1. Check CSV path: `Song Database/data/songdb_master_v2_enriched.csv`
2. Verify file permissions
3. Check backend logs for errors

### Changes not appearing in frontend
1. Regenerate JSON: `node scripts/sanitize_paths.js`
2. Refresh browser (hard refresh: Cmd+Shift+R)
3. Check React is using correct JSON file

## API Endpoints

### Authentication
- `POST /api/auth/login` - Login with password
- `POST /api/auth/logout` - Logout
- `GET /api/auth/check` - Check authentication status

### Songs (All require authentication)
- `GET /api/songs` - Get all songs
- `GET /api/songs/:id` - Get specific song
- `POST /api/songs` - Create new song
- `PUT /api/songs/:id` - Update song
- `DELETE /api/songs/:id` - Delete song

### Health
- `GET /health` - Server health check

## Next Steps

### Week 1 (Current)
- [x] Admin login
- [x] Song CRUD
- [x] Bulk import
- [ ] PDF upload
- [ ] Test all operations

### Week 2 (Deployment)
- [ ] Deploy backend to AWS Lambda
- [ ] Deploy frontend to S3/CloudFront
- [ ] Move PDFs to S3
- [ ] Test production admin

## Environment Variables

### Backend (`materials-server/.env`)
```bash
PORT=3002
ADMIN_PASSWORD=your-secure-password
SESSION_SECRET=your-session-secret
FRONTEND_URL=http://localhost:3000
MATERIALS_PATH=/path/to/pdfs
```

### Frontend (`sou-song-browser/.env.local`)
```bash
REACT_APP_MATERIALS_URL=http://localhost:3002
```

## Support
For issues or questions, see `PROJECT_STATUS.md` for full project context.
