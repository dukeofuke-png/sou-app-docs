# SOU Song Database — Status Snapshot (2025-10-25)

## Project folders
/data, /tools, /logs  (VS Code + integrated terminal; Miniforge base)

/tools key scripts
- backup.sh
- sanity_check.py
- sou_wiki_backfill.py        -> adds Charts (from Wikipedia) [Backfill]
- sou_enrich_mb_lastfm.py     -> MB + Last.fm enrichment (fills blanks; retries)
- sou_wikidata_backfill.py    -> Charts (from Wikidata) via P2291/P1352/P585 (some items lack data)
- sou_charts_consolidate.py   -> Creates "Charts (Consolidated)" preferring WD over WP
- sou_release_wd_backfill.py  -> NEW: Release Date (WD/P577) + MB/WP fallback (versioned outputs)

/data latest
- input:  output_with_chart_mb_wd_full_wd2.csv
- output: songdb_output_v1_dates.csv  (created by release backfill)
- DB:     song_database.db (Phase 5 SQLite)

/config
- version.txt -> current version number for outputs (start: 1)

## Next actions
- (Optional) Retrofit all scripts to use versioned outputs “songdb_output_v{V}_…”
- Add SQLite build step for the newest versioned CSV
- Add quick QA script: list rows still missing Release Year (Consolidated) or Charts (Consolidated)✅ v11 Discogs enrichment complete – Sat Oct 25 21:39:54 BST 2025
