# SOU App — Product & UX Specification

**Status:** Canonical living product-design record — approved 20 September 2026  
**Version:** 1.0  
**Date:** 20 September 2026  
**Authority:** Defines what the SOU App has been resolved and designed to do. It does not, by itself, prove implementation.  
**Primary recovery source:** “SOU App 21–26 Aug 2026 — Visualising Lesson Planning” (Q1–Q294), reconciled with the previous consolidated specification and verified September architecture decisions.

---

## 0. How to use this document

This specification is the canonical product and UX source of truth for the SOU App.

The project’s truth layers are deliberately separate:

| Document | Authority |
|---|---|
| `MASTER_ARCHITECTURE.md` and current code | What actually exists now; code wins if they conflict |
| `PRODUCT_UX_SPEC.md` | What the product is resolved and designed to do |
| `PRODUCT_ROADMAP.md` | What is built next, in what order |
| Current Build Gap Analysis | Historical reconciliation snapshot; not a living source of truth |
| `PROJECT_FEATURE_MAP.md` | Superseded historical product thinking; retained only to explain earlier decisions and not a current source of truth |

This document uses four decision labels:

- **RESOLVED** — product behaviour has been decided.
- **IMPLEMENTED** — verified as built, but only to the extent stated.
- **UNRESOLVED** — a genuine product decision remains open.
- **DEFERRED** — intentionally postponed; absence of detail is not an oversight.

If product behaviour changes, update the relevant section and the change log in the same decision-making session. Never silently replace a resolved rule. Record what changed, why, and what it supersedes.

The former `SOU APP — CONSOLIDATED PRODUCT & UX SPECIFICATION.md` is superseded by this document and remains only as an archive of the 25 August consolidation.

---

## 1. Product vision

SOU is a shared operating and teaching environment for the School of Uke. It joins the school’s repertoire, arrangements, resources, courses, lessons, tutors, students and accumulated teaching memory without turning teaching into administration.

Its two mutually supporting product layers are:

1. **SOU operating product** — the practical system used to find and create teaching material, run courses and lessons, coordinate work, and retain institutional memory.
2. **Music-learning intelligence** — the gradually accumulated relationships between songs, skills, theory, teaching interventions, learners and outcomes.

The first layer must be independently useful. The second grows from ordinary teaching work rather than demanding a separate research or data-entry practice.

The central product loop is:

> **Plan → Teach → Appraise → Capture follow-ups → Carry forward → Plan next**

Every appraisal interaction should either take seconds or visibly make the next lesson easier.

### 1.1 Governing experience principles — RESOLVED

- Lead with curiosity, music and creation. Let organisation travel alongside the tutor.
- Help the tutor teach; do not ask the tutor to document their teaching while teaching.
- Songs lead. Theory, technique and exercises are normally taught through musical use.
- Structured objects and natural language are complementary, not competing interfaces.
- The lesson, song or other work object is primary; SA helps the tutor operate it.
- Organisation should be ambient rather than accusatory.
- Use progressive disclosure. Home gives awareness and memory cues, not reports.
- Capture structure as a quiet by-product of ordinary work.
- Preserve provenance, authorship, history and uncertainty. Never fabricate certainty.
- Personalise without enclosing: use a learner’s tastes as a doorway, while deliberately broadening repertoire and musical knowledge rather than algorithmically reinforcing a narrow profile.

---

## 2. Product domains

The platform has three distinct interface domains sharing one underlying object and service layer.

### 2.1 Admin — RESOLVED at boundary level; detailed UX DEFERRED

Admin operates and governs SOU: catalogue quality, accounts and permissions, scheduling and operational oversight, publishing, migration review, system exceptions and canonical curriculum/content governance.

Immediate Admin surfaces may favour functional clarity over polish. Admin is not the template for Tutor Studio; Tutor Studio is a distinct teaching product, not Admin with controls hidden.

Detailed information architecture and workflows remain deferred and must be designed from real administrative tasks before a broad Admin rebuild.

### 2.2 Tutor Studio — RESOLVED

Tutor Studio helps tutors teach, plan, create, reflect and develop musically. It supports both:

- **conversation-first work**, where an exploratory SA conversation produces or relates real Studio objects; and
- **object-first work**, where the tutor opens a Course, Lesson, Arrangement, Theory resource or other object and SA works alongside it.

Both routes act on the same objects. Chat must not become a parallel place where useful work is stranded as prose.

### 2.3 Student — architectural direction RESOLVED; detailed UX DEFERRED

The Student experience must ultimately expose the right lesson materials, learning journey, recordings and communications with appropriate permissions and low friction. Detailed Student UX remains deferred pending student consultation. Tutor/Admin delivery must not be delayed by speculative student-facing design.

---

## 3. Core domain model

### 3.1 Principal objects — RESOLVED

| Object | Product meaning |
|---|---|
| Song | Musical identity and catalogue record; not a particular teaching version |
| Arrangement | Editable structured interpretation of one Song for teaching/performance; key-flexible in the destination model |
| Print | A persistent published presentation of an Arrangement at a fixed print key |
| PDF file | A generated or preserved file attached to a Print; never the teaching object itself |
| Medley / Composite Arrangement | A teaching/performance object involving multiple Songs; not itself falsely modelled as a Song |
| Resource | A reusable teaching artefact such as theory material, exercise, game, recording or handout |
| Concept | An idea such as a skill or theory concept; not interchangeable with the Resource that teaches it |
| Course | A reusable course identity and teaching context, independent of one participant group |
| Cohort | The participant group for one specific run of a Course |
| Course Plan | Flexible intended arc for a Course run |
| Course Notes | Shared chronological, attributed notes stream for the Course context |
| Lesson | A scheduled teaching event |
| Lesson Plan | Canonical plan/workspace for that Lesson |
| Prompt Sheet | Delivery-focused projection generated from the Lesson Plan; not separately maintained |
| Appraisal | Low-burden record of what happened and what should carry forward |
| Student Journey | Persistent 1:1 learning context across lessons |
| Chat | Independent contextual conversation object |
| Action | Assigned follow-up work with context and lifecycle |
| Project | Possible cross-cutting workspace; long-term boundaries unresolved |

