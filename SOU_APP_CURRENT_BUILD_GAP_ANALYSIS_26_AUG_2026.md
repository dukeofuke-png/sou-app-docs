# SOU APP — CURRENT BUILD GAP ANALYSIS
**Date:** 26 August 2026  
**Purpose:** Compare the resolved August 2026 Product/UX specification with the actual current repo/database state, then identify the shortest coherent September Utility path before changing implementation sequence or beginning major new work.

---

## 1. Executive conclusion

The current SOU App is **not a blank slate**, but it is still structurally much closer to a **Song Catalog + Admin + AI Chat application** than to the resolved Tutor Studio product.

The current build already has valuable foundations:

- a working public SOU song catalog
- a mature Admin song-database interface
- a 47K+ discovery/seed catalog
- rich song metadata/enrichment
- Cloudflare R2 PDF storage
- authentication for the Admin surface
- a persistent AI conversation system
- SA tool execution for teaching-library search, discovery search and song promotion
- an audit trail for AI tool calls
- a minimal tutor/profile data model
- production deployment architecture

However, almost all of the **teaching-operating model** resolved during the four-day Q&A is still absent from the database and UI:

- no Tutor Studio product shell
- no Courses
- no Course Plans
- no Lessons
- no full/short-form Lesson Plans
- no Registers
- no Appraisals
- no Actions
- no Notifications
- no 1:1 Student Journeys
- no native Arrangement object
- no Song Canvas
- no Theory objects
- no standalone TAB publication model
- no native PDF-generation/archive workflow
- no Teaching Mode
- no contextual object relationships for Chats/SA beyond the current tutor conversation

This means the **full resolved Tutor Studio MVP remains a substantial build**.

But the gap analysis also shows a plausible much shorter internal-use path: build forward from the strongest thing the app already understands, the **Song**, rather than first attempting to build the entire Tutor operating system.

The likely shortest September Utility spine is:

**existing Song Catalog → native Arrangement object → simple Song Canvas → A4 Print View/PDF publication → then lightweight Course/Lesson Planning integration**

This path can create real operational value before the wider Tutor Studio is complete.

---

# 2. Evidence / confidence

This analysis is based on:

1. the current Google Drive repo copy uploaded 25–26 August 2026;
2. direct inspection of current frontend/backend source files;
3. direct inspection of the current SQLite database;
4. `MASTER_ARCHITECTURE.md`;
5. `PRODUCT_ROADMAP.md`;
6. `SOU_APP_PRODUCT_UX_SPEC_CONSOLIDATED_25_AUG_2026.md`.

The Google Drive desktop client is still reconciling/syncing generated folders, but the core source files required for this assessment are present and readable.

The analysis does **not** treat absence from Google Drive as proof of absence unless corroborated by the current database/code/docs.

---

# 3. Current application shape

## 3.1 Frontend

Current frontend is a Create React App application.

The public side is primarily a song browser with:

- text search
- level filter
- key filter
- mode
- era
- season
- genre
- chart filters
- song detail modal
- PDF access

The Admin route is effectively a separate internal application shell with:

- Dashboard
- Seed Database Search
- Bulk Add Songs
- Conversation / Studio Chat
- Manage SOU Database
- Settings placeholder

The Admin navigation is state-based rather than a mature multi-domain routing architecture.

### Important implication

There is **no current Tutor Studio shell** to evolve incrementally screen-by-screen. Tutor Studio will be a meaningful new product surface, although existing React components can be reused.

---

## 3.2 Backend

Current backend is an Express service backed by SQLite.

Core areas include:

- song CRUD
- seed/discovery search and promotion
- song enrichment
- auth
- PDF retrieval/links
- chat/conversations
- AI provider/tool execution
- tool-call audit
- duplicate checking/import
- chart/popularity/enrichment services

This backend is usable as the base for expansion, but much of it remains organized around the `songs` domain.

---

## 3.3 Current SQLite data model

Direct inspection of the current database shows these product-relevant tables:

- `songs` — 218 records
- `tutors` — 1 record
- `tutor_profiles` — 1 record
- `conversations` — 12 records
- `messages` — 56 records
- `tool_calls` — 7 records

There are no current tables for:

