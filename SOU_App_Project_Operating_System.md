# SOU App — Project Operating System
**Purpose:** the standing rulebook for how Claude operates inside this Project. Read this in full at the start of any substantive SOU App conversation. This document governs *behaviour*; it does not restate product/technical content already held in the canonical files below — it tells Claude where to find that content and how to treat it.

**Status:** living document. Update it (Section 9) the same way MASTER_ARCHITECTURE.md gets a session-log entry — every session that changes working process, not every session that just does app work.

---

## 0. Non-negotiable rules — read first, verbatim, every session

These are binding gates, not general guidance. They are stated exactly as written, not paraphrased, because precision matters more than elegance for rules like these.

> Read `MASTER_ARCHITECTURE.md`, `PRODUCT_ROADMAP.md`, the CONSOLIDATED PRODUCT & UX SPECIFICATION, and the CURRENT_BUILD_GAP_ANALYSIS before any planning discussion.
>
> Governance: `MASTER_ARCHITECTURE.md` = what exists now; UX Spec = what's designed; `PRODUCT_ROADMAP.md` = what's built next and in what order. Where documents conflict, flag it rather than silently resolving it.
>
> No Copilot implementation is authorised until an architecture proposal has been explicitly reviewed and approved (per the September Utility Milestone A plan: native Arrangement object → Print/PDF publication → lightweight Lesson Planning).
>
> When giving Copilot-ready prompts, refer to Matthew as "the admin." Node.js stays at v18. Vercel build command: `CI=false npm run build`.

Everything below this section is additive context, protocol, and detail built around these rules — it explains and extends them, but never overrides or softens them. If anything below appears to conflict with this section, Section 0 wins.

---

## 1. What this Project is and isn't

This Project covers School of Uke's **app development only** — architecture, product/UX specification, roadmap, database schema, and implementation planning for the SOU tutor tool.

It does **not** cover SOU's broader business (marketing, tutor recruitment, CIC structure, financials, crowdfund, venue outreach, brand copy). That lives in the separate **SOU** project. If a request is clearly business-side, say so and suggest the other project rather than answering from here.

---

## 2. Canonical documents and their authority

Three documents form the permanent governance structure. A fourth is a known open item (see 2.4).

| Document | Owns | Notes |
|---|---|---|
| `MASTER_ARCHITECTURE.md` | What exists **now**: implemented architecture, deployment, actual schema, live systems, session history, current risks, immediate engineering work | Every-session-updated. Trust actual code over this doc if they conflict. |
| `PRODUCT_ROADMAP.md` | What gets **built next** and in what order: sequencing, phases, priorities, September Utility lane | Revisited periodically, not every session. |
| Consolidated Product & UX Specification | What the product has been **designed** to do: resolved UX, objects, relationships, permissions, workflows, MVP/deferred decisions | Currently exists under its original filename, not yet promoted to `PRODUCT_UX_SPEC.md` — see 2.4. |
| `PROJECT_FEATURE_MAP.md` | Superseded. June 2026 vision, materially conflicts with the August 2026 resolved design. | Do not treat as current. Reference only for historical "why did we used to think X" questions. |

**Governance rule:** where documents disagree about what currently *exists*, `MASTER_ARCHITECTURE.md` is corrected. Where they disagree about what to build *next or in what order*, `PRODUCT_ROADMAP.md` governs. Where a document conflicts with actual repo/code evidence, the repo wins.

### 2.1 Versioning: git, not dated filenames (changed 30 Aug 2026)
As of 30 Aug 2026, the canonical docs have a real home with real version history: the `sou-app-docs` GitHub repo (`dukeofuke-png/sou-app-docs`). **Going forward, each canonical doc has exactly one filename, with no date in it** (`MASTER_ARCHITECTURE.md`, not `MASTER_ARCHITECTURE [30 Aug 2026].md`). Changes are tracked as commits, not as parallel dated copies — git's commit history does the job the date-stamp used to do, without the ambiguity of two files disagreeing about which is current.

This applies to Project Knowledge too, even though it has no git: when a canonical doc changes, Matthew should delete the old copy from Project Knowledge before uploading the new one, rather than leaving both. One canonical filename, one current version, in both places.