### 3.2 Relationship principle — RESOLVED

Objects participate in a reusable “relates to” network rather than being trapped in one folder hierarchy. Examples include:

`Resource ↔ Song ↔ Arrangement ↔ Skill ↔ Theory Concept ↔ Course ↔ Lesson ↔ Student ↔ Chat`

Relationships must remain explicit enough for people and SA to navigate, reason over and correct. A Chat or Resource may appear in several contextual histories without being duplicated.

Titles and subtitles are semantic data, not decorative labels. They should communicate the object’s teaching purpose well enough for tutors and SA to distinguish, find and relate it.

For reusable content, distinguish:

- **identity** — title and optional subtitle;
- **intrinsic relationships** — what the object is inherently about or designed to teach;
- **usage relationships** — where and how it has actually been used;
- **creation context/provenance** — why, how and from what it was created.

Search and SA retrieval may use all four, while keeping explicit facts separate from inferred/proposed relationships.

### 3.3 Concept versus Resource — RESOLVED

A Concept describes knowledge or capability: for example syncopation, a I–V–vi–IV progression or a particular chord transition. A Resource is a reusable artefact used to teach, practise or explain something. One Concept may relate to many Resources; one Resource may relate to several Concepts, Songs and Lessons.

### 3.4 Adaptation and lineage — RESOLVED in principle

Resources and Arrangements can be reused, adapted and related to their source. Material created while preparing a Lesson should be retainable as a reusable object without requiring a separate “contribute to library” administrative workflow.

The exact boundary between update, iteration, contextual adaptation and historical snapshot is **UNRESOLVED**; see Section 30.

---

## 4. Tutor Studio Home

### 4.1 Fixed core — RESOLVED

Home persistently provides:

- Chat / SA
- current date and Schedule doorway
- Actions indicator
- Notifications indicator
- Studio
- Groups when the tutor currently teaches groups
- Students when the tutor currently teaches 1:1 students

Foreground should favour SA/Chat, Music Inspiration, Studio/Create and personalised musical-tool shortcuts. Schedule, Groups, Students, Actions and Notifications remain quieter.

### 4.2 Date, Actions and Notifications — RESOLVED

- The current date is always visible and opens Schedule.
- Subtle upcoming-date markers indicate commitments without spilling event detail onto Home.
- Actions and Notifications show simple counts, not feeds.
- Notifications form one chronological history with lightweight labels such as Insight, Action, Admin, System and Schedule/change.
- “Needs Attention” is not a new object type; it points to an underlying Action or Notification.

### 4.3 Groups, Students and recent work — RESOLVED

Groups and Students act as memory cues, not miniature dashboards. Subtle interaction may reveal the next course/week, current songs or next 1:1 bookings. Studio may similarly reveal recent work without competing with the main Home experience.

### 4.4 Personal Home — RESOLVED

Tutors may add, remove and reorder shortcuts to tools, people and contexts—for example Chord Finder, Beats, Circle of Fifths, Scales, Catalog, Playlists, a Group, a Student or a Studio tool. Fixed SOU core destinations remain non-removable. Preferences sync across devices.

### 4.5 Music Inspiration — RESOLVED

Music Inspiration is a contained, finite area that rewards musical curiosity. It may use current/recent repertoire, artist and song connections, film/TV/game appearances, relevant musical ideas, connected tutor listening and SA-generated discoveries.

It must sometimes broaden knowledge instead of merely mirroring taste. It is distinct from formal SA Insights derived from teaching patterns and must avoid infinite-scroll distraction design.

---

## 5. SA — Studio Assistant

### 5.1 Role — RESOLVED

SA is the application’s contextual intelligence and collaboration layer, not an autonomous authority and not a generic chatbot pasted onto the product. It should be available wherever teaching content is created or reviewed and inherit the authorised context of the active object.

SA may help tutors:

- explore ideas and repertoire;
- retrieve relevant SOU history and materials;
- draft or restructure Plans and resources;
- translate natural teaching language into proposed structured relationships;
- surface patterns and carefully qualified insights;
- propose carry-forwards and follow-up Actions;
- identify missing, conflicting or uncertain information;
- operate tools only within explicit permissions and confirmation rules.

### 5.2 Truth and agency — RESOLVED

> **Tool truth beats conversational claim.**

A write has happened only when the responsible backend confirms it. SA must not claim that an object was created, changed, published, sent or assigned merely because it described doing so.

- General musical knowledge is allowed by default.
- Claims about SOU’s own records require actual retrieval/tool evidence.
- Ambiguity that could change the source, action or result must be clarified.
- Extraction, matching and inference must retain confidence and provenance.
- SA proposals remain editable and reviewable by humans.
- SA must never guess uncertain PDF-to-Arrangement matches, missing musical keys or canonical content.

### 5.3 Context — RESOLVED

In a Course, Lesson, Student Journey, Arrangement or Appraisal context, SA may access all authorised linked objects, including historical Plans, Appraisals, Registers, Actions, Songs, Resources, playlists, Chats and curriculum context. Relevance determines what it foregrounds. A persistent technical “SA context” indicator is not required.

### 5.4 Personalisation — RESOLVED

SA may learn a tutor’s musical tastes, teaching strengths, explanation preferences and working habits. Tutors can inspect, correct or remove inferred information.

For students, SA must **personalise without enclosing**. Existing preferences can help create engagement, but recommendations and planning should also broaden taste, context and capability. Student-profile data must not become a filter bubble.

### 5.5 SA Insights — RESOLVED