- students
- courses
- course_tutors / teaching relationships
- course plans
- lessons
- lesson plans
- lesson plan versions
- appraisals
- registers / attendance
- actions
- notifications
- playlists
- arrangements
- arrangement sections
- chord/lyric content
- theory resources
- exercise resources
- TAB resources
- publications / PDF archive objects
- media/recordings
- chat-object relationships
- projects
- SA Insight tags/relationships
- Journey Plans / milestones

### Important implication

The four-day design exercise is not simply describing missing screens. It introduces a **large new relational teaching/content model**.

---

# 4. What is genuinely reusable

## 4.1 Song identity + metadata layer — STRONG FOUNDATION

The current `songs` table is rich and mature.

It already contains:

- title / artist
- release dates
- BPM
- key
- time signature
- level
- SOU teaching keys
- genre/tags
- teaching notes
- Spotify / YouTube references
- cover art
- PDF/TAB presence/status
- R2 URLs
- enrichment provenance

This should remain the canonical **Song identity/catalog layer**.

The new Arrangement system should be added **beside** this model, not replace it.

---

## 4.2 Public Catalog — REUSABLE

The current public catalog/filtering UI is already useful and should be retained while Tutor/Student interfaces evolve.

It can later become one surface onto the same shared content model.

No need to rebuild this immediately for September Utility.

---

## 4.3 Admin database management — REUSABLE

`ManageSOUDatabase` is already a substantial internal tool:

- 30+ fields
- search
- sorting
- inline editing
- bulk edit
- saved views
- enrichment
- R2 PDF awareness

The resolved design deliberately allows Admin UX to remain basic.

Therefore this should **not** be replaced as part of the immediate Tutor build.

---

## 4.4 Seed/discovery catalog — REUSABLE

The 47K+ discovery catalog plus search/promote flow is already present.

This maps well to the resolved Song Studio flow:

**Find Song → select existing Song / promote discovery Song → create Arrangement**

This is a strong reason to build Arrangement creation from the existing Song domain rather than invent a separate content-ingestion system.

---

## 4.5 AI Conversation engine — STRONG BUT NARROW FOUNDATION

Current chat already supports:

- persisted conversations
- conversation history
- tutor profile context notes
- Gemini/Anthropic provider abstraction
- full message history
- `searchSongs`
- `searchSeedCatalog`
- `promoteSongFromSeed`
- tool-call audit
- guard against raw tool JSON leaking into chat

This is a meaningful foundation for SA.

But current chat is:

- scoped to hardcoded `tutor_id = 1`
- not related to Courses/Students/Lessons/Arrangements
- not omnipresent in the broader app
- not object-context aware
- not Project aware
- not SA Insight aware
- not capable of Actions/Appraisal processing

### Conclusion

Do **not** rebuild Chat from scratch.

Instead, expand its relationship/context model as new objects are introduced.

---

## 4.6 Tutor model — SKELETON ONLY

The database already has:

- `tutors`
- `tutor_profiles`
- `role`
- context notes

But:

- only Matthew exists
- Chat is hardcoded to tutor ID 1
- current production auth is still effectively Admin-centric
- session storage is in-memory
- multi-tutor authentication/permissions are not implemented

This is useful future-proofing, not a functional Tutor account system yet.

---

## 4.7 PDF infrastructure — READ SIDE EXISTS, WRITE/PUBLICATION SIDE DOES NOT

Existing SOU PDFs are already hosted on Cloudflare R2 and linked from Song records.

The current app can:

- know whether a Songsheet/TAB exists
- expose R2 links
- fetch/serve PDFs

But the current app does **not** yet have a native workflow to:

- generate a Songsheet PDF from an Arrangement
- upload the generated PDF to R2 from the app
- create a persistent publication/archive record
- manage multiple explicitly-created key PDFs
- create searchable standalone TAB-sheet PDF records
- maintain publication history independently of the Song row

### Conclusion

R2 storage is a foundation, but the new Print/PDF publication workflow is still a build.

---

# 5. Resolved-design gap matrix

