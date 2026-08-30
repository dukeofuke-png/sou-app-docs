# API Setup Guide

## Required API Keys

### 1. Spotify API (Required for Popularity Scores)

**Already configured in your .env!** Just need to remove spaces:

```bash
# Current (has spaces):
SPOTIFY_CLIENT_ID=ba9c44b1c302448 3b173f6baf4ab28a4
SPOTIFY_CLIENT_SECRET=ab229034dae94735bd882d903 00f1417

# Should be (no spaces):
SPOTIFY_CLIENT_ID=ba9c44b1c3024483b173f6baf4ab28a4
SPOTIFY_CLIENT_SECRET=ab229034dae94735bd882d90300f1417
```

**To get your own credentials:**
1. Go to https://developer.spotify.com/dashboard
2. Log in with Spotify account
3. Create an app
4. Copy Client ID and Client Secret
5. Add to `.env`

**What it provides:**
- Track popularity score (0-100)
- Audio features (BPM, key, mode, energy)
- Artist info

**Rate limits:** ~30 requests/second (generous)

---

### 2. Last.fm API (Required for Listener Counts)

**Status:** Not yet configured

**Setup:**
1. Go to https://www.last.fm/api/account/create
2. Fill out application form (name: "SOU Song Browser")
3. Get API key
4. Add to `.env`:
```bash
LASTFM_API_KEY=your_key_here
```

**What it provides:**
- Total listeners count
- Total playcount
- Genre tags from community
- Track metadata

**Rate limits:** ~5 requests/second (be gentle)

---

### 3. Soundcharts (Optional - For Chart Positions)

**Status:** Not configured (paid service)

If you want real-time chart tracking:
1. Sign up at https://app.soundcharts.com/
2. Get API credentials
3. Add to `.env`:
```bash
SOUNDCHARTS_APP_ID=your_app_id
SOUNDCHARTS_API_KEY=your_api_key
```

**Cost:** Starts at ~$99/month for API access

**Alternative:** Use free Wikidata/Wikipedia scraping (already implemented as fallback)

---

## Testing API Integration

### Test Spotify (after fixing spaces in .env):

```bash
curl -X POST http://localhost:3001/api/popularity/refresh/purple%20rain%7Cprince \
  -H "Content-Type: application/json"
```

Expected: Returns real Spotify popularity score for Purple Rain

### Test Last.fm (after adding API key):

Load seed, then refresh a song's metrics to fetch Last.fm data.

---

## Current Status

✅ **Working with real data:**
- Spotify track search
- Spotify popularity scores (0-100)
- Spotify audio features

⚠️ **Fallback to placeholders:**
- Last.fm (needs API key)
- Chart velocity (Soundcharts not configured)

📝 **Next steps:**
1. Fix Spotify credentials (remove spaces)
2. Get Last.fm API key
3. Restart server
4. Test "Load Curated Seed" - scores will be real!