An SA Insight is a Chat initiated by SA because it has noticed something sufficiently useful to mention. Insights are sparse, appear as Notifications tagged Insight, open into ordinary Chats and remain searchable, renameable, organisable and deletable. There is no separate Insights dashboard. Thresholds are tuned through use.

---

## 6. Chats and Projects

### 6.1 Chats — RESOLVED

A Chat is an independent object with automatic origin/context relationships plus human curation. Tutors may rename it, add a subtitle, relate or unrelate it to other objects, or ask SA to organise it. The same Chat may appear in multiple relevant histories without copying its content.

### 6.2 Projects — UNRESOLVED

Projects may support intentional cross-cutting work that does not fit one Course, Song or other object. They must not become mandatory folders for every Chat. Their long-term purpose, boundaries and relationship to ordinary object networks remain open.

---

## 7. Actions

Actions are Studio-wide, arising from planning, appraisal, Student Journey work, Chat, content work or personal tutor organisation.

- Explicit tutor requests may create an Action through SA.
- If SA detects an implied commitment, it offers rather than silently creating it.
- Each Action has one assignee and retains its relevant object relationships.
- The original assigner controls assignment; the assignee cannot delegate onward.
- Tutor-assigned lifecycle: assign → work → assignee marks complete → assigner signs off → closed.
- Rejected completion can be returned/reopened with an optional note.
- A likely Admin Action is proposed for Lead confirmation. Once confirmed, responsibility transfers to Admin while teaching context and status visibility remain linked.
- Due dates and reminders are optional. Priority labels are not required.
- Everyday view is one simple searchable/filterable list. Older completed Actions remain searchable and available to SA.

Carry-forward teaching intentions (“revisit D→Em”) and operational Actions (“correct verse 2 on the sheet”) are distinct.

---

## 8. Tutors, roles and permissions

### 8.1 Contextual roles — RESOLVED

A tutor’s role varies by relationship.

- **Lead:** broad edit authority across the Course teaching context.
- **Shared Lead:** equal read/edit authority; every change remains individually attributable.
- **Assistant:** broad relevant read access; may update Register and add attributed Appraisal observations, but cannot complete the Appraisal or independently create lesson follow-up Actions.
- **Substitute:** relevant Course read access plus temporary editing while covering; access expires afterward.
- **1:1 tutor/content creator:** permissions follow the relevant Student or content relationship.

When a teaching relationship ends, current editing rights end; historical contributions and attribution remain, with appropriate historical read access.

### 8.2 Course-to-Lesson inheritance — RESOLVED

Course stores its Lead/Shared Lead/Assistant relationships. Every Lesson inherits those tutor assignments by default. A Lesson-level override is used for substitutions or exceptional staffing, rather than copying and independently maintaining tutor assignments for every Lesson.

### 8.3 Corrections and audit — RESOLVED

Authorised tutors may correct straightforward source-record errors. Corrections retain an audit trail and alert Admin. Ambiguous or consequential conflicts are escalated rather than silently overwritten.

### 8.4 Tutor profile — RESOLVED at product level

The private Tutor Account may include identity, bio, teaching philosophy, instruments, levels, child/adult suitability, locations, formats, styles, techniques, favourite teaching songs, musical interests, availability, connected media accounts, contact/settings, Admin-verified DBS status and private SA personalisation context.

Any future public tutor profile is a separate projection. Public-profile and social design are deferred.

### 8.5 Tutor Studio without active teaching — RESOLVED

Tutor Studio remains useful when a tutor has no current SOU Groups or 1:1 students. Home naturally simplifies around SA, Studio, Music Inspiration, Catalog, playlists, Chats/Projects, Actions and musical tools. A former or temporarily inactive tutor may retain an account subject to ordinary access and data-retention policy.

### 8.6 Tutor onboarding — RESOLVED for immediate scope

The immediate flow is: Admin creates/invites Tutor → Tutor establishes account → basic profile/setup → Tutor Studio. An elaborate tour, questionnaire or preference-mining sequence is not required for initial use.

---

## 9. Schedule and booking

SOU Calendar contains SOU commitments only: group and 1:1 Lessons, events, meetings, training and similar work. SOU should push/sync commitments to tutor-selected external calendars rather than replace personal calendars.

Sesami/Shopify remains the near-term booking foundation. The product direction is a native-looking SOU Schedule over that service where practical; a fully native booking/availability engine is later scope.

---

## 10. Course and Cohort

### 10.1 Course — RESOLVED

Course is the teaching identity and context: title/level, venue and schedule, tutor relationships, Course Plan, Lessons, playlist, curriculum relationships, Notes, Appraisals and Actions.

A Course may begin as a **lightweight operational shell**, created by Admin or an authorised tutor with only the information needed to run it. The teaching arc is optional at creation. A tutor may later develop/adapt the arc directly or with SA in relation to curriculum, repertoire and the actual Cohort.

Course is not identical to Curriculum. Tutors may locally diverge and experiment in their Course Plans. Canonical Curriculum changes require authorised Curriculum/Admin approval.

### 10.2 Cohort — RESOLVED

Cohort is a distinct participant-group object for one specific Course run.

- Course ≠ Cohort.
- Students persist independently and may join different Cohorts over time.
- A Cohort does not persist as the same group across unrelated Course runs.
- Attendance, group-level appraisal and the group’s teaching history naturally anchor to Cohort and its Lessons.
- The model must permit repeat runs of the same Course identity with different Cohorts.

### 10.3 Course landing and journey — RESOLVED

The landing view orients the tutor with Course details, current position, Cohort/master Register, Course Plan, Lessons, Lesson Plans and repertoire. Position is derived from schedule/status—for example Week 5 of 10.

All scheduled Lessons appear as past, current or future entries. Future Lessons may show derived planning state such as Ready, Draft or Not started. Past Lessons expose Plan, Appraisal and content quickly.

### 10.4 Course Plan — RESOLVED