| Resolved area | Current state | Gap |
|---|---|---|
| Public Song Catalog | Built | Low |
| Admin Song DB | Built | Low |
| Seed/Discovery Catalog | Built | Low |
| SA Chat | Built, narrow | Medium |
| Tool-call audit | Built | Low |
| Multi-tutor accounts | Skeleton | High |
| Tutor Studio Home | Absent | High |
| Music Inspiration | Absent | Deferred/High |
| Chat-object relationships | Absent | High |
| Projects | Absent | Medium |
| Actions | Absent | High |
| Notifications | Absent | High |
| Courses | Absent | High |
| Course Plan | Absent | High |
| Lessons | Absent | High |
| Full Lesson Plan | Absent | High |
| Short-form Plan / Prompt Sheet | Absent | High |
| Lesson Hub | Absent | High |
| Teaching Mode | Absent | High |
| Registers | Absent | High |
| Lesson Appraisal | Absent | High |
| Course Appraisal | Absent | High |
| Institutional teaching memory | Absent | High |
| 1:1 Student Journey | Absent | High / not September-critical |
| Arrangement object | Absent | **Critical September gap** |
| Song Canvas | Absent | **Critical September gap** |
| Dynamic transposition | Absent | Medium / can phase |
| Reference audio workspace | Absent | Medium / can phase |
| Beats | Absent | Medium / can phase |
| Theory Studio | Absent | Medium |
| Exercise/Game | Absent | Deferred |
| TAB image component workflow | Absent | Medium |
| Native Print View | Absent | **Critical September gap** |
| Generated PDF archive | Legacy PDFs exist; generation absent | **Critical September gap** |
| Student UX | Existing public Catalog only | Explicitly deferred |
| Polished Admin UX | Basic Admin built | Explicitly deferred |
| Full mobile Creative Studio | Absent | Explicitly deferred |
| Offline mode | Absent | Explicitly deferred |

---

# 6. Main architectural work still required

## 6.1 New relational object layer

The largest foundational gap is the new object model.

The app needs a migration path from essentially:

`song + chat`

to:

`song → arrangement → publication`

and later:

`course → lesson → plan → appraisal → actions`

plus:

`student → journey → lesson → appraisal`

The danger would be attempting to add all of these at once.

They should be introduced in coherent vertical slices.

---

## 6.2 Object relationships / contextual SA

The resolved UX depends heavily on SA being able to know:

- which Course
- which Lesson
- which Student
- which Arrangement
- which Appraisal
- which Action
- which Project

a conversation relates to.

Current conversations contain only:

`conversation → tutor`

Therefore a generalized relationship/context mechanism will eventually be needed.

This is architecturally important, but it does **not** have to be fully built before the first native Arrangement utility slice.

---

## 6.3 Authentication / permissions

The resolved Tutor product assumes:

- multiple Tutor accounts
- Lead / Assistant / Shared Lead / Substitute relationships
- read/edit distinctions
- context-dependent permissions
- Admin override

Current auth does not yet provide this.

This should be built before wider Tutor rollout, but it is **not required to let Matthew use an internal September build** if the first utility milestone remains single-user/internal.

---

# 7. September Utility — shortest coherent path

## 7.1 What September Utility should NOT attempt first

Do not begin by building:

- full Tutor Home
- full multi-tutor permissions
- Student UX
- Notifications
- complete Actions system
- Teaching Mode
- Music Inspiration
- 1:1 Journeys
- white-label foundations beyond cheap schema hygiene
- sophisticated audio playback
- Beats
- full Theory Studio

Those are product-destination work, not the shortest route to internal operational payoff.

---

## 7.2 Recommended September Utility spine

### UTILITY MILESTONE A — Native Arrangement + Print/PDF

Build the smallest system in which Matthew can:

1. find/open an existing Song
2. create an Arrangement linked to that Song
3. edit a native structured song body
4. save/reopen the Arrangement
5. create a one-page A4 landscape Print View
6. export/store a persistent PDF publication
7. reopen/edit the native Arrangement later

This is closely aligned with the old Roadmap Phase 2 objective, but the newly-resolved product model makes two important changes:

- the object should be called an **Arrangement**, not merely a Songsheet;
- the saved PDF should remain a **real publication/archive artefact** for the current MVP, not merely an ephemeral generated view.

### Why this is the best first utility slice

It attaches directly to the strongest existing domain: **Song**.

It does not require Courses, Students, notifications, registers or complex permissions first.

It also stops new SOU teaching content from being born solely in Keynote/PDF form.

That produces immediate operational return and reduces future migration.

---

### UTILITY MILESTONE B — Lightweight Lesson Planning

