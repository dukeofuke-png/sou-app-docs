# SOU App — Product Roadmap
**Status:** Working strategic/product direction  
**Date:** 20 August 2026  
**Read alongside:** `MASTER_ARCHITECTURE.md`  
**Audience:** Matthew, Claude, GitHub Copilot / VS Code agents

---

## 1. Purpose of this document

This document is the forward product-development companion to `MASTER_ARCHITECTURE.md`.

- `MASTER_ARCHITECTURE.md` remains the technical source of truth for what the app currently is, how it is deployed, and how its existing systems work.
- This document describes where the SOU App should go next, in what order, and why.

Do not treat this roadmap as permission to build every idea in it immediately. The central principle is to preserve momentum, solve School of Uke's real operational needs first, and make small architectural choices that quietly create the foundations for a more valuable music-learning intelligence asset over time.

Before implementing any roadmap item:
1. Read the current `MASTER_ARCHITECTURE.md`.
2. Inspect the existing codebase and current uncommitted work.
3. Confirm the proposed change is not already implemented.
4. Show the proposed schema/architecture/change before implementation.
5. Preserve existing production behaviour unless the task explicitly requires changing it.

---

## 2. Product thesis

The SOU App should be treated as two compatible layers.

### Layer A — SOU Operating Product

The practical tutor-first product School of Uke needs now:

- discover songs
- curate the SOU teaching catalogue
- manage teaching metadata
- create and access songsheets, TAB and theory resources
- plan lessons and courses
- support tutors through Studio Chat
- track progression
- eventually provide useful student-facing digital materials

This layer must remain the development priority.

### Layer B — Music Learning Intelligence

A longer-term strategic asset that can grow as a by-product of normal SOU teaching.

The potentially defensible intelligence is not the 47K song catalogue or generic AI-generated metadata. Those are increasingly commoditisable.

The more interesting asset is structured, real-world knowledge connecting:

`music → skills → theory → teaching methods → curriculum → learners/groups → observed outcomes`

Over time this could answer questions such as:

- Which songs work well for introducing a particular skill?
- Which chord transitions repeatedly cause difficulty at a given stage?
- Which teaching interventions help?
- Which repertoire works well after a particular skill has been introduced?
- Which teaching keys are genuinely effective with adult learners?
- What sequencing produces successful progression?
- What tends to work with real groups, rather than what an AI predicts should work?

This must emerge gradually from normal product use. Do not turn the app into a research project or burden tutors with data-entry bureaucracy.

---

## 3. Current product state relevant to this roadmap

The app already has:

- a live React frontend
- Express/Node backend
- SQLite production teaching database
- approximately 218 songs in the curated SOU Catalog
- approximately 47,273 songs in the discovery/seed catalogue
- enriched song metadata
- SOU-specific teaching metadata
- PDF songsheet relationships/storage
- Studio Chat using an AI provider abstraction
- AI tool access to the SOU teaching catalogue
- locally built/tested work for AI seed-catalogue search and song promotion
- a tutor-first product vision that already anticipates digital songsheet creation and a Studio workspace

The current teaching schema already contains useful fields including:

- SOU keys
- level
- chords
- chord numerals
- teaching notes
- strum style
- fingerpicking style
- songsheet status
- TAB status

These should be reused rather than duplicated.

---

## 4. Immediate engineering context — 19/20 August 2026

The following work was in progress immediately before this roadmap was created.

### Production recovery
The Railway free trial expired and took the backend offline. Railway has now been upgraded to the Hobby plan and production is working again.

### Repository rule
There was a stale outer git repository at `SOU App/`. The active frontend repository is:

`sou-song-browser/`

Always confirm the correct repository before git operations. Do not commit from the stale outer workspace repo.

### UI terminology already shipped
- Conversation → Studio Chat
- Conversations → Chats
- Manage SOU Database → Manage SOU Catalog

### Seed catalogue AI tools — built locally, NOT yet shipped
Two AI tools were added:

- `searchSeedCatalog`
- `promoteSongFromSeed`

A bug that limited search to 500 of 47,273 songs was found and fixed.

A hallucinated/fabricated Spotify ID was also observed. A mitigation instruction was added requiring IDs to be copied verbatim rather than guessed.