Course Plan is a flexible intended roadmap, not a rigid contract. It reflects the current point in the Course and retains history when intentionally revised. The difference between intended Plan and actual Lesson/Appraisal history is valuable teaching evidence.

Course Plan generation/drafting may use four inputs:

1. canonical Curriculum and relevant prior Course structure;
2. Cohort/student context;
3. repertoire/playlist and available teaching Resources;
4. **Course Notes**, including attributed chronological observations.

### 10.5 Course Notes — RESOLVED

Course Notes is one shared chronological stream. Each entry records author/account, contextual role and timestamp. It supports practical memory that does not yet belong in a formal Plan, Appraisal or Action and is an explicit input to Course Plan development.

### 10.6 External ecosystem — RESOLVED direction; transition duration UNRESOLVED

Course may relate to:

- a WhatsApp group;
- a Google Drive folder or equivalent document store;
- a Spotify playlist;
- later provider-neutral replacements or integrations.

Spotify should progress from linking to direct playlist creation/management where feasible. The current live playlist relationship should remain useful, while historical teaching records preserve a snapshot of playlist membership relevant to the period being reviewed.

Which external operations remain represented during migration, and for how long, is unresolved. The architecture should describe capabilities and relationships without becoming provider-specific at its core.

---

## 11. Repertoire and playlists

The Course playlist is the repertoire palette or “totem pole.” It contains Songs, not Arrangements.

The normal progression is:

> Playlist Song → potential repertoire → teaching choice → Arrangement choice → Lesson Plan → taught/appraised Lesson

Adding a Song to a playlist does not select an Arrangement or commit it to a Lesson.

A Course normally draws flexibly from roughly 6–10 songs rather than obeying one fixed repertoire shape. This is a working teaching pattern, not a validation rule.

---

## 12. Lessons and planning

### 12.1 Lesson identity and status — RESOLVED

Group Lessons belong to Course/Cohort and have Scheduled, Took place or Cancelled status. Group attendance is Present/Absent and may be updated by Lead or Assistant. Register must be complete before Appraisal is processed.

1:1 Lessons require reliable Scheduled, Took place, Cancelled and—where relevant—No-show state, but no Register.

### 12.2 Full Lesson Plan — RESOLVED

The Full Lesson Plan is the canonical planning record and workspace. It may contain:

- Course/Cohort or Student context;
- previous Lesson/Appraisal and open Actions;
- Course/Journey direction and curriculum context;
- objectives and intended teaching;
- ordered lesson chunks;
- Songs and selected Arrangements;
- song-specific teaching prompts;
- related Concepts and Resources;
- notes and optional timings.

Tutors can assemble the Plan directly using structured blocks or describe intent in ordinary teaching language while SA proposes the same editable blocks. SA output must enter the workspace as real proposed structure, not remain a chat answer to be copied manually.

### 12.3 Prompt Sheet — RESOLVED

The Tutor Prompt Sheet is a delivery-focused projection of the Full Lesson Plan, never an independently maintained second plan.

It foregrounds only what is useful in the room:

- main objectives/teachings;
- ordered chunks such as Warm-up, Song, Break and Finale;
- Song, artist, release context and selected key where relevant;
- relevant chords, transitions, TAB, theory, rhythm/picking and teaching notes;
- optional timings.

Unused headings disappear. The Prompt Sheet may be viewed digitally or published as a Print/PDF output.

### 12.4 Practical lesson-shape guidance — RESOLVED as guidance, not schema

- Group lessons commonly include a 15-minute break.
- Theory should normally be embedded in Songs rather than isolated by default.
- A Lesson should deliberately end on a high: something musically satisfying, achievable or energising.
- These are planning and SA guidance, not mandatory fixed templates.

### 12.5 Lesson Hub — RESOLVED

Opening an upcoming Lesson gives equal/direct access to Prompt Sheet, Full Plan, Lesson Content and Course Plan/Student Journey. Previous Lesson, previous Appraisal and current Songs remain subtly available. Exact visual hierarchy should be tested rather than frozen prematurely.

---

## 13. Teaching Mode

Teaching Mode is manually entered and visually distinct. It presents the Prompt Sheet like a setlist, with previous/next, free jumping and tap-to-reveal detail.

Potential tools include Lesson Content, Register, playlist/reference player, Metronome, Beats, Recording, SA and personalised shortcuts.

- Do not require live Done/Skipped/Unfinished tracking.
- Do not require note-taking while teaching.
- SA is quiet/reactive by default; voice access is prominent but not always listening.
- The screen may remain awake and live state may continue across devices.
- Multiple simultaneous devices may act as different windows into the same Lesson state.

Recordings are separate media objects related to Lesson, Course/Cohort, Student, Song/Arrangement and tutor as appropriate. Consent, retention, access and student-facing use require explicit policy before broad rollout.

---

## 14. Appraisal and carry-forward

### 14.1 Lesson Appraisal — RESOLVED

Appraisal is lightweight and adaptive. It uses the Plan and known Lesson context so it does not ask the tutor to restate obvious facts.

A typical flow asks:

1. How did they get on?
2. Anything else worth remembering?
3. Anything you need to do before next time?

Voice and free text are first-class. SA may propose structured observations, carry-forwards and Actions for tutor confirmation. Assistants may contribute attributed bullets; Lead/Shared Lead completes the Appraisal.

Saving should immediately show what has been carried into the next planning context.

### 14.2 Completion — RESOLVED

Lesson completion does not demand a bureaucratic checklist. Reliable status, completed Register where applicable and an appropriately completed/skipped Appraisal are enough. The product should handle late or partial reflection without pretending it occurred.

### 14.3 Course Appraisal and institutional memory — RESOLVED

End-of-Course reflection can compare intended arc, actual Lessons, Cohort progress, successful/unsuccessful Resources and unresolved issues. It should preserve useful group-level teaching memory without converting students into scores or treating correlation as causation.