After Arrangement creation is usable, add the smallest Course/Lesson planning spine:

- Course
- Lesson
- Full Lesson Plan
- linked Arrangements/resources
- SA-assisted planning
- Short-form Plan / Prompt Sheet generation

Do **not** initially require:

- full Tutor Home
- Teaching Mode
- Actions
- Registers
- Appraisals
- multi-tutor role complexity

This gives Matthew an internal real-world workflow:

**Course → upcoming Lesson → plan → attach teaching content → generate Prompt Sheet**

At this point the app can begin reducing weekly lesson-preparation overhead materially.

---

### UTILITY MILESTONE C — Close the feedback loop

Once real Lesson Planning is being used:

- Register
- bullet Appraisal
- SA processing
- next-lesson context

This is the point where the app begins accumulating genuine teaching memory rather than merely helping create materials.

---

# 8. What this says about the “few days from useful?” question

The full resolved Tutor Studio is **not** a few-days build from the current application.

There is too much missing relational/product infrastructure.

However, the current build is sufficiently mature that a **first genuinely useful internal slice does not require building the full Tutor Studio**.

The strongest evidence-backed conclusion is:

> **First internal utility is plausible much sooner than full MVP, provided the build is aggressively vertical and single-user/internal at first.**

The first utility target should be **Arrangement production replacement**, not “Tutor Studio complete”.

Exact elapsed time should still be estimated by Claude/Copilot against the local repo before committing to a September promise.

---

# 9. Roadmap implications

The old roadmap spine was:

`Studio stabilisation → PDF extraction → Digital Songsheet Builder → Curriculum/Lesson integration → Pedagogy → Appraisal`

The new design and current-build analysis suggest this should be reconsidered, not blindly discarded.

### Likely adjustment

**Legacy PDF extraction may no longer need to block the first September utility slice.**

Why:

- extraction solves understanding of historical PDFs;
- Arrangement Builder solves creation of new native content;
- September Utility is about stopping further print-only accumulation and improving current work.

Therefore a plausible revised operational spine is:

`stabilise current build`
→ `Arrangement data model + Builder`
→ `Print/PDF publication`
→ `minimal Course/Lesson Planning`
→ `legacy PDF extraction in parallel / immediately after`
→ `Appraisal/teaching memory`

This is a sequencing proposal for review, **not yet a canonical Roadmap update**.

---

# 10. Important technical cautions before implementation prompts

Before asking Claude to implement the first slice, require a short architecture proposal covering:

1. new `arrangements` schema;
2. section/content representation;
3. relationship to `songs`;
4. draft/published status;
5. print-layout representation;
6. PDF publication/archive representation;
7. R2 upload/write path;
8. how current legacy PDF fields coexist during migration;
9. how future Tutor ownership/attribution is preserved even if first build is single-user;
10. whether current CRA/Express structure can support the editor cleanly without premature framework migration.

Do not code the new schema until this proposal has been reviewed.

---

# 11. Repo-sync observation

The top-level generated folders were removed from the Google Drive copy, but additional generated folders still exist inside child projects, including nested:

- `sou-song-browser/node_modules`
- `sou-song-browser/.git`
- `sou-song-browser/build`
- `materials-server/node_modules`
- `materials-server/.git`
- `materials-server/coverage`

These may explain continued Google Drive syncing.

They are **not required** for the product gap analysis.

Core frontend/backend source files and the current SQLite database are readable, so analysis is no longer blocked by the Drive spinner.

---

# 12. Recommended immediate next decision

Do **not** start implementation yet.

The next step should be:

### September Utility Review

Review this gap analysis through the Business Manager lens and decide whether the first implementation target should be:

**A. Native Arrangement + Print/PDF production replacement**  
or  
**B. another narrower internal utility slice**

My recommendation from the code evidence is **A**.

Once that is confirmed, the next technical step is a tightly scoped **Arrangement Builder architecture proposal**, not implementation code.

---

## Definition of Done for this gap-analysis phase

This phase is complete when Matthew can answer:

1. What does the current build actually contain?
2. What major resolved product systems are absent?
3. What existing foundations can be reused?
4. What is the shortest coherent internal-use path?
5. What must be designed technically before Claude is allowed to implement it?

This document provides those answers sufficiently to support the September Utility decision.
