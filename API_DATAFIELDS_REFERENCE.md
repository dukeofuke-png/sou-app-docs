# API Datafields Quick Reference

**Last Updated:** December 1, 2025  
**Database Schema:** 177 columns in `sou_songs.db`

---

## 🎵 Musical Attributes

| Field | 1st Priority | 2nd Priority | 3rd Priority | 4th Priority | 5th Priority | Status |
|-------|-------------|-------------|-------------|-------------|-------------|--------|
| **bpm_best** | Deezer | GetSongBPM | Spotify Audio | TuneBat | Librosa | ⚠️ GetSongBPM Working, Spotify 403 |
| **bpm_spotify** | Spotify Audio Features | - | - | - | - | ❌ 403 Forbidden |
| **bpm_getsongbpm** | GetSongBPM API | - | - | - | - | ✅ Working |
| **bpm_tunebat** | TuneBat API | - | - | - | - | ⏳ Not implemented |
| **bpm_teaching** | Manual Override | - | - | - | - | ✅ Manual entry |
| **key_best** | GetSongBPM | Spotify Audio | TuneBat | Librosa | - | ⚠️ GetSongBPM Working, Spotify 403 |
| **spotify_key** | Spotify Audio Features | - | - | - | - | ❌ 403 Forbidden |
| **key_getsongbpm** | GetSongBPM API | - | - | - | - | ✅ Working |
| **key_tunebat** | TuneBat API | - | - | - | - | ⏳ Not implemented |
| **original_key** | Manual Override | - | - | - | - | ✅ Manual entry |
| **mode** | GetSongBPM | Spotify Audio | Librosa | - | - | ⚠️ GetSongBPM Working, Spotify 403 |
| **spotify_mode** | Spotify Audio Features | - | - | - | - | ❌ 403 Forbidden |
| **mode_getsongbpm** | GetSongBPM API | - | - | - | - | ✅ Working |
| **mode_tunebat** | TuneBat API | - | - | - | - | ⏳ Not implemented |
| **time_signature_best** | GetSongBPM | Spotify Audio | - | - | - | ⚠️ GetSongBPM Working, Spotify 403 |
| **spotify_time_signature** | Spotify Audio Features | - | - | - | - | ❌ 403 Forbidden |
| **time_signature_getsongbpm** | GetSongBPM API | - | - | - | - | ✅ Working (not extracted yet) |
| **tempo_label** | Derived from BPM | - | - | - | - | ✅ Calculated |
| **duration_s** | Spotify Track | - | - | - | - | ✅ Working |

---

## 🎭 Genres & Tags

| Field | 1st Priority | 2nd Priority | 3rd Priority | 4th Priority | 5th Priority | Status |
|-------|-------------|-------------|-------------|-------------|-------------|--------|
| **genres_best** | MERGED (all sources) | - | - | - | - | ✅ Multi-source |
| **genres_spotify** | Spotify Artist | - | - | - | - | ✅ Working |
| **genres_lastfm** | Last.fm Artist Tags | - | - | - | - | ✅ Working |
| **genres_mb** | MusicBrainz | - | - | - | - | ⏳ Not implemented |
| **genres_deezer** | Deezer | - | - | - | - | ⏳ Not implemented |
| **genres_getsongbpm** | GetSongBPM Artist | - | - | - | - | ⚠️ Available, not extracted |
| **tags_lastfm_track** | Last.fm Track Tags | - | - | - | - | ✅ Working |
| **tags_lastfm_artist** | Last.fm Artist Tags | - | - | - | - | ✅ Working |
| **discogs_genre** | Discogs | - | - | - | - | ⏳ Not implemented |
| **discogs_style** | Discogs | - | - | - | - | ⏳ Not implemented |

---

## 📅 Release Dates