---

## 15. 1:1 Student Journeys

A Student Journey is the continuing learning context across 1:1 Lessons. It includes goals, repertoire, relevant profile/taste, Plans, Appraisals, Actions, Resources, recordings and an adaptable longer arc.

The tutor should see what matters next, not a clinical case-management dashboard. SA may help connect current interests to broader musical development under the “personalise without enclosing” principle.

Student Journey Plan is intended direction; Lessons and Appraisals are the actual journey. Optional milestones may be created and marked achieved with Tutor+SA confirmation. Gamified rewards are deferred.

Detailed Student-facing visibility, editing and communications remain deferred pending consultation.

---

## 16. Studio creative workspace

Studio exposes creation/review tools for Arrangements, Prints, TAB, Theory, Exercises/Games, Lesson Plans and other Resources. It supports large-screen split workspace plus contextual SA. Mobile prioritises teaching, review and lightweight editing; sophisticated creative authoring may remain desktop/tablet-first initially.

Content created during normal lesson preparation can quietly become reusable. The tutor should not have to leave their task and complete a separate contribution form merely to grow SOU’s library.

### 16.1 Ownership, drafts and publishing — RESOLVED

- Draft work belongs to its creator/collaborators and remains private to the appropriate workspace.
- Publishing makes an authorised canonical/shared version available to the SOU library.
- Publishing status is distinct from whether a PDF has been generated successfully.
- Contributions and later changes retain attributable authorship.
- Shared editing must not erase who did what.
- Likes/favourites are lightweight personal signals. “Report a problem” creates a traceable Admin correction workflow, does not automatically suppress the content, and notifies the reporter when the reviewed outcome is signed off.

---

## 17. Song, Arrangement and Medley

### 17.1 Song — RESOLVED

Song represents musical identity and catalogue/discovery metadata. It is dynamic and independent of any particular SOU teaching interpretation, key, layout or PDF.

### 17.2 Arrangement — RESOLVED destination

Arrangement is structured, editable teaching/performance content related to one Song. It preserves lyrics, chords, section structure, annotations, authored spelling and chord-to-lyric positioning. An Arrangement is not a PDF and should not be reduced to one fixed key.

Beginner, intermediate and advanced musical treatments are normally distinct Arrangements because their musical/teaching content differs. A mere change of view or Print key does not by itself create a new Arrangement.

The persistent key model is:

- Arrangement: key-flexible content with tutor-confirmed transposition origin where available;
- Current View Key: transient interface state;
- Print: fixed `print_key`;
- Lesson activity: optional `lesson_key` when that later model is implemented.

The existing legacy `default_teaching_key` is transitional and conceptually deprecated once Print Key selection replaces it.

### 17.3 Arrangement history rule — RESOLVED reconciliation

Forward-looking contexts may deliberately follow the current published Arrangement or Print. Completed/historical teaching records must preserve the exact content iteration or immutable snapshot actually used.

Therefore:

- a future Lesson Plan may “float” to current publication when the tutor deliberately chooses that behaviour;
- at the point of teaching/completion, the record fixes the exact iteration/snapshot used;
- later Arrangement changes never rewrite historical Lesson evidence;
- a deliberate contextual adaptation may retain lineage without silently replacing its source.

The exact authoring UI and thresholds for creating a new iteration versus updating remain **UNRESOLVED**.

### 17.4 Medley / Composite Arrangement — partially RESOLVED

A medley is not a fake Song. It requires its own composite structure and must relate to its component Songs/Arrangements while retaining its own order, transitions and teaching/performance notes.

The canonical cardinality and snapshot/version behaviour for component relationships remain **UNRESOLVED**.

---

## 18. Song Canvas

### 18.1 Product purpose — RESOLVED

Song Canvas is the digital consumption and creation environment for a Song and its selected Arrangement. It should ultimately combine the readability and direct usefulness tutors/students expect from mature chord-sheet products with SOU’s teaching context, structured native content and trustworthy publication history.

Ultimate Guitar is a consumption-UX benchmark—particularly speed of finding, opening, reading, scrolling and changing view—not a provider-specific architecture or a mandate to copy its business model.

### 18.2 Canvas capabilities — RESOLVED direction

Depending on role, device and implementation phase, Canvas may provide:

- lyrics/chords with faithful spatial alignment;
- section navigation and collapse/expand;
- selected Arrangement and publication history;
- transient view transposition when technically safe;
- explicit key/accidental display;
- playback/reference links;
- tempo/metronome/Beats access;
- teaching notes, TAB, Concepts and related Resources;
- Print/PDF generation and access;
- contextual SA;
- editing/review tools for authorised creators.

The intended creation flow is: Create → find/confirm the correct Song and recording → inspect existing published Arrangements → create a Draft Arrangement. Where evidence, rights and source quality allow, a new Draft may be pre-populated with candidate lyrics, chords, detected sections, useful metadata and reference media, but nothing uncertain becomes canonical without review.

The authoring destination includes:

- section detection with rename, hide/remove, reorder, split, merge and repeat controls;
- linked repeated sections with per-occurrence variation when the later content model supports them;
- an editable harmonic timeline capable of simplification, embellishment, substitution and changes in harmonic rhythm;
- automatically derived chord palette and context-appropriate voicing choices;
- Performance Notes related to a section, phrase, lyric, chord or bar;
- structured lyrics/chords in the main Song flow, with freer teaching overlays where needed.

Canvas must not become a general desktop-publishing application. Print composition is a purpose-specific projection.

### 18.2.1 Transposition and playback — RESOLVED destination; largely DEFERRED

Canvas should ultimately offer dedicated target-key and semitone transposition, updating chord symbols and diagrams only when the Arrangement is demonstrably transposition-ready. Section-level modulation belongs in the intended model. A Course/Lesson or Student context may remember its selected teaching/lesson key; that selection does not rewrite the Arrangement. The Arrangement’s confirmed transposition origin remains musical reference data, while a Print alone fixes a publication key.

