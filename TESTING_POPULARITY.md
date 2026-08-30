# Testing Guide: Popularity Catalog & Admission Policy

## Backend Setup (Complete)
Server running on port 3001 with new endpoints:
- `POST /api/popularity/seed/load` - Load curated_seed.json
- `GET /api/popularity/catalog?minScore=0.5&tier=Classic&limit=50` - List songs with filters
- `GET /api/popularity/seed/stats` - Seed identities/metrics counts + threshold
- `POST /api/popularity/refresh/:canonicalKey` - Force refresh metrics for a song
- `GET /api/popularity/recommendations` - Songs above threshold not in CSV (promote candidates)
- `POST /api/search/advanced` - Now returns `sessionId` in meta

## Frontend Testing Steps

### 1. Start Frontend (Admin Dashboard)
Open a second terminal and run:
\`\`\`bash
cd /Users/matthew/Documents/SOU\ App
npm start
\`\`\`

This should start React dev server on http://localhost:3000

### 2. Access Admin Dashboard
Navigate to: http://localhost:3000/admin
(Auth bypass active in dev mode—you're auto-logged in)

### 3. Verify Popularity Catalog Section
Scroll to bottom of admin dashboard after song table. You should see:
- **Filters row**: Min Score slider, Tier dropdown, Source input, Limit
- **Action buttons**: "Refresh" and "Load Curated Seed"
- **Stats line**: "Seed Identities: 0 | Metrics: 0" (initially)
- **Empty table**: Title, Artist, Score, Tier, Sources, Components columns

### 4. Load Seed
Click **"Load Curated Seed"** button.
Expected behavior:
- Button disables after first click
- Stats update to show: "Seed Identities: 5 | Metrics: 5" (5 demo songs from curated_seed.json)
- Table populates with rows:
  - Purple Rain (Prince), ~74 score, Classic tier
  - Billie Jean (Michael Jackson), ~77 score, Classic tier
  - Rolling in the Deep (Adele), ~78 score, Classic tier
  - Bad Guy (Billie Eilish), ~80 score, Classic tier
  - Take Me Home, Country Roads (John Denver), ~67 score, High tier

Each row should display:
- **Score**: Percentage (0–100 scale)
- **Tier badge**: Color-coded (Classic=purple, High=blue, Solid=green, Emerging=orange)
- **Sources**: "curated"
- **Components**: Tiny text showing pSpotify, pListeners, etc. (may be hidden on mobile)

### 5. Test Filters
Adjust **Min Score** slider to 0.75:
- Table should hide songs below 75% score
- Expected visible: Billie Jean, Rolling in the Deep, Bad Guy (3 songs)

Select **Tier dropdown** to "Classic":
- Should filter to only Classic tier songs

Reset filters to see all again.

### 6. Recommendations Endpoint Test (Manual API)
Open browser console (F12) and run:
\`\`\`javascript
fetch('http://localhost:3001/api/popularity/recommendations', {credentials: 'include'})
  .then(r => r.json())
  .then(console.log)
\`\`\`

Expected: JSON with `recommendations` array containing songs above threshold (0.62) that are not already in CSV database. Since these are new demo songs and likely not in your CSV, they should appear as candidates for promotion.

### 7. Session ID in Search (Verify)
Test advanced search with artist filter:
- Navigate to Search Music Database section (if you have it in UI)
- Or use browser console:
\`\`\`javascript
fetch('http://localhost:3001/api/search/advanced', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  credentials: 'include',
  body: JSON.stringify({ artist: 'Weeknd' })
}).then(r => r.json()).then(console.log)
\`\`\`

Expected response includes \`meta.sessionId\` field (UUID).

## Troubleshooting

### Server not responding
Ensure backend running: \`lsof -ti:3001\` should return process ID.
Restart if needed:
\`\`\`bash
cd /Users/matthew/Documents/SOU\ App/materials-server
PORT=3001 node server.js
\`\`\`

### CORS errors in browser
Check server logs for request arrival. Verify frontend URL matches \`corsOptions.origin\`.

### Empty catalog after load
Check server logs for "Seed load error". Verify \`curated_seed.json\` exists in materials-server directory.

### Components not visible
Responsive CSS hides component breakdown on screens <900px width. Widen browser window.

## Next Steps After Verification
Once catalog displays correctly:
1. **Promote flow**: Add "Promote to Database" button in UI for recommendation rows, calls new endpoint to write song to CSV.
2. **Real API integration**: Replace placeholder metrics in \`popularityAggregator.js\` with Spotify/Last.fm calls.
3. **Refresh automation**: Schedule background task to update metrics daily.
4. **Expanded seed**: Ingest larger curated sources (e.g., Billboard Top 40 per decade).

## Documentation References
- \`POPULARITY_APIS.md\` - External data source API details
- \`ARCHITECTURE_POPULARITY_NEXT_STEPS.md\` - Implementation roadmap & MVP sequence
- \`admissionPolicy.js\` - Threshold logic (currently 0.62 min score)
- \`popularityScore.js\` - Scoring algorithm & tier boundaries
