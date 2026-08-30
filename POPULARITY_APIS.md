# Popularity Data Source APIs (Draft)

## Spotify
- Endpoint: `GET https://api.spotify.com/v1/search?q=track:{TITLE} artist:{ARTIST}&type=track&limit=1`
  - Extract `id` then `GET https://api.spotify.com/v1/tracks/{id}` for `popularity` (0–100).
- Audio Features: `GET https://api.spotify.com/v1/audio-features/{id}` (tempo, key, mode, danceability, energy).
- Auth: Client Credentials or OAuth; popularity available with client credentials.
- Rate: ~30 req/sec soft; batch via ids list endpoints.

## Last.fm
- Endpoint: `http://ws.audioscrobbler.com/2.0/?method=track.getInfo&api_key=KEY&artist={ARTIST}&track={TITLE}&format=json`
  - Fields: `listeners`, `playcount` (lifetime). For 12m playcount need internal delta snapshots.
- Tags: `track.getInfo` includes top tags.
- Rate: ~5 req/sec recommended.

## Soundcharts (Velocity / Real-time Charting)
- Private/paid API (placeholder) for trajectory: returns weekly positions, derived velocity.
- If unavailable: approximate velocity via Spotify weekly popularity diffs + Last.fm weekly playcount deltas.

## Wikidata / Wikipedia
- SPARQL for historical chart peaks (where modeled).
- Wikipedia scraping fallback for specific notable tracks.
- Use to backfill `peakPosition` / `weeksOnChart` when not in Soundcharts.

## YouTube (Future)
- Endpoint: `GET https://www.googleapis.com/youtube/v3/search?part=id&q={TITLE}+{ARTIST}` then `videos.list` for `statistics.viewCount`.
- Use viewCount (log-normalized) as cultural reach component.

## Normalization Summary
| Component | Raw | Transform | Notes |
|-----------|-----|-----------|-------|
| Spotify popularity | 0–100 | /100 | Already bounded |
| Listeners (Last.fm) | integer | log(value+1)/log(max+1) | max baseline 1M |
| Playcount 12m | integer | log(value+1)/log(max+1) | Need rolling window |
| Velocity % | delta fraction | clamp(delta*2) | +50% => 1.0 cap |
| Peak position | 1..100 | bucket scoring | See peakScore() |
| Weeks on chart | integer | log capped / log(60) | cap at 60 |
| Volatility ratio | std/mean | 1 - clamp(ratio) | stability proxy |
| Age days | integer | multiplier | penalize <90 days unless velocity high |

## Data Freshness Targets
- Spotify popularity: daily.
- Last.fm listeners/playcount: daily (or every 3 days if rate-limited).
- Velocity: weekly recompute.
- Chart peaks/weeks: static after initial retrieval.
- Age days: derived each score computation (no fetch).

## Fallback Logic
1. Missing Spotify => reduce weight (0 for pSpotify part).
2. Missing listeners/playcount => use median of existing dataset to avoid distortion.
3. Missing peak/weeks => treat as neutral (0.45 peak baseline, 0 weeks => 0).
4. High volatility with high velocity: cap negative impact to avoid punishing trending songs.

## Error Handling
- Rate limit responses => exponential backoff, mark metrics stale but keep last value.
- Partial failures => merge successes, flag components error states for UI transparency.

## Future Enhancements
- Integrate Shazam monthly tags for emerging detection.
- Playlist inclusion metrics (number of large editorial playlists > threshold).
- Social sentiment (Twitter/X, TikTok) via hashtag frequency ingestion.