Reference playback may use Spotify, YouTube or other replaceable providers and should ultimately support play/pause, scrub/jump, speed without pitch change, pitch shift and looping. Preferred looping selects a lyric line, phrase or section; waveform/timeline provides fallback fine control. When view key changes, reference audio may follow by default, with an explicit reset/match control. Synchronous lyric/chord/section following is later scope.

### 18.2.2 Metronome and Beats — RESOLVED destination

Metronome and Beats are shared tools across Canvas, Teaching Mode, Home shortcuts and relevant Theory/Exercise contexts. Song analysis may propose BPM and time signature; tutors can correct them. Tap Tempo is shared.

Beats keeps the surface simple—choose groove → sound → BPM → play—over a deeper high-quality library of genres, feels, sounds and time signatures. Canvas may suggest a few plausible grooves while still allowing full browse. Related variations such as Simple/Standard/Busy/A/B/Fill should switch cleanly at a musical boundary. Do not turn the initial tool into a section sequencer or DAW. An Arrangement may remember selected Beat settings and Beats has its own simple volume control.

The first implemented authoring route—paste, parse, review/edit, persist—is a reduced-fidelity entry path into the broader Song Canvas destination, not the completed Canvas UX.

### 18.3 Device principle — RESOLVED

Reading and teaching use must work well on phone. Deep positional editing and complex composition can be optimised for larger screens first without making mobile content inaccessible.

### 18.4 Instruments and tunings — DEFERRED but architecturally required

The core model must not hard-code standard ukulele tuning as the only future profile. Later instrument/tuning profiles should support alternatives including baritone ukulele DGBE. Exact transposition, fingering, chord-diagram and teaching behaviour is deferred.

---

## 19. Prints, PDFs and publication

### 19.1 Canonical relationship — RESOLVED

> **Song → Arrangement → Print → PDF file**

- Arrangement owns editable musical content.
- Print is a persistent publication/composition at a fixed key and contains an immutable content snapshot appropriate to that publication.
- A PDF is one generated or legacy file manifestation associated with the Print.
- Lesson Plans reference Arrangements—and, where a fixed handout matters, an explicit Print—not PDFs masquerading as teaching objects.

### 19.2 Publication lifecycle — RESOLVED

Publishing and file generation are separate facts:

1. publish the authorised Arrangement/Print record locally;
2. generate and store the PDF;
3. if generation fails, publication remains valid and generation is retryable;
4. deterministic file identity prevents orphaned retry files;
5. infrastructure/provider details stay behind replaceable boundaries.

No system may infer a missing print key. It must ask or fail clearly.

### 19.3 Historical integrity — RESOLVED

Prints preserve the fixed key and content snapshot used for distribution or historical teaching. Regenerating a file for the same unchanged Print may replace its file manifestation; changing meaningful musical content creates or points to a different snapshot/iteration according to the still-to-be-finalised versioning rules.

The main Song Print is intended to support a carefully composed A4 landscape teaching sheet. Automated layout may propose compression, but the tutor retains editorial/layout control. Changes that would invalidate manual layout decisions trigger review rather than silently destroying them. TAB and Theory may also have their own deliberately created Print/PDF publications; dynamic transposition does not imply that a PDF exists for every possible key.

### 19.4 Legacy PDFs — RESOLVED

- Preserve every existing SOU Arrangement PDF.
- Relate it to the correct Song, Arrangement and fixed key/Print where evidence allows.
- Uncertain matches enter manual review; they are never guessed.
- Text extraction supports search, comparison and migration, but does not automatically create trustworthy canonical Arrangement content.
- Raw extracted/source material remains available for provenance and debugging.
- A tutor reviews and corrects structured content before it becomes canonical.

### 19.5 Licensing and providers — RESOLVED direction

Architecture should express rights, provenance, source and permitted uses without binding the product to one commercial provider. Future licensed repertoire/catalogue interoperability should remain possible. Provider integrations are replaceable adapters, not the canonical Song/Arrangement model.

---

## 20. Catalogue migration

### 20.1 Migration destination — RESOLVED

Every Song in the SOU teaching Catalog must ultimately have a digital Song Canvas. This is mandatory catalogue migration, not optional conversion only when a Song happens to be edited.

Migration is staged and auditable:

1. inventory every catalogue Song and every known PDF/resource;
2. identify high-confidence Song/PDF relationships;
3. queue uncertain, conflicting or duplicate matches for human review;
4. retain original files and raw extraction/import text;
5. extract/parse candidate lyrics, chords, headings, blank-line structure and chord-to-lyric position;
6. review and correct candidates;
7. persist native structured Arrangement content;
8. create/link fixed-key Prints and preserved legacy PDFs;
9. mark migration completeness and unresolved exceptions explicitly.

No stage may silently discard source material or promote uncertain extraction to canonical truth.

### 20.2 Native structured content — RESOLVED

Canonical Arrangement persistence must represent at least:

- lyrics as text lines;
- chord symbols and decomposed pitch information where known;
- chord-to-lyric positional anchors;
- section headings and ordering;
- blank-line/spacing structure needed for faithful teaching display;
- annotations where supported;
- schema version and provenance.

Raw pasted/imported text is retained unchanged for provenance/debugging but is never rendered as the competing canonical source after review.

---

## 21. Theory, Exercises, Games and other Resources

### 21.1 General Resource model — RESOLVED

Theory sheets, exercises, games, recordings and similar materials are reusable Resources with identity, title/subtitle, ownership, status, lineage and explicit relationships.

A Resource can relate to multiple Songs, Arrangements, Concepts, levels, Courses and Lessons. Reuse must not require duplication. A contextual adaptation may remain related to its source.

### 21.2 Theory Studio — RESOLVED direction