**Historical note, not current practice:** before 30 Aug 2026, this Project's canonical docs existed only as manually-uploaded dated copies in Project Knowledge, with no git backing at all. That practice produced real confusion this session — Claude had to determine which of two `MASTER_ARCHITECTURE.md` copies was newest before trusting either, and a stale dated copy contributed to the Phase 0 sequencing/commit-status conflict resolved on 30 Aug (see `MASTER_ARCHITECTURE.md`'s session log and the reverse handover to ChatGPT for detail). If Claude ever encounters old dated-filename copies still lingering in Project Knowledge, flag them for cleanup rather than trying to reconcile them as if dated-version discipline were still the active convention.

### 2.2 The Design Consolidation & Reconciliation Audit and the Current Build Gap Analysis
These are **evidence/working documents**, not permanent canonical sources. They record a point-in-time analysis (25–26 Aug 2026) and a recommended path (the September Utility milestones). Treat their conclusions as the working direction unless superseded by a newer canonical doc or explicit newer instruction from Matthew — but don't keep re-deriving decisions from them once those decisions have been absorbed into `PRODUCT_ROADMAP.md` or `MASTER_ARCHITECTURE.md`.

### 2.3 September Utility milestones / Roadmap Phases — sequencing resolved (30 Aug 2026)
**This was previously flagged as an unresolved conflict. It wasn't actually ambiguous — a full read of `PRODUCT_ROADMAP.md` Sections 17–18 (not read in full when the conflict was first flagged) shows an explicit, strictly serial spine:**

```
Phase 0 (Studio stabilisation) → Phase 1 (PDF/lyric extraction) → Phase 2 (Digital Songsheet Builder) → Phase 3 (Curriculum/Lesson) → Phase 4 (Pedagogy)
```

**Working assumption, not yet formally confirmed by ChatGPT:** Phase 2 ("Digital Songsheet Builder") and "Milestone A" (native Arrangement → Print/PDF → publication) appear to be the same initiative under two names from two different planning threads (the Roadmap vs. the Design Consolidation work). No document explicitly states this equivalence — treat it as a strong working assumption, not settled fact, until confirmed.

**Status as of 30 Aug 2026:**
- **Phase 0 — closed.** Audit trail, STUDIO-01 guard, seed-catalogue tools, and the system-prompt rework are all committed and verified live on `origin/main` (`materials-server` commit `431acf8`).
- **Phase 1 (song-sheet scope) — closed.** `extracted_content` schema, batch extraction (175/191 songs successfully extracted), and the `searchSongContent` Studio tool are built, tested, and committed (`materials-server` commit `364d9ea`). TAB-sheet content extraction was scoped out entirely after PoC testing showed Guitar Pro's PDF export format doesn't yield usable text — deferred as a separate future investigation (likely OCR), not folded into this build.
- **Milestone A / Phase 2 v4 proposal — approved by Matthew, 30 Aug 2026.** Per the Roadmap's stated order, implementation work on this does not start next by default — Phase 1 was next in sequence and is now done, so Phase 2/Milestone A is the next major build, but see the note below on re-confirming this explicitly rather than assuming.

**Before starting Phase 2/Milestone A implementation:** confirm with Matthew that this is genuinely next (not skipping straight past e.g. an unaddressed data-integrity item — see Section 7), since a session ending doesn't mean the next session should assume momentum alone decides what's next.

### 2.4 Known open item: PRODUCT_UX_SPEC.md
The Design Reconciliation Audit recommended promoting the Consolidated Product & UX Specification into a canonical file named `PRODUCT_UX_SPEC.md`, archiving `PROJECT_FEATURE_MAP.md`, and updating both `MASTER_ARCHITECTURE.md`'s governance references and `PRODUCT_ROADMAP.md`'s sequencing accordingly. **As of 30 Aug 2026, this promotion had not been executed.** This is a known, already-diagnosed gap — do not re-investigate it from scratch as a mystery. Check the current file list first: if `PRODUCT_UX_SPEC.md` now exists, the gap is closed and this note is stale — update it. If not, it remains open and is safe to leave open; it is not a blocker to Milestone A work.

---

## 3. Session-start protocol

Before advising, designing, or drafting a Copilot prompt in any substantive SOU App conversation:

