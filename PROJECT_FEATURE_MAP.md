# SCHOOL OF UKE — PRODUCT VISION
*Updated June 2026*

---

## Overview

The School of Uke platform is two distinct products sharing a single database and codebase:

1. **The Tutor Platform** — a creative workspace and teaching design tool for SOU tutors and administrators
2. **The Student Platform** — a music learning resource for students

These have different default experiences, different feature access, and will eventually have different visual personalities. The tutor platform is a professional creative tool. The student platform is a consumer-facing music resource.

---

## Product 1: The Tutor Platform

### Vision

A creative workspace that tutors actively want to visit every day. Not a database management tool. A place for musical exploration, lesson design, and content creation — guided by an AI that knows music deeply and understands SOU's teaching philosophy.

The primary interface is conversation. Everything else — the song library, chord library, theory sheets, TAB sheets, lesson plans — is context the AI can reach into on the tutor's behalf. The experience should feel like having a knowledgeable musical thinking partner available at all times: something like Claude, but with a stronger music brain and the ability to turn conversations into useable songsheets, bespoke theory sheets, TAB sheets, interactive exercises, and lesson plans.

### Interface Model

One codebase, one shell. The same black sidebar and content area serves all roles. Role determines what appears in the sidebar and what is accessible — not a separate app.

**Super Admin** — full sidebar, all sections visible, including database management, user management, and system settings.

**Tutor** — collapsed icon-only sidebar, limited to their own tools. Default landing page on login is the conversation workspace, not a dashboard.

The tutor sidebar (icon-only, collapsible):
```
💬  Conversation (default)
🎵  Song Library
📄  My Creations
📚  Courses & Lessons
👤  My Profile
```

Super Admin sees all of the above plus:
```
🗄️  Manage Database
🔍  Song Discovery
👥  User Management
⚙️  Settings
```

### User Roles

**Super Admin (Matthew)**
Full access. Database management, user management, system settings, all content, all tutors' work, analytics. Can unpublish or remove any content.

**Tutor**
Access to their own workspace only. Can create, draft, proof, and publish content. Cannot access other tutors' content, database management, or system settings.

Content published by a tutor goes through AI proofing before going live. There is no mandatory Super Admin approval gate — tutors publish directly after AI proofing passes. Super Admin can review, edit, or unpublish content at any time.

### The Conversation Workspace

The tutor's default landing page and primary tool. An AI chat interface where the tutor can:

- Explore song ideas and get musical recommendations
- Understand theory in the context of specific songs
- Plan lessons and courses
- Ask the AI to open a songsheet, theory sheet, or TAB sheet template
- Continue working in the split-screen editor alongside the conversation

The AI reads the tutor's profile notes before every conversation — context about their teaching style, current groups, student levels, and musical preferences. Tutors write these notes themselves in plain language, like briefing a new assistant. Each tutor has their own persistent, searchable conversation history.

### The Split-Screen Editor

When a tutor asks the AI to create a sheet, the workspace splits:

- **Left/main panel**: the visual editor showing the sheet being built
- **Right panel**: the ongoing AI conversation

The AI pre-fills the sheet based on the conversation. The tutor edits manually or continues directing the AI. Both modes work simultaneously.

The editor shows the **digital representation** of the sheet. A separate PDF preview mode exists for checking the print layout.

### Two Forms of Every Sheet

**1. Digital form**
Dynamic and interactive. For songsheets this includes:
- Chords above lyrics with beat markers
- Moving BPM cursor
- Chord diagrams on hover/tap
- Chord difficulty/inversion selector
- Key transposition with sharps/flats toggle
- YouTube official video embed for reference playback
- Capo toggle

MVP digital features match Ultimate Guitar as a baseline: autoscroll, speed control, transpose, simplify, capo adjustment, YouTube embed.

**2. Print/PDF form**
One page. Follows the established SOU songsheet format:
- Title, artist, year, key top centre; SOU logo top right
- Chord diagrams top left (2 or 3 columns)
- Lyrics with chord symbols in 2 or 3 columns
- Strum pattern box bottom left
- Optional TAB notation at bottom if space allows
- Speech bubbles for performance notes

Layout decisions are suggested by the AI and confirmed or overridden by the tutor.

### The Songsheet Creation Process (New Workflow)