Theory Studio supports small useful explanations and “nuggets” as well as fuller sheets. Theory is normally connected to a musical need or Song rather than presented as abstract syllabus administration. SA can help draft and relate content, with tutor review before publication.

### 21.3 Exercises and Games — RESOLVED direction

Exercises/Games are first-class reusable Resources, not miscellaneous attachments. They can record purpose, relevant Concepts/skills, level/context, instructions and related Songs or Lessons. Their design should grow from real SOU teaching practice rather than an invented comprehensive taxonomy.

### 21.4 TAB — RESOLVED direction; implementation DEFERRED

TAB may exist within an Arrangement or as a related Resource depending on its scope and reuse. Authoring, rendering, playback and import behaviour require dedicated design and must not be improvised as plain text if doing so destroys musical meaning.

For an interim image-based TAB workflow, authorised tutors may upload/paste, crop, resize, position and remove backgrounds/transparency. TAB may appear in a main Song Print where space permits or in a separate landscape TAB Print. Native interactive TAB remains later scope and may supersede this interim representation.

---

## 22. Catalog and discovery

Catalog supports finding existing SOU Songs and relevant teaching material. Discovery may extend beyond SOU’s own teaching catalogue to help identify potential repertoire, but promoting a discovered Song into SOU creates/links a governed Song record and never implies that licensed lyrics, chords or arrangements were acquired.

Search must distinguish metadata search, native-content search and legacy-extracted-content search. SA must describe which source it actually searched.

---

## 23. Student-facing content access

Architecture must allow authorised Students to receive or revisit the exact relevant Lesson materials without exposing drafts, private tutor notes, other students’ information or Admin data.

Potential future capabilities include current Lesson content, selected Prints, recordings, practice prompts and journey context. Exact navigation, notification, consent, download and annotation behaviours are **DEFERRED pending student consultation**.

### 23.1 Remote/online lessons — DEFERRED direction

Near term, Tutor Studio supplies Lesson context while tutor and student use an existing call platform and independently open the relevant SOU material. A later native Studio Room may add music-friendly audio/video, tutor-controlled student view, phrase jump/rewind/loop and shared playback state. This must not block Tutor MVP.

---

## 24. Admin boundary

Admin eventually needs coherent controls for:

- accounts, roles and permissions;
- Songs, Arrangements, Prints, Resources and publishing;
- migration queues and uncertain matches;
- Courses, Cohorts, schedules and tutor assignments;
- curriculum governance;
- content reports, corrections and audit;
- integration status and operational exceptions.

This list defines responsibility, not a decided dashboard layout. Detailed Admin UX is **DEFERRED**.

---

## 25. Cross-cutting product requirements

### 25.1 History and attribution — RESOLVED

Material changes retain actor, time and relevant source/context. Historical teaching records remain stable even when current objects evolve. Avoid parallel copies of ownership or identity facts that can drift.

### 25.2 Search — RESOLVED direction

Search spans objects and relationships while respecting permission and source truth. It should support direct lookup and contextual discovery without pretending inferred relationships are confirmed ones.

### 25.3 Offline and continuity — DEFERRED direction

Teaching-critical views should tolerate weak venue connectivity where practical. Exact offline editing, conflict and sync behaviour is not yet designed.

### 25.4 Security and privacy — RESOLVED principle

Access follows the tutor/student/admin relationship and least privilege. Private notes, student data, recordings and SA personalisation require explicit boundaries, auditability and appropriate retention/consent policy.

### 25.5 Export and portability — RESOLVED principle

SOU must be able to export its canonical structured content, relationships and files in usable forms. Provider integrations must not become data prisons.

### 25.6 White-label/commercial future — DEFERRED

The core object model may later support other teaching organisations, but near-term SOU utility and clarity take precedence. No current feature should become abstract merely to simulate future multi-tenant needs.

---

## 26. Product destination versus delivery horizons

### 26.1 Product destination

The complete Tutor Studio operating loop, digital Song Canvas, structured Resource network, Course/Cohort teaching memory, contextual SA and appropriate Admin/Student experiences described above.

### 26.2 Immediate/internal utility

The shortest coherent internal path remains:

1. trustworthy native Arrangement authoring and migration;
2. Print/PDF publication;
3. lightweight Course/Lesson/Plan authoring;
4. Prompt Sheet delivery;
5. low-burden Appraisal/carry-forward.

This is a delivery slice, not a reduction of the destination model.

### 26.3 Later development

Full Tutor Studio Home, contextual SA across all objects, advanced Course/Cohort operations, student UX, detailed Admin redesign, sophisticated pedagogy intelligence, native booking, offline collaboration, full transposition/playback, TAB and instrument/tuning profiles come later according to Roadmap sequencing.

---

## 27. Reconciliation with the verified build

The following table reflects the current documented implementation as at 20 September 2026. `MASTER_ARCHITECTURE.md` and code remain authoritative for exact implementation truth.