| Field | 1st Priority | 2nd Priority | 3rd Priority | 4th Priority | 5th Priority | Status |
|-------|-------------|-------------|-------------|-------------|-------------|--------|
| **release_date_consolidated** | Soundcharts | MusicBrainz | Spotify | Wikidata | Wikipedia | ✅ Multi-source |
| **release_year_consolidated** | Soundcharts | MusicBrainz | Spotify | Wikidata | Wikipedia | ✅ Multi-source |
| **release_date_soundcharts** | Soundcharts | - | - | - | - | ✅ Working |
| **release_year_soundcharts** | Soundcharts | - | - | - | - | ✅ Working |
| **release_date_mb** | MusicBrainz | - | - | - | - | ⏳ Not implemented |
| **release_year_mb** | MusicBrainz | - | - | - | - | ⏳ Not implemented |
| **release_date_spotify** | Spotify Album | - | - | - | - | ✅ Working |
| **release_year_spotify** | Spotify Album | - | - | - | - | ✅ Working |
| **release_date_wd** | Wikidata P577 | - | - | - | - | ⏳ Not implemented |
| **release_year_wd** | Wikidata P577 | - | - | - | - | ⏳ Not implemented |
| **release_date_genius** | Genius | - | - | - | - | ⏳ Not implemented |
| **release_date_manual** | Manual Override | - | - | - | - | ✅ Manual entry |
| **release_year_manual** | Manual Override | - | - | - | - | ✅ Manual entry |
| **release_month** | Derived from date | - | - | - | - | ✅ Calculated |
| **release_season** | Derived from date | - | - | - | - | ✅ Calculated |
| **release_era** | Derived from year | - | - | - | - | ✅ Calculated |
| **year** | Legacy field | - | - | - | - | ✅ Working |

---

## 📊 Chart Positions

| Field | 1st Priority | 2nd Priority | 3rd Priority | 4th Priority | 5th Priority | Status |
|-------|-------------|-------------|-------------|-------------|-------------|--------|
| **chart_peak_position** | Soundcharts | Wikipedia | Wikidata | Internal Dataset | Popularity Fallback | ✅ Multi-source |
| **chart_source** | Soundcharts | Wikipedia | Wikidata | Internal | - | ✅ Multi-source |
| **top_10** | Derived from peak | - | - | - | - | ✅ Calculated |
| **top_40** | Derived from peak | - | - | - | - | ✅ Calculated |
| **wiki_chart_peak** | Wikipedia Scrape | - | - | - | - | ✅ Working |
| **wiki_chart_source** | Wikipedia Scrape | - | - | - | - | ✅ Working |
| **charts_from_wikidata** | Wikidata P2291 | - | - | - | - | ⏳ Partial |
| **charts_from_wikipedia** | Wikipedia Tables | - | - | - | - | ✅ Working |
| **popularity_tier** | Derived from metrics | - | - | - | - | ✅ Calculated |

---

## 📈 Popularity Metrics

| Field | 1st Priority | 2nd Priority | 3rd Priority | 4th Priority | 5th Priority | Status |
|-------|-------------|-------------|-------------|-------------|-------------|--------|
| **spotify_popularity** | Spotify Track | - | - | - | - | ✅ Working |
| **lastfm_plays** | Last.fm Track | - | - | - | - | ✅ Working |
| **lastfm_listeners** | Last.fm Track | - | - | - | - | ✅ Working |
| **youtube_views** | YouTube Data API | - | - | - | - | ✅ Working |
| **genius_pageviews** | Genius API | - | - | - | - | ⏳ Not implemented |

---

## ✍️ Songwriters & Credits

