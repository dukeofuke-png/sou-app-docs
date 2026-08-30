# SOU App — Handover Summary (for new Project)

**Purpose of this document:** condensed context from prior SOU App conversations across the project's full history, so this Project doesn't start blank. Supplements the five canonical files (MASTER_ARCHITECTURE.md, PRODUCT_ROADMAP.md, the Consolidated Product & UX Specification, the Design Reconciliation Audit, the Current Build Gap Analysis) rather than duplicating them.

---

## History — how the app got here

**October 2024 — first ever conversation about it.** Matthew asked how to build a simple School of Uke tutor app: hosted songsheets/theory sheets, a song finder filterable by year, key, difficulty, chord count, TAB, fingerpicking. Generic early-stage advice (Firebase/AWS, React Native/Flutter, Firestore). Nothing built yet — this is the seed idea, not an architectural decision.

**Mid-2025 — no-code detour, abandoned.** Explored Famous.ai (AI agent/automation tool) and Supabase as a possible backend, in a conversation called "Song Database Dev." Famous.ai was assessed as unsuitable for building the actual platform (booking, dashboards, content delivery) — at most useful as a natural-language query layer sitting on top of a real backend. This entire direction (Famous.ai + Supabase) was later abandoned in favour of the current stack (React/Vercel + Express/Railway + SQLite). Worth knowing this existed so old references to Supabase in very early chats aren't mistaken for current architecture.

**Also mid-2025 — Wave, a separate app concept.** Not to be confused with the SOU App. Wave is Matthew's personal task-manager companion app concept (task list as "menu not debt ledger," emotional tagging, resonance state) — a distinct, personal project unrelated to School of Uke. If old chats reference "Wave," it's this, not a SOU feature.

**AWS attempted and abandoned** at some point before the current stack — six months of failed deployment attempts, per the Developer Handover doc. Railway + Vercel was adopted as the stable replacement.