| Area | Verified current state | Product-spec gap |
|---|---|---|
| Existing Catalog/Admin | Mature song metadata catalogue, discovery/enrichment and Admin management exist | Not yet the full governed object network or redesigned Admin UX |
| SA/Chat | Conversation engine and tool-integrity rules exist | Contextual SA across Course/Lesson/Canvas and full object operations largely absent |
| Arrangement persistence | Structured native Arrangement schema, provenance, parser, validation and chord decomposition implemented | Full Song Canvas, mature history/version UX, bulk migration and composite Arrangements absent |
| Arrangement authoring | Paste → parse-preview → paired-row review/edit → persist/reopen implemented | This is an entry workflow, not the complete Canvas/editor experience |
| Key foundation | Pitch-class chord data and tutor-confirmed transposition origin implemented | Current View Key, Print Key UI, Lesson Key and transposition controls not implemented; legacy `default_teaching_key` remains transitional |
| Print/PDF | Resource snapshot, publish/generate lifecycle, PDF rendering, storage ledger and a single Publish PDF action implemented | Refined persistent Print UX, key selection, legacy-PDF migration/review and broader publication management remain incomplete |
| Courses/Lessons | Basic Course and scheduled Lesson persistence/endpoints plus minimal Course→Lesson navigation implemented | Cohort, roles/inheritance, Course Notes, Course Plan, ecosystem integrations and complete UI absent |
| Lesson Plans | One structured Plan per Lesson with minimal Arrangement/timing/notes chunks, persistence and a verified direct chunk-authoring UI implemented | Rich plan blocks, contextual/SA planning, Prompt Sheet generation and historical fixing not complete |
| Prompt Sheets | Data/file destination anticipated | Generation and delivery UX not implemented |
| Appraisal/carry-forward | Product design resolved | Not implemented |
| Tutor Studio Home/Teaching Mode | Product design resolved | Not implemented as the designed experience |
| Student UX | Deliberately deferred | Not implemented/designed in detail |
| Detailed Admin UX | Deliberately deferred | Existing functional Admin is not the final design |
| Catalogue migration | Destination and integrity rules resolved | Complete inventory, matching/review workflow and mandatory Canvas migration not implemented |

Implementation must never be inferred from the richness of the destination specification.

---

## 28. Confirmed decisions not to lose

This section is a compact protection against future consolidation drift.

- Course may start as a lightweight shell; SA-assisted arc development comes later.
- Course tutor assignments inherit to Lessons and are overridden only when necessary.
- Cohort is a distinct run-specific participant group.
- Course Notes is an attributed chronological planning input.
- Course relationships may include WhatsApp, Drive and Spotify, with historical playlist snapshots.
- Playlist contains Songs, not Arrangements.
- Typical Course repertoire is flexible, often 6–10 songs.
- Group Lesson guidance includes a 15-minute break and deliberately ending on a high.
- Theory should normally be taught through Songs.
- Resources form a reusable relates-to network; Concept is not Resource.
- Resource creation should be a quiet by-product of preparation.
- Personalisation broadens rather than encloses.
- A medley is not a Song.
- Every catalogue Song ultimately receives a digital Song Canvas.
- Every legacy PDF is preserved and linked where evidence supports the match.
- Uncertain matches are reviewed, never guessed.
- Extraction is not canonical authorship.
- Lesson Plans never use PDFs as substitute teaching objects.
- Historical Lessons fix what was actually used; future Plans may intentionally float.
- Native content is canonical; Print/PDF is a publication/output layer.
- Raw imported text remains provenance, not rendering truth.
- Provider/licensing architecture remains interoperable.

---

## 29. Deferred discovery

The following absences are intentional:

- detailed Student UX pending consultation;
- detailed Admin UX based on real operational task analysis;
- public Tutor Profile/social features;
- native booking engine;
- sophisticated offline editing/sync;
- exact recording consent/retention experience;
- full TAB model and UX;
- full multi-instrument/tuning behaviour;
- advanced playback and automated arrangement generation;
- comprehensive pedagogy ontology before historical-corpus analysis.

Deferred items must not be treated as permission to improvise implementation-level product decisions.

---

## 30. Open decisions requiring future answers

Only genuinely unresolved questions belong here.

1. **Composite/medley model:** What is the canonical relationship between a Medley, its component Songs/Arrangements, ordering, transitions and historical snapshots?
2. **Arrangement/Resource change semantics:** Precisely when does a change update an object, create an iteration, create a contextual adaptation or preserve a separate historical snapshot?
3. **Projects:** What durable job does Project perform that cannot be met by ordinary object relationships and contextual views?
4. **External Course migration:** Which Spotify, WhatsApp and Drive operations remain represented during transition, and what is the exit/retention policy for each?
5. **Student UX:** What do students actually need and consent to see, do and retain?
6. **Admin UX:** What is the task-based information architecture for governance, migration and operations?

These questions should be answered with focused evidence or consultation. They do not block using the many resolved rules elsewhere in this document.

---

## 31. Decision and maintenance protocol

When a future product decision is made:

1. identify whether it changes product destination, delivery horizon or implementation only;
2. update the relevant section here if it changes resolved product/UX behaviour;
3. update `MASTER_ARCHITECTURE.md` only for verified architecture/build truth;
4. update `PRODUCT_ROADMAP.md` only if sequence or priority changes;
5. add a dated change-log entry below;
6. preserve unresolved status where evidence is insufficient;
7. flag cross-document conflict explicitly instead of silently harmonising it.

Before implementation, proposals must state which resolved requirements they satisfy, which deferred/unresolved areas they avoid, and whether they require a new product decision.

---

## 32. Change log

### 20 September 2026 — Recovery revision 1.0

- Promoted the August consolidation into the intended canonical `PRODUCT_UX_SPEC.md` form.
- Recovered the omitted Cohort, Course Notes, Resource-network, personalisation, Course-ecosystem, lightweight-shell, tutor-inheritance, practical lesson-shape, medley and quiet-resource-creation decisions.
- Folded in verified post-August decisions for mandatory catalogue migration, legacy PDF integrity, native structured Arrangement content, provenance, Print/publication/key architecture, Canvas benchmark, tuning extensibility and provider-neutral licensing.
- Reconciled historical exactness with forward-looking floating references.
- Separated product destination from current implementation and recorded the verified September build boundary.
- Preserved genuinely unresolved and deliberately deferred decisions without inventing answers.
- During review, clarified that the file remained a draft until explicit approval and recorded `PROJECT_FEATURE_MAP.md` as superseded historical material.

### 20 September 2026 — Canonical approval

- Approved by Matthew following independent review against the recovered Q&A, omission analysis and current architecture.
- Promoted from recovery draft to the canonical living Product & UX Specification, version 1.0.
- Supersedes the former consolidated specification; future resolved product decisions must be incorporated here under Section 31.