| Field | 1st Priority | 2nd Priority | 3rd Priority | 4th Priority | 5th Priority | Status |
|-------|-------------|-------------|-------------|-------------|-------------|--------|
| **songwriters_best** | MusicBrainz | Wikidata | Wikipedia | Genius | Manual | ⏳ Not implemented |
| **songwriters_mb** | MusicBrainz | - | - | - | - | ⏳ Not implemented |
| **songwriters_wd** | Wikidata P676/P86 | - | - | - | - | ⏳ Not implemented |
| **songwriters_wikipedia** | Wikipedia Parse | - | - | - | - | ⏳ Not implemented |
| **songwriters_genius** | Genius API | - | - | - | - | ⏳ Not implemented |
| **writers_mb** | MusicBrainz | - | - | - | - | ⏳ Not implemented |
| **words_and_music** | Manual Override | - | - | - | - | ✅ Manual entry |
| **publisher** | Wikidata | Manual | - | - | - | ⏳ Not implemented |
| **producers_genius** | Genius API | - | - | - | - | ⏳ Not implemented |
| **has_writer_credits** | Derived boolean | - | - | - | - | ✅ Calculated |

---

## 🎨 Cover Art

| Field | 1st Priority | 2nd Priority | 3rd Priority | 4th Priority | 5th Priority | Status |
|-------|-------------|-------------|-------------|-------------|-------------|--------|
| **cover_art_url_best** | Spotify (640x640) | Soundcharts | Last.fm (300x300) | Genius | MusicBrainz | ✅ Multi-source |
| **cover_art_url_spotify** | Spotify Album | - | - | - | - | ✅ Working |
| **cover_art_album_spotify** | Spotify Album | - | - | - | - | ✅ Working |
| **cover_art_url_soundcharts** | Soundcharts | - | - | - | - | ✅ Working |
| **cover_art_url_lastfm** | Last.fm Album | - | - | - | - | ✅ Working |
| **cover_art_album_lastfm** | Last.fm Album | - | - | - | - | ✅ Working |
| **genius_album_art_url** | Genius API | - | - | - | - | ⏳ Not implemented |
| **discogs_cover_url** | Discogs | - | - | - | - | ⏳ Not implemented |

---

## 📺 YouTube

| Field | 1st Priority | 2nd Priority | 3rd Priority | 4th Priority | 5th Priority | Status |
|-------|-------------|-------------|-------------|-------------|-------------|--------|
| **youtube_video_id** | YouTube Data API | - | - | - | - | ✅ Working |
| **youtube_url** | Constructed from ID | - | - | - | - | ✅ Working |
| **youtube_title** | YouTube Data API | - | - | - | - | ✅ Working |
| **youtube_channel** | YouTube Data API | - | - | - | - | ✅ Working |
| **youtube_views** | YouTube Data API | - | - | - | - | ✅ Working |
| **youtube_published_date** | YouTube Data API | - | - | - | - | ✅ Working |
| **youtube_match_confidence** | Calculated | - | - | - | - | ✅ Working |
| **youtube_url_genius** | Genius API | - | - | - | - | ⏳ Not implemented |

---

## 🎵 Spotify-Specific

| Field | 1st Priority | 2nd Priority | 3rd Priority | 4th Priority | 5th Priority | Status |
|-------|-------------|-------------|-------------|-------------|-------------|--------|
| **spotify_track_id** | Spotify Search | - | - | - | - | ✅ Working |
| **spotify_isrc** | Spotify Track | - | - | - | - | ✅ Working |
| **spotify_release_date** | Spotify Album | - | - | - | - | ✅ Working |
| **spotify_danceability** | Spotify Audio Features | - | - | - | - | ❌ 403 Forbidden |
| **spotify_energy** | Spotify Audio Features | - | - | - | - | ❌ 403 Forbidden |
| **spotify_valence** | Spotify Audio Features | - | - | - | - | ❌ 403 Forbidden |

---

## 🔗 External IDs

| Field | 1st Priority | 2nd Priority | 3rd Priority | 4th Priority | 5th Priority | Status |
|-------|-------------|-------------|-------------|-------------|-------------|--------|
| **musicbrainz_recording_id** | MusicBrainz Search | - | - | - | - | ✅ Working |
| **wikidata_qid** | Wikidata Entity Search | - | - | - | - | ⏳ Not implemented |
| **genius_song_id** | Genius Search | - | - | - | - | ⏳ Not implemented |
| **discogs_master_id** | Discogs Search | - | - | - | - | ⏳ Not implemented |
| **discogs_release_id** | Discogs Search | - | - | - | - | ⏳ Not implemented |
| **apple_music_id_genius** | Genius API | - | - | - | - | ⏳ Not implemented |