1. Tutor names the song (or arrives at it through conversation)
2. AI fetches chord/lyric data, pre-fills the digital sheet
3. AI selects appropriate chord diagrams from the library based on chords and difficulty level
4. AI suggests strum pattern based on song data
5. AI embeds YouTube official video for reference playback
6. AI suggests layout for print version
7. Tutor reviews, listens alongside YouTube embed, corrects anything wrong
8. Tutor approves or overrides layout decisions
9. Adds performance notes, TAB reference if needed
10. Saves to draft
11. AI proofing runs — checks musical accuracy, chord correctness, formatting consistency
12. Tutor reviews proofing feedback, revises if needed
13. Publishes to central archive

### Content Types

**Songsheets**
Chord/lyric sheets for teaching. One per song per key. Digital-first with PDF export. The core content type.

**TAB Sheets**
Notation sheets showing melodies, riffs, arpeggios, and picking patterns. Currently created in Guitar Pro, exported as PNG, assembled with SOU branding in the platform. Multiple TAB sheets can exist per song at different difficulty levels (e.g. Beginner/Intermediate, Main Melody/Counter Melody/Bass Line). Future vision: notation created inside the platform itself, either via Guitar Pro integration or a native notation tool.

**Theory Sheets — three types:**

*Standalone theory sheets* — concept-first, not tied to a specific song. Form the backbone of the structured theory curriculum. Examples: "How Music Works: Circle of 5ths", "Intro to Pentatonic Scale".

*Song-annexed theory sheets* — attached to a specific song, emerging from it. Examples: "The chord sequence in Redemption Song and how it relates to I-IV-V", "The Dorian mode in Scarborough Fair".

*Thematic/bridging sheets* — branch off from a concept or song into adjacent territory. Create a web of connected content rather than a linear curriculum.

All theory sheets are tagged for searchability (concept, level, genre, scale, mood, instrument etc). Tags are the primary navigation mechanism for the theory database.

### Content Ownership and Publishing Flow

```
Tutor creates content → saved as draft in tutor's profile
        ↓
Draft is private — not accessible to students or other tutors
        ↓
AI proofing (musical accuracy, chord correctness,
curriculum consistency, formatting)
        ↓
Tutor reviews AI feedback, revises if needed
        ↓
Tutor publishes → content goes live in central archive
        ↓
Super Admin can review, edit, or unpublish at any time
```

Content is company IP. Tutor is credited on all published work. Unpublished drafts cannot be accessed digitally or exported as PDF.

### The Chord Library

194 PNG files (and growing), named by fret position code (e.g. `0432.png`, `1122_Barre.png`). Each voicing can represent multiple chord names depending on context.

Needs to be built as a proper database:
- All PNGs uploaded to Cloudflare R2
- AI vision model identifies fret positions and all chord names each voicing represents
- Human review pass to verify
- Each record: fret position code, chord names array, difficulty rating, finger count, R2 URL

Powers: automatic chord diagram selection in the songsheet creator, chord diagrams on hover in the digital songsheet, difficulty-based voicing suggestions, future transposition tools.

---

## Product 2: The Student Platform

### Vision

A sophisticated music learning resource. Not a lesson delivery system. A rich reference tool that students browse between lessons, use to practice, and return to as their playing develops. Should feel closer to Spotify or a music magazine than a school portal — consumer-facing, culturally alive, visually distinct from the tutor platform.

### Core Features

**Songfinder**
Primary tool. Sophisticated search and filter across the full SOU song library. Filter by key, genre, era, level, season, mood, tags. The current tutor-facing song browser is the technical starting point — the student version will be more visually refined and consumer-facing.

**Digital Materials**
Each song has associated published materials — the interactive digital songsheet, TAB sheets, annexed theory sheets. Students access these from the songfinder. Only published content is accessible.

**Theory Database**
All published theory sheets searchable by tag, title, concept, level, or instrument. A student can browse by concept ("show me everything about pentatonic scales") or by song ("show me theory sheets connected to Redemption Song").

**Student Profile**
Saved songs and theory sheets. Assigned materials from tutor (future). Lesson history (future).

---

## What Is Not Yet Designed

- Student profile and tutor-student relationship management
- Course and lesson planning tools
- Corporate workshop booking integration
- Tiered access / membership model
- Multi-instrument expansion beyond ukulele
- Mobile experience for students
- Interactive exercises and play-along features beyond BPM cursor
- Native notation creation tool (currently Guitar Pro → PNG workflow)

## Known Follow-Up: Multi-Tutor Auth Migration

**Current state (June 2026):** Auth uses a single shared `ADMIN_PASSWORD` env var (bcrypt-hashed via `auth.js`). The `tutors` table exists in the database (seeded with Matthew as `id=1, role=super_admin`) but is not yet connected to the login flow. `tutor_id=1` is hardcoded throughout the chat feature for now.

