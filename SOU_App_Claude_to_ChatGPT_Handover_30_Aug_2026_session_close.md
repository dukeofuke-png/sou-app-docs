# SOU App — Claude → ChatGPT Handover
**Date:** 30 August 2026 (session close)
**Purpose:** Reverse handover per the Project Operating System's Section 5.3. Carried across manually by Matthew.

---

## 1. Phase 1 (song-sheet content extraction) — built, tested, closed

Following your two schema corrections and the approved PoC-first sequencing, here's how it landed:

**Inspection finding:** no file/material identity concept existed anywhere in the schema prior to this — confirmed by live DB inspection. `extracted_content` is the first thing giving a PDF real row-level identity.

**PoC result:** song-sheet (Keynote) PDFs extract cleanly and usably. TAB (Guitar Pro) PDFs do not — only title/copyright labels survive intelligibly, actual tab content is fragmented and unusable via plain text extraction. You agreed to scope TAB out entirely; recorded as a deferred future investigation (likely OCR), not folded into this build.

**Schema shipped**, per your final correction (composite `UNIQUE(song_id, source_type)`, not standalone `song_id`):
```sql
CREATE TABLE extracted_content (
  id                  INTEGER PRIMARY KEY AUTOINCREMENT,
  song_id             TEXT NOT NULL REFERENCES songs(id),   -- see note below
  source_type         TEXT NOT NULL DEFAULT 'song_sheet' CHECK (source_type IN ('song_sheet')),
  source_url          TEXT NOT NULL,
  extracted_text      TEXT,
  extraction_status   TEXT NOT NULL DEFAULT 'pending'
                         CHECK (extraction_status IN ('pending', 'success', 'failed', 'low_confidence')),
  extraction_error    TEXT,
  extracted_at        TEXT,
  created_at          TEXT NOT NULL DEFAULT (datetime('now')),
  UNIQUE(song_id, source_type)
);
```

**One deviation from the approved version, caught during implementation, not before:** `song_id` had to be `TEXT`, not `INTEGER` — `songs.id` turns out to be a TEXT slug primary key (`craig_david_7_days`), not an integer. Copilot caught this via live schema inspection before writing any data, flagged it, and I approved the fix as a pure type correction with no other schema impact. Worth knowing for any future proposal referencing `songs(id)` — **including the approved Milestone A v4 proposal, which still has `arrangements.song_id INTEGER` and needs the same fix before implementation.**

**Batch extraction results (191 songs with a `song_sheet_path`):** 175 succeeded, 9 low-confidence, 7 failed. One failure (Espresso) is the already-known missing-file gap. **The other 6 are a new finding, not previously known:** genuinely empty (0-byte) PDF objects on R2, confirmed via direct HTTP check, not an extraction-script bug. Real broken song sheets currently live in production for: Alright (Supergrass), Creep (Radiohead), Hot Stuff (Donna Summer), San Francisco (Scott McKenzie), Staying Out For The Summer (Dodgy), Tubular Bells from The Exorcist (Mike Oldfield). This is a data-integrity issue independent of Phase 1 — logged, not yet fixed.

**`searchSongContent` tool:** built, wired into Studio via the existing `executeToolWithAudit()` pattern (confirmed compatible with read-only tools with no wrapper changes needed — `isMutation: false` cleanly skips the mutation-gating logic). Smoke-tested against real extracted data (not just syntax-checked): correctly surfaced "7 Days" for a "subway" query and "All About That Bass" for "no treble," no false positives on a nonsense query. System prompt extended so the Phase 0 "verify before asserting SOU-specific facts" principle explicitly covers content/lyric claims, with explicit tool-honesty language that it doesn't cover TAB content.

Committed and verified live on `materials-server`'s `origin/main` (`364d9ea`).

## 2. Milestone A v4 — approved by Matthew

No implementation started. Per `PRODUCT_ROADMAP.md`'s explicit sequencing (see item 3 below), this is next once Phase 0 and Phase 1 are both closed — which they now are — but the actual start of implementation should be reconfirmed explicitly at the next session's start, not assumed.

## 3. Resolved: the Phase 0/Milestone A sequencing question from earlier today

Turned out not to be genuinely ambiguous — `PRODUCT_ROADMAP.md` Sections 17–18 (not read in full when this was first flagged) explicitly serialize Phase 0 → 1 → 2 → 3 → 4. Full detail in the earlier reverse handover from today, not repeated here.

## 4. Documentation/infrastructure housekeeping this session

- All canonical docs (including this one's source material) now live in a real git repo, `dukeofuke-png/sou-app-docs`, after discovering they had zero backup anywhere except manually-uploaded Project Knowledge copies.
- Dated-filename versioning retired going forward — one canonical filename per doc, git history as the version record.

## 5. Still open, carried forward

1. The 6 empty-file songs above.
2. `sou-song-browser`'s `SongDetailModal.js` — uncommitted local diff, never triaged.
3. Unexplained `src`/`scripts` folder at the outer `SOU App/` root, outside both known repos.
4. `arrangements.song_id INTEGER` in the approved Milestone A v4 proposal needs the same TEXT correction Phase 1 already made.

---
*Snapshot for cross-checking, not a canonical file.*