---

## 📚 Wikipedia/Wikidata

| Field | 1st Priority | 2nd Priority | 3rd Priority | 4th Priority | 5th Priority | Status |
|-------|-------------|-------------|-------------|-------------|-------------|--------|
| **wikipedia_url** | Wikipedia API | - | - | - | - | ✅ Working |
| **wikipedia_intro** | Wikipedia Parse | - | - | - | - | ✅ Working |
| **wikipedia_charts_text** | Wikipedia Scrape | - | - | - | - | ✅ Working |
| **wikipedia_reception** | Wikipedia Parse | - | - | - | - | ⏳ Not implemented |
| **wikidata_properties** | Wikidata SPARQL | - | - | - | - | ⏳ Not implemented |

---

## 🎓 Teaching Metadata
*Source: School of Uke Song Sheets Database.csv (Manual Entry)*

| Field | Source | Status |
|-------|--------|--------|
| **level** | Teaching CSV | ❌ **NOT IN DATABASE** |
| **sou_keys** | Teaching CSV | ❌ **NOT IN DATABASE** |
| **num_chords** | Teaching CSV | ❌ **NOT IN DATABASE** |
| **chords** | Teaching CSV | ❌ **NOT IN DATABASE** |
| **chord_numerals** | Teaching CSV | ❌ **NOT IN DATABASE** |
| **strum_style** | Teaching CSV | ❌ **NOT IN DATABASE** |
| **fingerpicking_style** | Teaching CSV | ❌ **NOT IN DATABASE** |
| **teaching_notes** | Teaching CSV | ❌ **NOT IN DATABASE** |
| **notes** | Teaching CSV | ❌ **NOT IN DATABASE** |
| **bpm_teaching** | Teaching CSV | ❌ **NOT IN DATABASE** |
| **bpm_teaching_rule** | Teaching CSV | ❌ **NOT IN DATABASE** |
| **song_sheet_status** | Teaching CSV | ❌ **NOT IN DATABASE** |
| **tab_status** | Teaching CSV | ❌ **NOT IN DATABASE** |
| **song_sheet_url** | Teaching CSV | ❌ **NOT IN DATABASE** |
| **tab_url** | Teaching CSV | ❌ **NOT IN DATABASE** |
| **has_song_sheet** | Teaching CSV | ❌ **NOT IN DATABASE** |
| **has_tab** | Teaching CSV | ❌ **NOT IN DATABASE** |
| **materials_folder** | Teaching CSV | ❌ **NOT IN DATABASE** |
| **song_sheet_path** | Teaching CSV | ❌ **NOT IN DATABASE** |
| **melody_tab_path** | Teaching CSV | ❌ **NOT IN DATABASE** |

---

## 🔧 Enrichment Metadata

| Field | Source | Status |
|-------|--------|--------|
| **date_added** | Auto-generated | ✅ Working |
| **last_modified** | Auto-updated trigger | ✅ Working |
| **last_enriched_utc** | Enrichment service | ✅ Working |
| **spotify_audio_features_retrieved_at** | Enrichment service | ✅ Working |
| **lastfm_retrieved_at** | Enrichment service | ✅ Working |
| **getsongbpm_retrieved_at** | Enrichment service | ✅ Working |
| **youtube_retrieved_at** | Enrichment service | ✅ Working |
| **musicbrainz_retrieved_at** | Enrichment service | ✅ Working |
| **genius_retrieved_at** | Enrichment service | ⏳ Not implemented |
| **cover_art_retrieved_at** | Enrichment service | ✅ Working |
| **materials_linked_at** | Manual process | ⏳ Not implemented |
| **bpm_best_retrieved_at** | Enrichment service | ✅ Working |
| **songwriters_retrieved_at** | Enrichment service | ⏳ Not implemented |