### Important unresolved AI integrity issues
The feature has deliberately not been pushed to production because:

1. **There is no adequate audit trail for AI tool activity.**
2. **The AI can claim an action occurred when no backend tool call actually happened.**
3. **STUDIO-01:** raw internal tool-response JSON can leak into the live chat UI.

These are not merely cosmetic bugs. Reliable distinction between **AI language** and **confirmed backend action** is an architectural requirement for future write-capable Studio tools.

### System-prompt issue
A previous rule effectively forced Studio to use database tools for all relevant answers and suppressed legitimate general-knowledge brainstorming.

The intended replacement behaviour is:

1. General knowledge is allowed by default.
2. A real database lookup is required when Studio makes a factual claim specifically about SOU-owned data.
3. Ambiguous requests should be clarified rather than guessed.

### Uncommitted Deezer work
Old Deezer-first BPM work may still be sitting uncommitted in the codebase. Inspect it and explicitly decide whether to commit, retain separately, or discard. Do not accidentally mix it into unrelated work.

---

# 5. Roadmap priority order

## PHASE 0 — Close and stabilise the current Studio tool work

**Priority:** Immediate  
**Scope:** Small, bounded, finish work already underway.

Before opening another major development front:

- resolve/implement an AI tool-action audit trail sufficient to verify actual calls and outcomes
- prevent Studio from claiming successful write actions unless backend execution confirms success
- fix STUDIO-01 raw tool JSON leakage
- replace the overly restrictive system-prompt rule with the agreed three-part behaviour
- validate `searchSeedCatalog` and `promoteSongFromSeed`
- only then decide whether to ship the feature
- inspect the unrelated uncommitted Deezer work and keep it out of the feature commit unless deliberately included

### Architectural rule: tool truth beats conversational claim

For any current or future write-capable AI tool:

> A write is considered to have happened only when the backend confirms it.

Studio text must never be the source of truth for mutations.

Future tools such as songsheet creation/editing, curriculum editing, or pedagogy/outcome recording must inherit this rule.

### Definition of Done

The seed search/promote feature can be safely shipped without:
- fabricated identifiers
- unverifiable write claims
- raw internal tool payloads appearing in user-facing chat

---

# 6. PHASE 1 — Legacy content intelligence: PDF / lyric text extraction

**Priority:** High, already agreed as next capability after current Studio stabilisation.

### Problem

The app can identify and serve existing songsheet PDFs, but Studio cannot meaningfully reason over their contents.

This prevents content-aware queries such as:

- find songs mentioning summer
- find songs with a particular lyrical theme
- compare what is actually on two SOU songsheets
- inspect arrangements, lyric structure or annotations held only inside PDFs

### Goal

Extract searchable text/content from the existing songsheet/resource library and make it available to Studio through explicit retrieval/tooling.

### Important boundary

This solves the **legacy library problem**.

Do not make PDF extraction the canonical architecture for future digital songsheets.

Future native songsheets should store their actual content structurally and generate PDFs as an output.

### Definition of Done

Studio can retrieve/search meaningful content from existing SOU PDFs/resources without pretending metadata search is content search.

---

# 7. PHASE 2 — Digital Songsheet Builder

**Priority:** URGENT / next major product build.

This is the most important operational product capability after the currently open Studio/PDF work is closed.

## Why this is urgent

School of Uke is still creating teaching materials that primarily become static print/PDF assets.

Every new print-only songsheet creates future migration work.

The objective is to cross a threshold as quickly as practical where **new SOU songsheets are born digital**.

This is not a side feature. It is a central transition in how SOU produces teaching content.

## Product principle

### Native digital content is canonical.
### PDF/print is an output format.

Do NOT design a system where:
1. the app generates a PDF,
2. stores only the PDF,
3. and Studio later has to extract the PDF to understand what was created.

The native songsheet should be directly readable/editable by the app and AI.

## v1 Definition of Done

Matthew can:

1. open/create a song in the SOU App
2. create a new SOU songsheet digitally
3. enter/edit the song's core sheet content comfortably
4. save and reopen it without loss
5. associate it with the correct song
6. produce a professional printable/exportable PDF
7. continue editing the native digital version later