**What needs to be built when onboarding additional tutors:**
- Per-account login: `POST /api/auth/login` validates against `tutors.password_hash` (bcrypt) instead of the shared `ADMIN_PASSWORD`
- Session stores `tutor_id` (replacing the current boolean "is authenticated" check)
- All chat API endpoints read `req.session.tutorId` to scope conversations and profiles
- Admin UI: user management page for Super Admin to create/edit tutor accounts
- Retire `ADMIN_PASSWORD` env var once all admins have individual accounts

**Trigger:** Do this when the second tutor account is needed — not before.

---

## Known Follow-Up: Conversation as Default Landing Page

**Current state (June 2026):** The Conversation feature is a regular nav item in the admin sidebar alongside Dashboard, Song Discovery, etc.

**Desired future state:** Conversation should be the default landing page for tutors (i.e., `activePage` initialises to `'conversation'`). This was deliberately deferred because it belongs to the same work item as role-based sidebar visibility — once tutors and super_admins have different sidebar menus, the landing page should also differ by role.

**What needs to change:**
- Role-based sidebar: tutors see Conversation, super_admins see the full menu (or vice versa as designed)
- `activePage` initial state in `AdminDashboard.js` should default to `'conversation'` for tutors
- The current `'dashboard'` default is fine for super_admin

**Trigger:** Do this alongside the Multi-Tutor Auth Migration (above) — the role information needed to determine the landing page is the same as what's needed for sidebar gating.

---

## Next Session Priority: Wire Rich Enrichment Pipeline Into Live App

> **Priority: HIGH — this is the clearest remaining gap before promoted songs are truly useful in the app.**

### The Two-Pipeline Finding (discovered 12 June 2026)

There are two completely separate enrichment implementations with no connection to each other:

**Pipeline A — `enrichmentService.js`** (lightweight, runs automatically on promote)
- Called by `POST /api/seed/promote`
- Input: `(title, artist)` strings
- Sources: GetSongBPM (BPM/key/mode/timeSignature) + Spotify (trackId/year/genre) + YouTube (videoId)
- Automatic: ✅ runs on every promote

**Pipeline B — `enrichmentService_sqlite.js`** (rich, CLI-only)
- Called only by `node batchEnrichAll.js` (offline script, not wired to any HTTP endpoint)
- Input: `songId` (database row ID)
- Sources: Wikipedia (intro/charts/songwriters/release date) + Last.fm (play counts/listeners/tags) + GetSongBPM + Spotify + Soundcharts
- Automatic: ❌ **never runs in the live app**

This means every promoted song is missing: Wikipedia intro, Last.fm play counts, chart positions, cover art waterfall, and rich release metadata — until Pipeline B is manually run from the CLI.

---

### Concrete Next Steps

**Step 1 (prerequisite): Confirm Railway environment variables**

The following keys are set in the local `.env` but their Railway status is unconfirmed. Verify in Railway → Variables before any enrichment testing on production:

| Variable | Used for | Local status |
|---|---|---|
| `YOUTUBE_API_KEY` | YouTube video ID on promote | ✅ set locally |
| `LASTFM_API_KEY` | Last.fm play counts/listeners | ✅ set locally |
| `SOUNDCHARTS_APP_ID` | Chart positions (fallback) | ✅ set locally |
| `SOUNDCHARTS_API_KEY` | Chart positions (fallback) | ✅ set locally |
| `MUSICBRAINZ_USER_AGENT` | Release date fallback | ✅ set locally |

If `YOUTUBE_API_KEY` is missing in Railway, every promoted song will show "YouTube: (pending)" regardless of code.

**Step 2: Wire Pipeline B into the live app**

Two approaches — decide at start of next session:

**Option A: "Enrich Song" admin button (on-demand, recommended)**
- Add `POST /api/songs/:id/enrich` endpoint backed by `enrichmentService_sqlite.enrichSong(songId)`
- Add an "Enrich" button in the song detail modal (admin-only, same auth guard as edit/delete)
- Pros: no impact on promote flow performance; admin can trigger when needed; surgical per-song; observable (returns enrichment results)
- Cons: manual step — newly promoted songs show blank Wikipedia/Last.fm until triggered