**November 2025 — code quality audit (via ChatGPT, reviewing Copilot-generated code).** Found: inconsistent patterns from Copilot-generated code (duplicated logic, invented helper functions instead of reusing existing ones), no unified "shared utilities" concept across scripts, folder structure unclear (tools vs enrichment vs tests), and a general lack of guardrails when prompting Copilot (it doesn't retain project-wide context, so it invents new files unless explicitly told to search first). Recommended: a PROJECT_GUIDE.md with naming/structure rules, a shared helpers file, explicit "check for existing functions before creating new ones" instructions to Copilot, and regular refactor passes. This is the origin of the session-discipline conventions (read-architecture-first, no new MD files without updating the master doc, etc.) that are now standard practice.

**Also in this period — data pipeline and enrichment work.** Extensive API-enrichment pipeline built for song metadata: Spotify, MusicBrainz, Last.fm, Soundcharts, Discogs, Deezer, Genius, YouTube, GetSongBPM, Wikipedia/Wikidata. Two databases exist conceptually: a large "seed/discovery" catalogue (tens of thousands of songs, prioritises speed/coverage) and the curated SOU teaching database (~200+ songs, prioritises accuracy — multi-source verified). Genre/BPM/key/release-date fields all follow documented priority waterfalls across these APIs. Known issue from this era, still true: `chords` field ~14% populated with ~56% of existing values corrupted from a historical CSV column-shift bug (never parse CSV with naive `.split(',')` — always use a real CSV parser; this caused three separate corruption incidents).

**June 2026 — Developer Handover document written** (for an incoming developer, per its own header — unclear if this developer ever came on, but the document is a genuinely thorough snapshot of the app at that point). Key facts as of then:
- 216 songs in the teaching DB, 47K in the seed catalogue, 177-column schema
- Stack confirmed: React (Create React App) on Vercel, Express/Node 18 on Railway, SQLite via Railway Volume, PDFs on Cloudflare R2
- AI: provider-agnostic wrapper, default Gemini 2.5 Flash, fallback Claude Haiku
- Auth: single shared `ADMIN_PASSWORD`, no multi-tutor accounts yet; `tutors` table exists with Matthew seeded as `tutor_id=1` (still true as of the August gap analysis)
- Known deferred items at the time: MemoryStore sessions wipe on every Railway redeploy (fix deferred until a second tutor account is needed), Deezer-as-primary-BPM-source (deferred), Gemini system prompt still a placeholder, Postgres migration deferred until multi-tutor concurrency needed, no Google Drive → app PDF auto-sync
- Working style already established: one step at a time, show proposed schema/code before implementing, read MASTER_ARCHITECTURE.md at the start of every session, log every session, no new MD files without updating the master doc, strategic thinking to Claude / code execution to Copilot in VS Code agent mode

**August 2026 — four-day design/UX Q&A, then reconciliation.** This produced the Consolidated Product & UX Specification (the big "Tutor Studio" vision — Courses, Lessons, Appraisals, Actions, native Arrangement objects, Teaching Mode, SA-as-omnipresent-assistant, etc.), which the Design Reconciliation Audit then assessed against the existing canonical docs, and which the Current Build Gap Analysis then compared against the actual repo/database. Full detail on this phase is in the five canonical documents themselves — not repeated here.

**26 August 2026 — most recent confirmed session.** TAB files migrated to Cloudflare R2 (49/49, having been stuck at 0/49), plus a related rendering bug fixed in `SongDetailModal.js` where the TAB link was silently unreachable whenever a Song Sheet link was also present. Full detail in MASTER_ARCHITECTURE.md's session log.

---

## Current technical state (as of 26 Aug 2026)

- **Stack:** React frontend (sou-song-browser) on Vercel, Express/SQLite backend (materials-server) on Railway.
- **Database:** 218 curated SOU teaching songs + ~47,000-song discovery/seed catalogue. Schema recently confirmed at 239 columns (grew from 177 via migrations).
- **PDF storage:** Migrated to Cloudflare R2. Song sheets 190/191 migrated; TAB files 49/49 migrated as of the 26 Aug session (previously 0/49 — this was a live bug fixed that day). `/materials/*` legacy endpoint retired (returns 410 Gone).
- **Known outstanding issue:** `materials_json` field still holds stale local Google Drive paths never updated by either R2 migration script — not read by the primary render path, but a lingering inconsistency worth cleaning up eventually.
- **AI chat (Studio):** Live using Gemini 2.5 Flash via a provider-agnostic wrapper. Tool-call audit trail exists. Currently scoped to hardcoded `tutor_id = 1` — no real multi-tutor auth yet.

## Where the product direction stands

An August 2026 four-day design/UX Q&A produced a consolidated specification for a much richer "Tutor Studio" product (Courses, Lessons, Appraisals, Actions, native Arrangement objects, Teaching Mode, etc.) — almost none of which exists in the current build yet. A gap analysis concluded the current app is still closer to "Song Catalog + Admin + AI Chat" than to that resolved vision.

**The agreed shortest useful path (September Utility) is three milestones:**
- **Milestone A** — native Arrangement object + A4 Print View + persistent PDF publication. This is the current focus. A v4 architecture proposal was submitted for approval; no Copilot implementation has been authorised yet.
- **Milestone B** — lightweight Course/Lesson planning (Course, Lesson, Full Lesson Plan, Short-form Prompt Sheet).
- **Milestone C** — close the feedback loop (Register, Appraisal, SA processing, next-lesson context).

**Open governance gap (unresolved, not urgent):** the Design Reconciliation Audit recommended promoting the Consolidated Product & UX Specification into a fourth canonical file (`PRODUCT_UX_SPEC.md`), archiving the old `PROJECT_FEATURE_MAP.md`, and updating both `MASTER_ARCHITECTURE.md`'s governance references and `PRODUCT_ROADMAP.md`'s sequencing to reflect the new milestones. That update chain was only partially executed. Worth tidying up at some point, but not a blocker to Milestone A work.

## Working conventions established

- **Copilot session discipline:** every session starts by reading `MASTER_ARCHITECTURE.md` in full plus the latest session-log entry, and ends by updating the session log, current-status section, and decisions log within that same file. No new MD files without updating MASTER_ARCHITECTURE.md.
- **Repo rule:** active frontend repo is `sou-song-browser/`. A stale outer git repo exists at `SOU App/` — never commit from there.
- **Node.js locked to v18.** Vercel build command: `CI=false npm run build`.
- **Tool truth over conversational claim:** a mutation only counts once the backend confirms it — SA/chat text is never treated as proof of a write.
- **In Copilot prompts, Matthew is referred to as "the admin."**

## Addendum — 30 Aug 2026: newer session, not yet reflected above

A session after 26 Aug (before 30 Aug) did substantial AI/Studio tooling work. This is more recent than everything else in this document and should be treated as the current state of the AI/Studio thread specifically.

**Built, tested, but deliberately NOT committed or shipped:** `searchSeedCatalog` and `promoteSongFromSeed` (in `services/seedService.js`, `services/aiProvider.js`, `routes/chat.js`, backend repo `materials-server`) — gives Studio the ability to search the 47K discovery catalogue and promote songs into the teaching library. All code remains local/uncommitted because testing surfaced three reliability problems judged serious enough to block shipping:

1. **No audit trail for AI tool calls.** Only the AI's final natural-language text is saved to the `messages` table — no record of which tools were actually called or what they returned.
2. **The AI can claim a write happened when it didn't.** Observed directly in production: chat text said a promotion was in progress while the backend log showed no corresponding tool call that turn.
3. **STUDIO-01 (bug, confirmed live in production, unfixed):** raw internal tool-response JSON leaks directly into the Studio chat UI during multi-step searches instead of being hidden behind a loading state.

A related investigation into inconsistent AI behaviour was resolved mostly as "working as intended, not a bug" — the AI had genuinely lacked a tool at one point and honestly said so — but surfaced one confirmed hallucination: the AI claimed `searchSongs` could search the 47K discovery catalogue. It can't — `searchSongs` only ever queries the 218-song SOU teaching library; `searchSeedCatalog` is the (unshipped) tool for the discovery catalogue.

**System prompt rework — agreed, not yet written.** The current blanket "always use a database tool, never answer from general knowledge" rule is judged the wrong fix for an older problem (it over-suppresses legitimate brainstorming/discovery). Agreed replacement (not yet drafted into actual prompt wording): general knowledge allowed by default; a real tool lookup required only when asserting a fact specifically about SOU's own data; ask rather than silently assume when a request is ambiguous (e.g. "do we already teach X" — currently active vs. ever taught vs. should we teach it are different questions).

**Unrelated discovery, not yet resolved:** old Deezer-first BPM implementation work has been sitting modified/uncommitted in the backend repo for over a month, untouched since a July session. Needs an explicit decision (commit, keep separate, or discard) — don't let it get accidentally swept into an unrelated commit.

**New standing architectural rule, stated explicitly and applying to all future write-capable Studio tools, not just this feature:**
> A write is considered to have happened only when the backend confirms it. Studio text must never be the source of truth for mutations.

**⚠️ Open structural question, not yet resolved:** `PRODUCT_ROADMAP.md` now documents a **Phase 0 → Phase 1** sequence (Phase 0 = stabilise Studio tool integrity: audit trail, fix STUDIO-01, ship the system prompt rework, validate the two new tools; Phase 1 = PDF/lyric text extraction) that sits **alongside** the September Utility Milestone A/B/C spine (Arrangement → Print/PDF → Lesson Planning) documented elsewhere in this project. It is not yet clear whether Phase 0 blocks Milestone A, runs in parallel with it, or has superseded it as the actual next priority. **Confirm this with Matthew or by re-reading the current `PRODUCT_ROADMAP.md` in full before starting implementation work** — don't assume either sequencing silently.

## Song sheet database landscape (for context, not action)

Multiple versions of the Song Sheets Database exist in Google Drive (MASTER, GPT version, Oct 2025 backup) — these are data, not architecture, and don't need reconciling as part of app-build work unless a specific task calls for it.