The v1 goal is **production replacement**, not a perfect interactive-learning environment.

If Matthew can realistically stop creating new print-only songsheets, v1 has succeeded.

## Songsheet Builder v1 should prioritise

- fast authoring
- strong typography/readability
- chord + lyric layout
- song sections
- clear page/print behaviour
- SOU teaching key
- basic arrangement/teaching annotations where required
- save/edit/version reliability
- PDF/print export
- direct relationship to the SOU Catalog song record

## Do not block v1 on

- student interactivity
- automatic transposition
- advanced playback
- AI-generated arrangements
- full pedagogy ontology
- learner tracking
- sophisticated theory linking
- cross-instrument support
- knowledge graph infrastructure

Those may come later.

## Future-proofing requirement

Even though v1 should stay lean, do not store the entire native songsheet as an opaque PDF-like blob if a modest structured representation can preserve useful components such as:

`song → sections → chord/lyric content → arrangement annotations`

The exact schema must be designed against the real existing codebase before implementation.

## First functional slice: manual chord/lyric import (approved 31 Aug 2026)

School of Uke does not yet have commercial licensing/catalogue infrastructure for bulk song-content acquisition. Rather than block v1's "enter/edit the song's core sheet content comfortably" objective on that unresolved parallel track, the first concrete build step is a tutor-first manual import path: a tutor pastes chord/lyric text (initially copied from Ultimate Guitar desktop), the app parses it into structured Arrangement content, the tutor reviews and corrects it, and only the reviewed result is persisted.

**Paste → parse → review → persist is the first functional Arrangement Builder slice.** Print/PDF export (Definition of Done items 6–7 above) is a later slice, not part of this one.

Commercial licensing/bulk catalogue ingestion remains a parallel, non-blocking future track — this manual path exists so tutors have working Arrangement-creation capability now, independent of when that track resolves.

Full schema and parser specification live in `MASTER_ARCHITECTURE.md` Section 5.5 — not duplicated here.

---

# 8. PHASE 3 — Curriculum and lesson-plan integration

**Priority:** After digital songsheet production is operational.

The app currently does not contain the SOU curriculum/lesson-plan corpus in a properly usable native form.

This is a major dependency for richer tutor intelligence.

## Goal

Studio should eventually understand:

- course
- level
- term/cohort
- lesson/week
- songs/materials used
- theory/resources used
- intended lesson focus
- prior and subsequent lessons

This should support real tutor tasks such as:

- “What did Level 2 cover last term?”
- “What songs have we previously used to introduce Key F?”
- “Plan next week's lesson based on what this group has already done.”
- “Which theory sheet belongs with this lesson?”

## Architecture principle

Do not ingest curriculum merely as giant unstructured text blobs if avoidable.

Preserve the original prose, but give lessons stable identities and relationships to existing app objects such as songs and resources.

This is where the app begins to gain durable teaching memory.

---

# 9. PHASE 4 — Pedagogy Layer v0.1

**Priority:** Directional architecture now; active build later.

The Pedagogy Layer should inform how Songsheet Builder and Curriculum are designed, but it should NOT delay either.

## Purpose

Create a small structured teaching-memory layer that lets the app gradually connect:

`song ↔ skill ↔ theory ↔ level ↔ lesson ↔ learner/group ↔ outcome`

The historical SOU lesson plans and appraisals show that the most valuable information is often not merely “Song X teaches Skill Y”.

It is closer to:

> A particular group encountered a particular challenge; a teaching intervention was used; an observed response followed.

Example pattern:

`Escapism → D→Em difficulty → isolate/repeat transition → improved fluency`

That is much harder for generic AI to infer from a recording or score.

## Candidate Pedagogy v0.1 concepts

These are provisional and must be validated against the historical lesson corpus before schema implementation:

1. Skill
2. Theory Concept
3. Song Teaching Level
4. Song ↔ Skill relationship
5. Song ↔ Theory relationship
6. Prerequisite
7. Learning Objective
8. Teaching Method / Intervention
9. Challenge / Failure Point
10. Group / Cohort
11. Group Starting State
12. Outcome
13. Lesson Event