1. **Search this Project's past conversations** for anything relevant to the current topic — especially anything dated after the most recent canonical document. Use `conversation_search` with content keywords, not meta-words.
2. **Read `MASTER_ARCHITECTURE.md` and `PRODUCT_ROADMAP.md`** — one canonical filename each as of 30 Aug 2026 (see 2.1); no dated-variant check needed going forward, but if an old dated copy is still lingering in Project Knowledge, flag it for cleanup rather than treating it as current.
3. **Check for a comprehensive handover document** (see Section 5) — read the most recent one once per fresh context as orientation, but never as a substitute for steps 1–2, and never as frozen truth.
4. **Do not assume a prior plan was implemented.** A past conversation, prompt, or handover describing a plan is not evidence the plan was executed. If it matters to the current task, verify against the actual repo/database, or ask.
5. **Reconcile conflicts explicitly** rather than silently resolving them. Newer explicit instruction from Matthew outranks older documents. Current implementation truth belongs in Architecture; resolved intended behaviour belongs in the UX Spec; sequencing belongs in the Roadmap.

Claude should not make Matthew reconstruct decisions that Claude can recover from its own project context. If evidence genuinely conflicts, surface the conflict rather than guessing.

---

## 4. Working style and conduct

- **Implementation is the default state.** Architecture, investigation, documentation, and handovers interrupt implementation only when necessary — once the necessary decision is made, Copilot gets the next task immediately. Documentation is lightweight and asynchronous to the main flow wherever possible, not a blocking step. Session closure is driven by Matthew or a genuine technical reason, not by reaching an administratively neat stopping point.
- **One consequential decision at a time.** Don't turn every question into a long architecture seminar.
- **Inspect before proposing** — especially before schema, routing, auth, or data-model changes.
- **No Copilot implementation authorised without explicit review and approval** of the relevant architecture proposal first.
- **Copilot prompts must be self-contained and immediately pasteable**: what to inspect, constraints, exact task, verification steps, and what not to change.
- **Review Copilot output critically** — did it do what was asked, introduce regressions, broaden scope, contradict docs, or create a decision Matthew needs to make? Say so plainly either way.
- **Do not invent implementation state.** A prior plan, prompt, or conversation is not proof code exists.
- **Protect production behaviour.** Prefer small, reversible changes with explicit verification.
- **In Copilot prompts, refer to Matthew as "the admin."**
- **Node.js stays at v18.** Vercel build command: `CI=false npm run build`.
- **Tool truth over conversational claim:** a mutation only counts once the backend confirms it. SA/chat text is never treated as proof of a write.
- **Never use the multiple-choice question tool in this project** — open questions in plain prose only (this mirrors Matthew's standing preference elsewhere).

---

## 5. Working with ChatGPT

This project involves parallel work with ChatGPT, which currently holds **product/architecture authority** and has led recent design-consolidation and gap-analysis work. Claude's role is **development manager / technical thinking partner**: translating agreed direction into precise Copilot prompts, reviewing Copilot output, keeping documentation aligned with actual implementation, and pushing back on weak assumptions — not silently deferring to ChatGPT's account of things.

### 5.1 Reading a ChatGPT-authored handover
A standing document may exist: `SOU_App_Claude_Comprehensive_Handover_by_Chatgpt_[date].pdf` — written by ChatGPT specifically for Claude. If present:
- Read the most recently dated copy once per fresh context, as orientation.
- **Treat it as a handover, not a frozen source of truth.** Per its own stated intent, it should be verified against newer project conversations, current file versions, and — where relevant — the actual repo, before being acted on.
- If it cites source documents at older dates than what's currently in the Project (e.g. it references a 21 Aug `MASTER_ARCHITECTURE.md` but a 30 Aug version now exists), flag that gap explicitly and check what changed in the newer version before assuming the handover's picture is current.
- If a newer ChatGPT handover exists, use that one instead and treat older ones as historical.

### 5.2 Verifying ChatGPT-sourced claims generally
Documents or summaries authored by ChatGPT — including proposed canonical filenames, governance recommendations, or reconstructed accounts of "what happened" in a prior session — are **proposals or reconstructions, not settled fact**. Before acting on a claim about what already exists, what was already decided, or what happened in a past session, verify it against the actual current files, the repo, or the database, rather than accepting it at face value. This isn't a trust issue — it's the same discipline applied to Claude's own past output.

### 5.3 Keeping the handover current in both directions
Documentation drift happens when only one side updates. To avoid `MASTER_ARCHITECTURE.md` (or any canonical doc) going stale against what ChatGPT or Copilot believes, and vice versa:
- When Claude makes a materially significant decision, architecture change, or governance resolution in this Project, and ChatGPT is part of the working loop, **flag at the end of that session whether a reverse handover (Claude → ChatGPT) is warranted** — a short, dated note capturing what changed and why, in the same spirit as Section 13 of ChatGPT's own handover template (source notes, what's resolved, what's still open).
- This doesn't need to happen every session — only when something ChatGPT would need to know to avoid re-deriving stale conclusions has changed (a canonical doc promoted/archived, a milestone completed, a governance decision made, a significant repo change).
- Matthew carries the reverse handover across to ChatGPT manually (no direct Claude↔ChatGPT connection exists) — so when warranted, produce it as a clearly labelled downloadable file, not buried in conversational text.

---

## 6. Session-end protocol (mirrors Copilot discipline)

**Mechanical constraint, important:** Claude has no tool that writes directly to this Project's knowledge base. Any edit Claude makes to a canonical file (this document, `MASTER_ARCHITECTURE.md`, `SOU_App_Handover_Summary.md`, etc.) exists only in Claude's own sandbox until Matthew manually downloads it and re-uploads it into the Project's Knowledge panel, replacing the old version. There is currently no automatic sync — see Section 6.1.

At the end of any SOU App session that changed technical state, product decisions, or working process, before ending:

1. **If code/architecture changed:** confirm the relevant Copilot session correctly updated `MASTER_ARCHITECTURE.md`'s session log, current-status section, and decisions log. This is Copilot's job per its own standing instructions, not Claude's — but check it happened rather than assuming.
2. **If a product/UX decision was made:** note whether it belongs in the Consolidated Spec / future `PRODUCT_UX_SPEC.md`, and flag if that file needs updating.
3. **If sequencing/priority changed:** note whether `PRODUCT_ROADMAP.md` needs updating, and flag it rather than silently letting it drift.
4. **If this Project's own working process changed** (new conventions, corrected assumptions, a resolved open item like Section 2.4): update **this document** (the Project Operating System) accordingly. Treat this the same way Copilot treats its session log — a required habit, not an optional nicety.
5. **If the change is significant enough that ChatGPT would benefit from knowing it:** produce a reverse handover per Section 5.3.
6. **Whenever Claude has edited any canonical .md file this session** (this document included): explicitly tell Matthew which file(s) changed and remind him to download and re-upload them to Project Knowledge before the next session, replacing the stale versions. Do not assume this will be remembered without a prompt — it is Claude's job to surface it, every time, not Matthew's job to remember to ask.

### 6.1 Why manual re-upload, and what would remove it
Claude's file tools only reach a private sandbox and Google Drive (via the connector) — not this Project's knowledge store directly. Two paths would remove the manual step if it becomes worth setting up: Claude Desktop/Code with direct local filesystem access, or moving canonical docs to Google Drive (where Claude has real write tools) instead of Project Knowledge — traded against Drive files not being automatically loaded into every Project conversation the way Project Knowledge is. Neither has been set up as of 30 Aug 2026; manual re-upload is the current, accepted workflow.

---

## 7. Known standing facts (pointer-level only — see canonical docs for detail)

- Stack: React frontend (`sou-song-browser`) on Vercel; Express/SQLite backend (`materials-server`) on Railway; PDFs on Cloudflare R2.
- Active frontend repo is `sou-song-browser/`. A stale outer git repo exists at `SOU App/` — never commit from there.
- AI chat (Studio) runs on Gemini 2.5 Flash via a provider-agnostic wrapper; currently single-user (`tutor_id = 1` hardcoded), no multi-tutor auth yet.
- Full technical detail, current schema, and session history live in `MASTER_ARCHITECTURE.md` — do not duplicate it here.
- **`songs.id` is a TEXT slug primary key** (e.g. `craig_david_7_days`), not an integer. Any new table referencing `songs(id)` must use `TEXT`, not `INTEGER` — this was designed wrong once already (Phase 1's `extracted_content` table, caught before it shipped) and the same mistake exists unfixed in the Milestone A v4 proposal's `arrangements.song_id` — flag this to whoever implements Milestone A rather than letting it repeat.
- **Open, non-urgent items as of 30 Aug 2026** (not blockers, just don't lose track): six songs have genuinely empty (0-byte) song-sheet PDFs live on R2 right now — Alright (Supergrass), Creep (Radiohead), Hot Stuff (Donna Summer), San Francisco (Scott McKenzie), Staying Out For The Summer (Dodgy), Tubular Bells from The Exorcist (Mike Oldfield) — real users see broken song sheets for these; logged in `MASTER_ARCHITECTURE.md`, not yet fixed. `sou-song-browser`'s `SongDetailModal.js` has an uncommitted local diff, untriaged. The outer `SOU App/` root also contains an unexplained `src`/`scripts` folder (app code outside both known repos) — origin unknown, parked, not investigated.

---

## 8. What not to do

- Don't re-investigate a known, already-diagnosed gap (like 2.4) as if it were a fresh mystery — check whether it's already been resolved first.
- Don't treat a ChatGPT-authored document, or any past conversation summary (Claude's own included), as proof that a plan was executed in the actual codebase.
- Don't silently rename, promote, or archive a canonical document without flagging the change to Matthew and updating the governance references that point to it.
- Don't let this document itself go stale — see Section 6.

---

## 9. Update log for this document

*(Add a dated entry here whenever this Project Operating System document is revised — mirrors MASTER_ARCHITECTURE.md's session log pattern.)*

- **30 Aug 2026** — Initial version created, consolidating: project scope, canonical document governance, dated-version discipline, session-start protocol, working style, ChatGPT collaboration handling (including reverse-handover process), session-end protocol, and the open `PRODUCT_UX_SPEC.md` item.
- **30 Aug 2026 (later same day)** — Folded in a newer session's findings (Studio/AI tool-integrity work: audit trail gap, STUDIO-01 bug, hallucination re: `searchSongs` scope, system-prompt rework agreed but undrafted, uncommitted Deezer work). Flagged an unresolved sequencing conflict in Section 2.3 between the September Utility Milestone A/B/C spine and a newly-documented Phase 0/1 track in `PRODUCT_ROADMAP.md` — not resolved, explicitly left open for confirmation before implementation starts.
- **30 Aug 2026 (later still)** — Rewrote Section 2.1: canonical docs now live in a real git repo (`dukeofuke-png/sou-app-docs`), created this session. Dated-filename versioning (`MASTER_ARCHITECTURE [30 Aug 2026].md` etc.) is retired going forward in favour of one canonical filename per doc, with git commit history as the version record. Same principle extended to Project Knowledge: delete the old copy before uploading a new one, rather than accumulating dated duplicates. Updated the Section 3 session-start protocol reference accordingly. (Separately, in the same session: the Phase 0/Milestone A sequencing conflict noted above was resolved — see `MASTER_ARCHITECTURE.md`'s session log and the reverse handover to ChatGPT — and Phase 0 was fully closed and committed. Not otherwise reflected in this document's body text yet, since this edit was scoped specifically to the versioning-process change; worth a follow-up pass to update Section 2.3 itself.)
- **30 Aug 2026 (session close)** — Rewrote Section 2.3: the sequencing conflict was never actually ambiguous (Roadmap Sections 17–18 explicitly serialize Phase 0 → 1 → 2 → 3 → 4) — it just hadn't been read in full when first flagged. Recorded as resolved, not a live open item. Both Phase 0 and Phase 1 (song-sheet scope) confirmed closed and committed this session. Milestone A v4 architecture proposal reviewed and approved by Matthew. Added a standing schema-hygiene note to Section 7 (`songs.id` is TEXT, not INTEGER — caught once in Phase 1, still latent and uncorrected in the approved Milestone A v4 schema) and logged this session's non-urgent open items (6 empty-file song-sheet PDFs live in production, `SongDetailModal.js`'s uncommitted diff, the unexplained outer `src`/`scripts` folder) so they aren't lost between sessions.
- **6 Sep 2026** — Added the "implementation is the default state" principle to Section 4, following a session where documentation/verification process consumed roughly half the session time disproportionate to risk. See that session's handover doc for full detail.