---

## 📊 Confidence Scores

| Field | Source | Status |
|-------|--------|--------|
| **bpm_best_confidence** | Calculated 0-1 | ✅ Working |
| **bpm_best_match_score** | API response | ✅ Working |
| **musicbrainz_match_confidence** | Calculated 0-1 | ✅ Working |
| **youtube_match_confidence** | Calculated 0-1 | ✅ Working |
| **materials_match_score** | Calculated 0-1 | ⏳ Not implemented |
| **confidence** | Legacy field | ✅ Working |

---

## 🏷️ Discogs (Not Yet Implemented)

| Field | 1st Priority | Status |
|-------|-------------|--------|
| **discogs_year** | Discogs API | ⏳ Not implemented |
| **discogs_country** | Discogs API | ⏳ Not implemented |
| **discogs_label** | Discogs API | ⏳ Not implemented |
| **discogs_format** | Discogs API | ⏳ Not implemented |
| **discogs_resource_url** | Discogs API | ⏳ Not implemented |

---

## 🔑 Status Legend

| Symbol | Meaning |
|--------|---------|
| ✅ | **Working** - Implemented and functioning correctly |
| ⚠️ | **Partial** - Some sources working, others blocked/pending |
| ❌ | **Not Working** - Implemented but blocked/failing |
| ⏳ | **Not Implemented** - Planned but not yet built |

---

## 🚨 Critical Issues

### **High Priority**
1. ❌ **ALL Teaching Data Missing** - 20+ fields empty in database
   - Root cause: Database migrated from wrong CSV (enriched template, not actual teaching data)
   - Impact: Admin UI shows empty columns for Level, Keys, Chords, Song Sheets, etc.
   - Solution needed: Import from `School of Uke Song Sheets Database.csv`

2. ❌ **Spotify Audio Features Blocked** - 403 Forbidden
   - Fields affected: `spotify_danceability`, `spotify_energy`, `spotify_valence`, `spotify_mode`, `spotify_key`, `spotify_time_signature`, `spotify_bpm`
   - Root cause: Client Credentials flow doesn't support audio features endpoint
   - Workaround: Use GetSongBPM for BPM/Key/Mode/Time Signature

### **Medium Priority**
3. ⏳ **Songwriter Credits Not Implemented**
   - Fields affected: All `songwriters_*` fields
   - APIs available: MusicBrainz (best), Wikidata, Wikipedia parsing
   - Impact: Missing songwriter attribution

4. ⏳ **Deezer BPM Not Implemented**
   - Documented as primary BPM source but never built
   - Would improve BPM accuracy and reduce API dependency on GetSongBPM

### **Low Priority**
5. ⏳ **Genius API Not Integrated**
   - Fields affected: `genius_song_id`, `genius_pageviews`, `producers_genius`, `youtube_url_genius`
   - Alternative sources available for most data

---

## 📈 Implementation Progress

**Total Fields:** 177  
**Fully Working:** ~45 fields (25%)  
**Partially Working:** ~30 fields (17%)  
**Not Implemented:** ~80 fields (45%)  
**Missing Data (Teaching):** ~20 fields (11%)  
**Blocked/Failing:** ~2 fields (1%)

---

## 🎯 Next Steps (Priority Order)

1. **Import Teaching Data** - Populate 20+ empty teaching fields from CSV
2. **Extract GetSongBPM Time Signature** - Already available in API response
3. **Extract GetSongBPM Artist Genres** - Add to genre merge
4. **Implement MusicBrainz Songwriters** - High-value missing data
5. **Implement Deezer BPM** - Improve BPM source diversity
6. **Build Wikidata Integration** - Fill date/chart/songwriter gaps
7. **Add Discogs Enrichment** - Label/format/genre metadata
8. **Implement Genius API** - Lyrics and additional metadata
9. **Build Librosa Service** - Local MP3 analysis for drag-and-drop

---

**End of API Datafields Reference**