**Option B: Automatic post-promote background step**
- In `routes/seed.js` promote handler, after `dbManager.addSong()` succeeds, fire `enrichmentService_sqlite.enrichSong(newSong.id)` in a `setImmediate` block (don't await)
- Pros: promoted songs auto-enrich; no admin action needed
- Cons: enrichment takes 10–30 seconds per song (multiple API calls); silent failures go unnoticed; adds Railway API quota on every promote; Pipeline B is not yet verified stable in Railway
- Note: Pipeline B takes a song ID, not title/artist — call signature already correct

**Recommendation:** Start with Option A — lower risk, observable, re-triggerable if an API was temporarily down. Option B can be layered on once Pipeline B is confirmed stable.

**Step 3: Test with "Into the Groove" by Madonna**

This song is already in the teaching DB (`id: madonna_into_the_groove`, promoted 12 June 2026). It has title + artist + Spotify track ID + year, but null BPM/key (not in GetSongBPM DB), null YouTube, null Last.fm, null Wikipedia. It's the ideal test case once the enrichment endpoint is wired.

---

### What Is NOT a Gap: "No learning materials available yet"

This message in the song detail modal is **correct and expected** for promoted songs. Materials (song sheets, melody tabs) come from the physical PDF pipeline — `scanMaterialsToDatabase.js` scans a local materials folder and stores paths in `materials_json`/`song_sheet_path`/`melody_tab_path`. These PDFs only exist for the original 215 SOU teaching songs, created manually by Matthew in Guitar Pro and assembled with SOU branding. Newly promoted songs will always show this message until Matthew manually creates and uploads the sheet. This is a teaching-workflow step, not an enrichment gap. **No code change needed.**

---

## Standing Principle: Enrichment Must Never Overwrite Non-Null Fields With Null

**Established: 13 June 2026**

All enrichment code (both pipelines) must follow this rule:

> If a field already has a non-null value in the DB — from manual entry, the original SOU CSV, or a previous enrichment run — and the new API call returns null/empty for that field, **the existing value must be preserved**.

### What is implemented (as of 13 June 2026)

| Layer | Protection | Notes |
|---|---|---|
| `dbManager.updateSong()` | ✅ Strips `null`/`undefined` from updates before building SQL | Defence-in-depth — no caller can accidentally null a field |
| `enrichmentService_sqlite.enrichSong()` | ✅ Never adds null to `enrichmentData` (only writes when API returns actual data) | Additionally guards: `key_best`, `mode`, `time_signature_best`, `wikipedia_background`, `wikipedia_composition`, `wikipedia_recording`, `wikipedia_reception`, `bpm_best` (all checked against `!song.field`) |
| Our backfill scripts | ✅ Build update objects with `if (enriched.X)` guards | Never pass null values |

### What is intentionally "always overwrite" (fresh metrics)

- `lastfm_plays`, `lastfm_listeners` — always updated on re-enrich (stale counts are wrong)
- `chart_peak_*` from Wikipedia — updated if a lower peak is found (better data wins)

### What to check when adding new enrichment fields

Before adding a new field to `enrichmentData` in any enrichment service:
1. Should this field survive re-enrichment? → Add `!song.field_name` guard
2. Is this a "freshness" metric (play counts, timestamps)? → No guard needed
3. Is this a "best-wins" numeric? → Guard with `!song.field || newValue < song.field`

### Admin edit form

`PUT /api/songs/:id` passes `req.body` directly to `updateSong`. The null-strip in `updateSong` means cleared fields (sent as `null`) are silently ignored. If an admin genuinely needs to blank a field, they must send `""` (empty string) rather than `null`. This is a known trade-off — document in admin UI if it becomes a problem.

---

## Known Follow-Up: Deezer as Primary BPM Source

**Current state (June 2026):** The promote-song enrichment waterfall (`enrichmentService.js`) now uses GetSongBPM (`api.getsong.co`) as the primary BPM/key/mode/time-signature source, with Spotify for identity only (spotifyTrackId, year, genre). Spotify audio-features was dropped — it requires OAuth and returns 403 under Client Credentials flow (restricted mid-2025).

**Limitation:** GetSongBPM has incomplete catalog coverage. Songs not in its database get null BPM/key after promote (e.g. "Into the Groove" by Madonna). This is a data coverage gap, not a code bug.

**Desired future state:** Add Deezer as the primary BPM source (tested/working as of mid-2025 notes). Deezer's track endpoint returns BPM natively and has broader catalog coverage. GetSongBPM/TuneBat/Cyanite remain as specialist secondaries.

**What needs to be built:**
- `getDeezerData(title, artist)` method in `enrichmentService.js` — search via `https://api.deezer.com/search?q={artist}+{title}` (no auth needed for basic search), then fetch `https://api.deezer.com/track/{id}` for BPM field
- Move Deezer call first in the waterfall, before GetSongBPM
- No new API key needed — Deezer basic search is unauthenticated

**Trigger:** Next enrichment-quality session. Low-effort, high-impact for songs missing BPM.

---

## Architectural Decision: Chat as Persistent Overlay ("Studio" Concept)

> **Status: Decided, not yet implemented. Resolve open questions before starting layout work.**

### Reference UX Model: VS Code + Copilot Panel

The target layout pattern is **VS Code with the Copilot chat panel open**:

- The **chat panel** (Copilot) is dockable and collapsible — a persistent side panel that doesn't replace the main area
- The **main content area** (editor) is what the chat acts on: it can populate it, open files into it, navigate it, or operate alongside it
- The two panes are independent but coupled — collapsing the chat doesn't close or lose the main content; opening the chat doesn't navigate away from it
- The chat can reference and modify what's in the main area ("here's the song I found — want me to open the sheet?")

This is the exact spatial relationship and interaction model to implement in next session's planning. When designing the Studio layout, if the answer to a layout question isn't obvious, ask: "what would VS Code do here?"

### Decision

The current "Conversation" nav item (a standalone full-page component) will be superseded by a **persistent chat panel docked to the admin layout** — available on every admin page without navigating away. The chat panel can drive the main content area: opening songsheets, theory sheets, the song database, TAB sheets, or other content either behind it or alongside it (split-screen).

The word **"Studio"** describes this combined experience — the persistent chat plus the content area it controls — not a new page or nav item. The current `'conversation'` nav item should **not** be renamed yet; it will be superseded by the new layout, not renamed.

### What this changes

- `ConversationWorkspace` moves from a routed page to a panel component inside `AdminLayout`
- The panel is collapsible/dockable (collapsed = a floating button or narrow strip; expanded = full panel alongside content)
- When the AI references a song or sheet, it can open that content in the main area next to the chat
- The sidebar nav items drive the main content area as before; the chat panel is a layer on top of, not replacing, that navigation

### Open Questions (resolve next session before implementing)

1. **Conversation continuity:** Does the chat maintain one continuous conversation regardless of which admin page is open (i.e., context-less persistent history)? Or does it spawn per-context conversations (e.g., a different thread when Song Database is open vs. when a songsheet editor is open)?

2. **Role-based toolsets:** Tutors get Studio-scoped tools (`searchSongs`, songsheet tools, theory tools). Matthew (super_admin) potentially gets broader admin tools too (bulk DB edits, enrichment triggers, etc.) — same chat UI, different available tool declarations in `aiProvider.js`. How is the active toolset determined? By role in session, or by which page is open?

3. **Layout mechanics:** Where exactly does the panel live in `AdminLayout`? Options:
   - Right sidebar (fixed width, collapsible)
   - Bottom drawer (full-width, collapses to a tab)
   - Floating overlay (draggable, resizable)
   Which mode is default? Can the user switch modes?

4. **Content opening:** When AI suggests "here are your Bb songs" and the user wants to see one — does clicking the song open it in a split-pane inside the Studio, or does it navigate the main area (replacing current page)?

### Why this is a precursor to split-screen editor work

The split-screen songsheet editor (see `### The Split-Screen Editor` above) assumes a specific layout for how the main content area is structured. That design depends on knowing whether the chat panel is a sibling or a parent of the content area. Resolving the Studio layout questions first prevents having to redesign the editor frame after the fact.

### What NOT to do before resolving open questions

- Do not move `ConversationWorkspace` out of its current page route
- Do not add a persistent panel stub to `AdminLayout`
- Do not rename the `'conversation'` nav item
- Do not start split-screen editor layout work

---

## Key Principles

- **Conversation is the primary interface** for tutors — not menus and forms
- **The AI executes, not just advises** — every suggestion should have a one-click action
- **Digital-first, PDF as export** — the songsheet creator builds the interactive version; PDF is derived from it
- **One codebase, role-based access** — same shell for all users, sidebar and features determined by role
- **Company owns content, tutor is credited** — no content lives only on a tutor's device
- **Nothing unpublished is accessible** — draft content is invisible to students and other tutors
- **Tutors publish directly** after AI proofing — no mandatory Super Admin approval gate
- **Architecture before visual design** — get the data model and user flows right before investing in visual polish

---

## Core Modules & Features

### 1. Tutor UI (Instructor-facing features)
**Song Management:**
- Song Database access (Song Adder, Song Finder)
- Easy digital song sheet creator with print option
- Music Theory Internal database and Finder (~30 SOU Music Theory PDFs)
- Sheet Music Plus PASS Access integration (read-only for searching/viewing)

**Lesson Planning:**
- Trello-style drag and drop post-it notes for tutors to manage lessons
  - *Implementation TBD: persistence, templates, reusability across students/groups*
- Tutor Lesson Planning & Management Center
- Chord Finder tool (based on 'Chord!' app functionality)

**Student Management:**
- Students tracking
- Groups management
- Lesson Plans creation and management
- Lesson Appraisals & Student Progress tracking

**Integrations:**
- Sync with Sesame/Shopify Booking System or develop custom Shopify Booking Plugin
  - Required features: recurring lessons, package deals, waitlists
  - *Deep dive needed on current Sesame/Shopify integration*

---

### 2. Admin UI (Backend management)
**Database Management:**
- Bulk Import Database Search & 3rd Party API Query for expanding song database
- Song Management Centre
- Song Database Manual Song Data Editor
- Music Theory Sheets Database & Editor

**User Management:**
- Tutor Profile editor
- User access control and permissions

---

### 3. End-user UI (Student/public-facing features)
**Discovery & Search:**
- Songfinder tool
- Music Theory Finder (similar to Tutor song database: SOU PDF archive & search)
- Chord Finder (based on 'Chord!' app functionality)

**Content Access:**
- Digital Dynamic Songsheets & TABs with export/print options
  - Transpose keys on-the-fly
  - Adjust tempo markers
  - Customize chord diagrams
  - Audio playback for play-along functionality
  - Chord and lyrics tracker that moves with selected BPM
- Curated Digital Lesson Programs

**Membership & Booking:**
- Tiered Membership accounts with access to different features
  - Free: Browse only
  - Basic: Limited sheets access
  - Premium: Full access + lessons + booking
- Access to online & in-person lesson bookings

---

### 4. Genius Bar & Forum (Community support)
**Tools & Resources:**
- Music Theory Finder
- Chord Finder (based on 'Chord!' app functionality)
- Community forum for Q&A and discussions
- **Integration:** Built into SOU app (not separate platform)
- *Moderation requirements TBD*

---

## Technical Integration Points

1. **Song Database** - Central data store for all song metadata, sheets, and related content
2. **Music Theory Database** - Internal resource for ~30 SOU Music Theory PDFs (search functionality TBD based on collection growth)
3. **Sheet Music Plus** - Third-party integration (read-only) for searching/viewing expanded sheet music
4. **Booking System** - Sesame/Shopify integration or custom plugin for lesson scheduling
   - Must support: recurring lessons, package deals, waitlists
5. **Membership System** - Tiered access control across all UIs (Free/Basic/Premium)
6. **Chord Finder** - Based on 'Chord!' app - requires deep dive into existing codebase
7. **Audio Playback** - For dynamic songsheets with BPM-synced chord/lyrics tracker

---

## Research & Deep Dive Tasks

1. **'Chord!' App Analysis** - Reverse-engineer or study chord finder functionality for integration
2. **Sesame/Shopify Integration Review** - Assess current booking system integration patterns
3. **Lesson Planning UI/UX** - Define Trello-style interface behavior (persistence, templates, reusability)
4. **Music Theory Search** - Design search functionality appropriate for small PDF collection (may expand later)
5. **Dynamic Songsheet Audio** - Research audio generation/playback solutions for chord progressions

---

## Development Priorities & Roadmap

### Phase 1: Core Infrastructure (Current)
- Song Database with bulk import and API expansion
- Admin UI for song and user management
- Basic Songfinder and Chord Finder tools

### Phase 2: Tutor Features
- Lesson Planning & Management Center
- Student tracking and progress monitoring
- Integration with booking systems

### Phase 3: End-user Experience
- Dynamic songsheets with export/print
- Curated lesson programs
- Tiered membership implementation

### Phase 4: Community & Advanced Features
- Genius Bar & Forum
- Music Theory Finder expansion
- Advanced lesson planning tools

---

## Notes
- This map is a living document. Add new features and refinements as the project evolves.
- Reference this file for architecture, feature planning, and cross-team communication.
- All feature development should consider cross-UI integration points.

---

## Version History
- **v1.0** (Nov 21, 2025): Initial feature map transcribed from architecture diagram
- Future updates will be tracked here

---

## Source
This summary was transcribed and expanded from the provided SOU Platform/App feature chart image (Nov 2025).