Do not automatically add these as columns to the already-large `songs` table.

Many are many-to-many relationships or belong to lesson events rather than songs.

A small relational layer beside the existing song data is the likely direction, subject to code/schema review.

---

# 10. Historical pedagogy analysis — planned discovery work

Before implementing Pedagogy v0.1, analyse the historical SOU lesson-plan and appraisal corpus across levels and years.

The purpose is NOT merely to summarise the documents.

Extract candidate:

- skills
- theory concepts
- teaching methods/interventions
- recurring challenge types
- outcome language
- progression rules
- prerequisite relationships
- song ↔ skill relationships
- repeated teaching heuristics
- plan-versus-actual differences

Also distinguish:

- **Observed evidence** — directly recorded in SOU teaching history
- **Inferred hypothesis** — plausible pattern requiring more evidence
- **Proposed product structure** — a design choice, not an empirical finding

The resulting vocabulary should come from real SOU practice rather than being invented abstractly.

---

# 11. Low-burden future intelligence capture

When pedagogy/outcome capture eventually enters the product, tutors should NOT be required to maintain research spreadsheets or lengthy forms.

Ideal workflow:

1. Tutor teaches normally.
2. Tutor writes or dictates a natural appraisal.
3. AI proposes structured observations.
4. Tutor confirms/corrects them.
5. App stores the structured relationships.

Example:

Tutor says:

> “Double-time is coming along well. The D to Em change was much better after we isolated it. They still struggled with the faster section of Escapism.”

System might propose:

- Song: Escapism
- Skill: double-time strumming
- Challenge: D→Em transition
- Method: isolated repetition
- Outcome: improved
- Remaining challenge: faster section

Tutor confirms rather than manually completing a multi-column form.

**Design test:** if this creates meaningful extra admin after a lesson, redesign it.

---

# 12. Strategic asset hierarchy

Treat the following as a working hypothesis, not a valuation claim.

Likely long-term defensibility, strongest first:

1. observed pedagogy → outcome relationships
2. accumulated SOU teaching methodology / heuristics
3. relationships between repertoire and pedagogy
4. longitudinal curriculum history
5. enriched song metadata
6. raw discovery catalogue

Why:

Future AI will increasingly be able to infer BPM, key, chords, musical characteristics, approximate difficulty and possible teaching uses.

It cannot recover SOU's historical real-world evidence merely from a recording:

> what adult learners actually struggled with, what a tutor changed, and what happened afterwards.

That is the data to preserve.

---

# 13. AI / Studio architecture direction

Studio should progressively gain three forms of capability.

## A. Knowledge

What SOU owns/knows:

- catalogues
- metadata
- songsheets
- TAB
- theory resources
- curriculum
- lesson history
- eventually pedagogical observations

## B. Tools

What Studio can actually do:

Current/emerging examples:

- search SOU Catalog
- search discovery catalogue
- promote a discovery song

Future examples:

- search resource content
- open songsheet
- create songsheet
- edit songsheet
- search curriculum
- retrieve lesson history
- record/confirm lesson appraisal
- find songs by teaching objective

## C. Reasoning

What Studio can infer from the above:

- suggest suitable songs
- explain why they are suitable
- propose lesson sequences
- identify reinforcement opportunities
- surface prior SOU experience
- eventually compare outcomes across cohorts

Do not confuse AI fluency with data truth.

For SOU-specific factual claims, retrieval/tool evidence should ground the answer.

For confirmed write actions, backend success should ground the claim.

---

# 14. Future Music Learning Intelligence — NOT active roadmap

Possible future capabilities include:

- evidence-informed repertoire recommendations
- “next best song” recommendations based on group history
- adaptive progression
- curriculum intelligence
- teaching-intervention recommendations
- structured student practice pathways
- cross-cohort outcome analysis
- licensing/API access to pedagogical intelligence
- broader teacher/education platform
- partnerships with publishers, learning platforms or instrument companies
- eventual spin-out/acquisition thesis

These are strategic possibilities only.

Do not build them until the SOU operating product and accumulated evidence justify them.

---

# 15. Explicit non-goals / anti-scope-creep rules

Do NOT currently build:

- a universal music knowledge graph
- a new standalone “Music Learning Intelligence” startup
- detailed individual learner competency modelling
- predictive ML models
- formal causal-effectiveness claims
- cross-instrument pedagogy
- external teacher marketplace
- enterprise/institutional curriculum product
- large new data-entry workflows
- AI-generated metadata solely because it is interesting
- features that delay digital songsheet production without a compelling operational reason

The roadmap should compound quietly, not explode sideways.

---

# 16. Product decision filter

When deciding whether something belongs on the active roadmap, ask in order:

1. Does this materially improve SOU's current teaching/content workflow?
2. Does it reduce ongoing manual work or prevent future migration/rework?
3. Can it create useful structured teaching knowledge as a by-product?
4. Is the smallest implementation reversible?
5. Are we building something future AI will commoditise anyway?
6. Is this required now, or merely strategically interesting?

A “strategically interesting” idea is not automatically a feature request.

---

# 17. Near-term development spine

The current intended sequence is:

```text
STABILISE CURRENT STUDIO TOOL WORK
        ↓
PDF / LEGACY CONTENT EXTRACTION
        ↓
DIGITAL SONGSHEET BUILDER
        ↓
CURRICULUM / LESSON INTEGRATION
        ↓
PEDAGOGY LAYER v0.1
        ↓
LIGHTWEIGHT APPRAISAL / OUTCOME CAPTURE
        ↓
GROUP PROGRESSION / TEACHING MEMORY
        ↓
MUSIC LEARNING INTELLIGENCE
```

This is a dependency/priority spine, not a rigid waterfall.

Small architecture choices for later stages may be made earlier when doing so costs little and prevents rework.

But later stages must not hijack the urgent operating-product work.

---

# 18. Immediate instruction to Claude / Copilot

If this document is being read during the August 2026 development session:

**Do not begin implementing the Pedagogy Layer.**

First inspect:

- `MASTER_ARCHITECTURE.md`
- current git status/diffs
- the locally built seed-catalogue AI tool work
- current Studio tool execution flow
- the STUDIO-01 rendering issue
- current system prompt
- any uncommitted Deezer changes

Finish or safely isolate the existing work before opening the next major build.

After the current Studio loop is stabilised, continue with the already-agreed PDF/content extraction work.

The **next major product build after that is the Digital Songsheet Builder**, with the explicit objective of allowing SOU to stop creating new print-only songsheets as soon as practical.

Before implementing Songsheet Builder, inspect the existing product vision / `PROJECT_FEATURE_MAP.md` and current codebase, then propose the smallest viable native songsheet data model and editor architecture for review.

Do not code the schema before showing the proposal.

---

# 19. Definition of success for the overall direction

The SOU App succeeds in the near term if it makes School of Uke materially easier to run and teach.

The strategic layer succeeds if, while doing that, the app quietly begins preserving knowledge that would otherwise disappear into PDFs, lesson notes and tutors' heads.

The long-term flywheel is:

`better SOU data`
→ `better lesson/content tools`
→ `more normal tutor use`
→ `more real teaching evidence`
→ `better SOU intelligence`
→ `better teaching decisions`

The commercial asset should grow as a by-product of making the school better.

---

## Document governance

- Keep `MASTER_ARCHITECTURE.md` as the definitive technical/current-state document.
- Keep this file focused on forward product direction and sequencing.
- If implementation changes architecture, update `MASTER_ARCHITECTURE.md`.
- If strategic priorities materially change, update this roadmap.
- Do not allow the two documents to silently contradict each other.
- Significant new roadmap documents should be referenced from `MASTER_ARCHITECTURE.md` in accordance with the existing documentation rule.
- Division of responsibility: `MASTER_ARCHITECTURE.md` is the living, every-session-updated record of current technical state, session history, and immediate next steps; this roadmap is the longer-range strategic/phase map, revisited periodically rather than every session — a "Next Up" holding area above the day-to-day task list. `MASTER_ARCHITECTURE.md`'s own outstanding-items list should stay focused on near-term/immediate items; anything belonging to a later roadmap phase belongs here, not duplicated in detail in the architecture doc's outstanding list.
