#   
**SOU APP — CONSOLIDATED PRODUCT & UX SPECIFICATION**  
  
**Status:** Consolidated design decision record — pre-implementation  
**Date:** 25 August 2026  
**Scope:** Immediate Tutor/Admin/internal-SOU product direction arising from the August 2026 design/UX Q&A  
**Student UX:** Deliberately deferred pending student consultation  
**Implementation status:** This document describes resolved product behaviour and intended architecture. It is NOT evidence that features are implemented.  
  
  
  
**1. Purpose**  
  
This document consolidates the decisions reached during the August 2026 SOU App design/UX Q&A into a coherent product specification. It replaces reliance on the conversation transcript as the source of product truth.  
  
It should be read alongside:  
  
	●	MASTER_ARCHITECTURE.md — current technical state and implemented truth  
	●	PRODUCT_ROADMAP.md — forward sequencing and product priorities  
	●	existing product-vision documentation, subject to reconciliation  
  
Governing rule:  
  
> **Current code and `MASTER_ARCHITECTURE.md` describe what exists. This specification describes resolved product/UX behaviour. `PRODUCT_ROADMAP.md` determines sequencing.**  
  
Where later Q&A decisions superseded earlier ones, the later decision is used here.  
  
  
  
**2. Product shape**  
  
**2.1 Three interface domains**  
  
**Admin**  
  
Purpose: operate and govern SOU. Functional controls matter more than polish for the immediate MVP.  
  
**Tutor Studio**  
  
Purpose: help tutors teach, plan, create, reflect and develop musically. Tutor Studio is a distinct tutor-facing product experience, not merely the existing Admin UI with permissions hidden.  
  
**Student**  
  
Detailed Student UX is deliberately deferred until SOU students can be consulted. The architecture should remain compatible with a future Student interface, but immediate Tutor/Admin MVP must not be delayed by speculative Student UX.  
  
All three share the same underlying platform, data and services.  
  
  
  
**3. Tutor Studio experience principles**  
  
Tutor Studio should feel inviting, musically alive and creatively useful, not like a productivity dashboard that greets tutors with overdue work.  
  
Core principle:  
  
> **Lead with curiosity, music and creation. Let organisation travel alongside them.**  
  
Organisation should be ambient rather than accusatory.  
  
The same information hierarchy and core capabilities should exist across desktop, tablet and phone, while sophisticated Studio creation can be optimised for larger screens first. Teaching/use workflows on phone matter; full Creative Studio phone optimisation is deferred.  
  
Use progressive disclosure: Home gives awareness and memory cues, not reports.  
  
  
  
**4. Tutor Studio Home**  
  
**4.1 Fixed SOU core**  
  
Persistent Home elements:  
  
	●	Chat / SA  
	●	current date / schedule doorway  
	●	Actions indicator  
	●	Notifications indicator  
	●	Studio  
	●	Groups, only if tutor currently teaches groups  
	●	Students, only if tutor currently teaches 1:1 students  
  
**4.2 Dominant Home elements**  
  
Foreground should favour:  
  
	●	SA / Chat  
	●	Music Inspiration  
	●	Studio / Create  
	●	personalised musical-tool shortcuts  
  
Schedule, Groups, Students, Actions and Notifications should be more subtle.  
  
**4.3 Date / Schedule**  
  
Current date and upcoming schedule cues can share one row/block, e.g.:  
  
Saturday 22 August     📅 25   📅 27   📅 29  
  
Rules:  
  
	●	current date is always visible and clickable  
	●	clicking it opens Schedule/Calendar  
	●	upcoming date markers show only that commitments exist on those dates  
	●	event detail remains behind Schedule  
	●	if no upcoming commitments exist, the current date still opens Schedule  
  
**4.4 Actions / Notifications**  
  
Home shows simple counts only, e.g. Actions · 3 and a subtle unread Notifications count/icon.  
  
No task or notification feeds should spill onto Home.  
  
**4.5 Groups / Students memory cues**  
  
Groups and Students are conditional Home destinations.  
  
Subtle interaction may reveal:  
  
	●	Groups: next-up course/week and current songs  
	●	Students: next couple of scheduled 1:1 bookings  
  
Avoid mini dashboards.  
  
**4.6 Studio recent work**  
  
Studio may reveal subtle recent-work reminders on interaction. Recent work is memory support, not a major competing block.  
  
**4.7 Personal Home**  
  
Tutor can customise a shortcut area, analogous to a phone/desktop home screen.  
  
Possible shortcuts:  
  
	●	Chord Finder  
	●	Beats  
	●	Circle of Fifths  
	●	Scales  
	●	Catalog  
	●	Playlists  
	●	a particular Group or Student  
	●	frequently used Studio tools  
  
Initial implementation can be simple: Add to Home / Remove from Home / Reorder. Customisation syncs across devices. Fixed SOU core remains non-removable.  
  
  
  
**5. Music Inspiration**  
  
Music Inspiration is a contained but meaningful Home area designed to encourage tutors to visit for musical curiosity, not only admin.  
  
It may draw from:  
  
	●	artists/songs in current and recent course playlists, approximately last 3 months  
	●	current/planned teaching songs  
	●	song/artist appearances in film, TV and games  
	●	theory/music connections relevant to current students  
	●	tutor’s own connected Spotify/YouTube interests if linked  
	●	SA-generated musical nuggets  
  
SA nuggets should play a strong role and sometimes broaden musical knowledge rather than merely mirror taste.  
  
Music Inspiration should show a limited number of highly relevant items, avoid infinite-scroll distraction design, and remain distinct from formal SA Insights generated from teaching patterns.  
  
  
  
**6. SA — Studio Assistant**  
  
**6.1 Omnipresence**  
  
SA is available throughout Tutor Studio and inherits the context of wherever the tutor is working: Course, Lesson, Student Journey, Arrangement, Theory, Actions, Appraisal, Catalog and other Studio objects.  
  
**6.2 Chat on Home**  
  
Home contains direct Chat input. Tutor can type or speak immediately. Lightweight access to history/navigation should also exist.  
  
**6.3 Conversation-first + object-first**  
  
Tutor Studio supports both:  
  
	●	conversation-first work, where intentions develop into objects/actions  
	●	object-first work, where tutor opens a Plan, Song, Theory item etc. and SA accompanies them  
  
Neither mode is permanently subordinate.  
  
**6.4 Tool truth**  
  
> **Tool truth beats conversational claim.**  
  
A mutation has happened only when the backend confirms it. SA text must never be treated as proof of a write.  
  
**6.5 SOU-specific data truth**  
  
	●	general musical knowledge allowed by default  
	●	SOU-specific factual claims require real retrieval/tool evidence  
	●	ambiguous requests should be clarified rather than guessed  
  
**6.6 Context access vs foregrounding**  
  
In Course Mode or Student Journey Mode, SA should have access to all authorised linked objects, including historical Plans, Appraisals, Registers, Actions, Songs/resources, playlists, Chats and relevant curriculum context. Relevance determines what is foregrounded.  
  
No persistent “SA Context” indicator is required.  
  
  
  
**7. Chats, Projects and SA Insights**  
  
**7.1 Chats are independent objects**  
  
Chats do not literally live inside folders. A Chat is its own object with relationships to relevant context, e.g.:  
  
Chat ↔ Course ↔ Lesson ↔ Student ↔ Song ↔ Arrangement ↔ Project  
  
The same Chat may appear naturally in multiple contextual histories without duplication.  
  
**7.2 Automatic relationships + human curation**  
  
Studio should automatically retain where a conversation originated and what it materially relates to.  
  
Tutors should also be able to:  
  
	●	rename Chat  
	●	add/change subtitle  
	●	relate it to Courses, Students, Songs, Projects etc.  
	●	ask SA to organise it conversationally  
  
“Relates to” is preferred to a single rigid folder location.  
  
**7.3 Projects**  
  
Projects may remain useful for intentional cross-cutting workspaces, but should not become mandatory folders for every Chat.  
  
**7.4 SA Insights**  
  
An SA Insight is:  
  
> **A Chat initiated by SA because it has noticed something sufficiently meaningful or useful to mention.**  
  
Rules:  
  
	●	created sparingly  
	●	appears as a Notification tagged Insight  
	●	opening it opens the underlying Chat  
	●	after that it behaves like a normal Chat  
	●	searchable/filterable by Insight tag  
	●	tutor can rename, organise or delete it like any other Chat  
	●	no separate Insights Home icon/count/section  
  
Exact Insight-generation thresholds should be tuned through use.  
  
  
  
**8. Notifications**  
  
Notifications form one simple chronological stream with lightweight labels such as Insight, Action, Admin, System, Schedule/change and content/report outcome.  
  
Unread Notifications drive the subtle Home count. Read Notifications remain in history. No manual archive flow is required.  
  
“Needs Attention” is not a separate object. Home may surface something that needs attention today, but the underlying item must live in Actions or Notifications. System-generated items should be clearly labelled System.  
  
  
  
**9. Actions**  
  
Actions are a Studio-wide capability, not specific to Appraisals.  
  
They can emerge from Appraisal, Course/Lesson planning, Student Journey work, Studio Chat, Arrangement/Theory work or other authorised contexts. General personal Tutor Actions are also allowed.  
  
If tutor explicitly asks SA to create an Action, SA can. If SA detects an implied commitment, it should offer rather than silently create.  
  
Actions retain relevant relationships to Tutor, Course, Student, Lesson, Song/Arrangement, resource, Chat/Project etc.  
  
One Action has one assignee. Original assigner controls assignment; assignee cannot delegate onward.  
  
Tutor-assigned completion flow:  
  
Assigner → Assignee works → Assignee marks complete → Assigner signs off → Action closes  
  
If not accepted, Action can be returned/reopened with optional note.  
  
SA may identify likely Admin Actions. Lead confirms classification. Once confirmed as Admin Action, responsibility transfers to Admin; tutor retains visibility of status/history only. Admin completes/signs off. Outcome remains linked to teaching context.  
  
Due dates are optional. Priority labels are not required. Optional reminders may use in-app/email/WhatsApp, with tutor-selected defaults and per-Action overrides.  
  
Default My Actions view remains one simple list with filters/search. Completed Actions disappear from everyday active view, perhaps leaving the last 3 as working memory; older completed Actions remain archived/searchable and available to SA in context.  
  
  
  
**10. Tutor roles and permissions**  
  
Role is contextual. One tutor may be Lead, Shared Lead, Assistant, Substitute, 1:1 tutor and content creator across different relationships.  
  
	●	**Lead:** broad edit authority across Course teaching context.  
	●	**Shared Leads:** equal read/edit authority; changes remain individually attributable.  
	●	**Assistant:** broad read access; edit only where role requires it; can update Register and add attributed Appraisal bullets, but cannot edit/complete the Appraisal or create lesson follow-up Actions independently.  
	●	**Substitute:** read access to relevant Course context plus temporary edit access while responsible for cover; edit access expires afterwards.  
  
When teaching relationship ends, current editing rights end but historical contributions remain attributable and appropriate read access may remain.  
  
Tutors can challenge SA and correct simple source-record errors where authorised. Corrections retain audit trail and trigger Admin alert. Ambiguous/consequential conflicts should be escalated.  
  
  
  
**11. Tutor Profile**  
  
Tutor Account/Profile may contain:  
  
	●	name, photo, bio, teaching philosophy  
	●	instruments, levels, adult/child suitability, minimum ages  
	●	online/in-person, areas/locations  
	●	styles/genres, techniques/specialisms, favourite songs to teach  
	●	musical interests/artists  
	●	availability/preferences  
	●	connected Spotify/YouTube accounts  
	●	contact/account settings  
	●	DBS status with Admin verification  
	●	private SA personalisation context  
  
Distinguish full private Tutor Account from any future public-facing Tutor Profile. Public-profile/social design is deferred.  
  
SA may gradually learn tutor tastes, teaching strengths, explanation preferences, habits and preferred working style. Tutor can inspect/correct/remove inferred information.  
  
  
  
**12. Schedule / Calendar / Booking**  
  
SOU Calendar contains SOU-related events only: Group Lessons, 1:1 Lessons, parties/shows, tutor meetings, training and other SOU commitments.  
  
SOU should push/sync SOU commitments to tutor-selected external calendar(s). SOU is not intended to replace personal calendars.  
  
Current Sesami/Shopify booking remains in place. Near-term direction is a native-looking SOU Schedule experience with Sesami underneath where practical. Native SOU booking/availability engine is later.  
  
  
  
**13. Course / Group model**  
  
Course is independent of its cohort and has identity/schedule/teaching context including Level, venue, dates, tutors, Course Plan, Lessons, playlist, student/cohort relationships, Appraisals, Actions and curriculum relationship.  
  
Course landing page should orient tutor with Course details, where they are in the series, master Register/Students, Lessons, Course Plan, Lesson Plans and Playlist/Songs.  
  
Course position is automatic from schedule/Lesson status, e.g. Week 5 of 10.  
  
Course journey shows all scheduled Lessons: past/current/future placeholders. Future Lessons may show automatically derived planning status such as Ready / Draft / Not started. Tapping a future Lesson opens Lesson Planning with Course context.  
  
Past Lessons provide fast access to Lesson Plan, Appraisal and Lesson Content/Songs.  
  
Course Plan is the intended roadmap. It is flexible, reflects current Course position, and retains version history when intentionally revised. The gap between intended Plan and actual Lesson/Appraisal history is valuable intelligence.  
  
Curriculum relates to Course rather than being identical to it. Tutors may locally diverge/experiment in their own Course Plans. Canonical Curriculum changes require authorised Curriculum/Admin approval.  
  
  
  
**14. Course playlists**  
  
Each Course should be linked to its playlist, currently Spotify. The playlist is the repertoire palette/totem pole.  
  
Playlist contains Songs, not Arrangements.  
  
Flow:  
  
Playlist Song → potential repertoire → Song selected for teaching → Arrangement selected → Lesson planned → Lesson taught/appraised  
  
Adding a Song to playlist does not choose an Arrangement.  
  
  
  
**15. Registers / attendance**  
  
Group Register remains very simple: Present / Absent.  
  
It can be taken/updated at any time by Lead or Assistant.  
  
Course Students/Register area acts as a master register, with deeper attendance history fed by individual Lesson Registers.  
  
Tutor may open Appraisal at any time, but Group Register must be complete before Appraisal is processed.  
  
Attendance is Tutor/Admin data; no student-facing attendance score/history is required.  
  
1:1 needs no Register, but each Lesson must reliably know Scheduled / Took place / Cancelled / No-show where relevant. Booking system may populate this, but tutor can manually correct it.  
  
Group Lesson also has simple Scheduled / Took place / Cancelled status.  
  
  
  
**16. Lesson planning**  
  
Two representations:  
  
**Full Lesson Plan**  
  
Canonical comprehensive planning record/workspace.  
  
**Short-form Lesson Plan / Tutor Prompt Sheet**  
  
Delivery-focused teaching view derived from Full Lesson Plan. Not independently maintained.  
  
Full Plan can contain Course/Student context, previous Lesson/Appraisal, Actions, Course/Journey direction, curriculum context, objectives, Songs, lesson flow, song-specific prompts, resources, notes and optional timings.  
  
Prompt Sheet should foreground:  
  
	●	Main objectives/teachings  
	●	Songs: Song · Artist · release date · key  
	●	lesson chunks such as Warm-up, Song 1, Song 2, Break, Finale  
	●	only relevant Song prompts: chords, tricky transitions, TAB, theory, strum/picking, teaching note, other notes  
  
Unused headings disappear. Timing is optional.  
  
Songs lead. Exercises/theory are sewn into songs wherever possible.  
  
Tutor can plan in natural teaching language and SA translates that into structured relationships and visual Plan. Direct manual editing remains available.  
  
Planning screen should remind tutor of previous Lesson/Appraisal, Course Plan/Journey direction and relevant open Actions.  
  
  
  
**17. Lesson Hub**  
  
Opening an upcoming Lesson should provide equal/direct access to:  
  
	●	Short-form Plan  
	●	Full Lesson Plan  
	●	Lesson Content  
	●	Course Plan for Groups / Journey Plan for 1:1  
  
Also provide subtle access to previous Lesson, previous Appraisal and Songs in current Lesson.  
  
Exact visual hierarchy remains UX-testable.  
  
  
  
**18. Teaching Mode**  
  
Teaching Mode is a manually entered, visually distinct live-teaching environment.  
  
Purpose:  
  
> **Help the tutor teach. Do not ask the tutor to document their teaching.**  
  
Core tools may include Short-form Plan, Lesson Content, Register, Playlist, Song player/reference link, Metronome, Beats, Recording and SA, plus customisable shortcuts.  
  
Short-form Plan behaves like a setlist with previous/next, free jump and tap-to-reveal deeper detail.  
  
Do not require Done/Skipped/Unfinished tracking or live note-taking.  
  
SA stays quiet/reactive by default, with voice prominent. Tutor can ask quick questions or dictate notes. No always-listening requirement.  
  
Teaching Mode may keep screen awake, preserve live working state, support continuity across devices and allow multiple simultaneous devices as different windows onto the same Lesson. Exact sync depth is technical.  
  
Teaching Mode should simplify Studio, not trap tutor. Keep one unobtrusive route to wider Studio.  
  
  
  
**19. Lesson recordings**  
  
Teaching Mode supports deliberate tutor-controlled recording of practice, exercises, tutor demonstrations, recitals/play-throughs and other useful moments.  
  
Group context: Course → Lesson → Recording  
1:1 context: Student → Lesson → Recording  
  
For now recordings remain stored unless deliberately deleted. Retention/storage policy deferred. Audio-aware SA Appraisal analysis is future development.  
  
  
  
**20. Appraisals**  
  
Appraisal is primarily an SA-aware bullet notepad, not a form and not mainly an interview.  
  
Tutor can type/dictate bullets before/during/after Lesson. SA can comment, ask follow-ups, refine wording and identify potential Actions.  
  
Assistant can add attributed bullets but cannot edit existing Appraisal or process/complete it.  
  
Complete Appraisal is a processing event, not a permanently locked state.  
  
When triggered:  
  
	1.	verify Group Register where applicable  
	2.	SA reads current bullets  
	3.	lightly refines/organises  
	4.	identifies Tutor/Admin Actions  
	5.	proposes classifications/assignees  
	6.	Lead confirms  
	7.	Appraisal is processed  
	8.	Lesson becomes Complete  
  
Appraisal can later be reopened/edited and processed again. System must reconcile existing Actions instead of duplicating them.  
  
Everyday view shows latest processed Appraisal; prior processed states remain in version/audit history.  
  
Latest Appraisal is primary context for next planning, while SA retains full Appraisal history and can recognise longitudinal/cross-course patterns.  
  
  
  
**21. Lesson / Course completion**  
  
No separate Complete Lesson button.  
  
Group: Lesson took place → Register complete → Appraisal processed → Lesson Complete  
  
1:1: confirmed Lesson status + Appraisal processing produces completed teaching record.  
  
Rescheduling must preserve prepared Lesson work/context; exact DB approach can be technical.  
  
Course completion flow:  
  
Final Lesson Complete → Course Appraisal due → Course Appraisal processed → Course Complete  
  
Completed Course remains fully available historically.  
  
  
  
**22. Course Appraisals and institutional memory**  
  
Course Appraisal is a separate end-of-Course bullet-based, type/dictate, SA-aware reflection.  
  
SA has full Course history available and Course Appraisals can inform future Courses at the same Level, across tutors/cohorts with provenance preserved.  
  
SA may create Insights from recurring patterns but must not autonomously rewrite canonical Curriculum/Templates.  
  
  
  
**23. 1:1 Student Journeys**  
  
Student replaces Course as organising context.  
  
1:1 Student landing page includes brief Student Profile, onboarding answers relevant to tutor, favourite artists/songs/genres, aspirations, Journey position such as Lesson 16, Journey Plan, Lessons, Lesson Plans, Appraisals, Playlist/Songs and Actions.  
  
1:1 Journey is open-ended, not Lesson 16 of 20.  
  
Student Profile is controlled by Student + Admin. Tutor sees only Tutor-visible fields and does not edit Profile directly. SA-inferred interests remain Journey context, not Profile facts.  
  
Journey Plan is created/maintained by Tutor + SA, attached to Student Journey, informed by Student aspirations/preferences and actual progress, and rolling rather than fixed-length.  
  
Student-facing Journey view later should be a simplified projection, not the full tutor planning object.  
  
Journey Plan = intended direction  
Lessons + Appraisals = actual journey  
  
Optional milestones may be created and marked achieved with Tutor+SA confirmation. Gamified rewards are deferred.  
  
  
  
**24. Tutor Studio without active teaching**  
  
Tutor account remains useful even with no current SOU Groups/1:1 students. Home naturally simplifies around SA, Studio, Music Inspiration, Catalog, Playlists, Projects/Chats, Actions and musical tools.  
  
Tutor may keep account indefinitely even if no longer actively teaching SOU. This supports future white-label potential, which is not MVP scope.  
  
  
  
**25. Studio — Creative Workspace**  
  
Studio MVP landing stays lean:  
  
**Create**  
  
	●	Song  
	●	Theory  
	●	Exercise  
  
**Work/library**  
  
	●	Recent Work  
	●	My Drafts  
	●	Published  
  
SA is omnipresent.  
  
Song is flagship creation environment. Theory is flexible digital-first multimedia resource. Exercise/Game remains recognised but detailed authoring toolkit is deferred.  
  
  
  
**26. Content ownership, Drafts and Publishing**  
  
Content produced in SOU belongs to SOU. Tutors are credited as creators/arrangers/contributors.  
  
Draft content is accessible to authoring tutor(s)+Admin only. Published content enters shared SOU teaching library visible to all tutors/admins. Student entitlement is separate from publication status.  
  
Normal lifecycle:  
  
Draft → SA Proofing → Human Publish  
  
SA proofing can check completeness, obvious errors, internal consistency, relationships, duplication/adaptation concerns and formatting/musical issues. SA does not silently publish.  
  
Content lifecycle also supports Archived, which removes content from active Catalog without breaking history.  
  
Creation work autosaves continuously by default.  
  
Destructive/irreversible actions require explicit confirmation; ordinary reversible actions should remain frictionless.  
  
  
  
**27. Collaborative authorship**  
  
A Draft may have multiple named authors/arrangers. Admin has access by default. Named authors have collaborative access.  
  
Creative credit is explicit; edit/version history records who changed what/when.  
  
All credited arrangers should sign off before publication under joint credit.  
  
True real-time Google-Docs-style co-editing is not required initially, but silent overwrites must be prevented. Advanced live collaboration can be phased later.  
  
  
  
**28. Song model**  
  
Core model:  
  
Song → one or more Arrangements  
  
Song is underlying musical identity. Arrangement is an SOU teaching/performance interpretation.  
  
Song/Arrangement default reference is original recorded key. Key is dynamically transposable.  
  
Course/Student context may remember working state such as key, strum/picking pattern and selected chord voicings without changing published Arrangement globally.  
  
Published Arrangement is stable. Meaningful later revision by credited arranger creates a linked new iteration. Original arranger may branch their own publication into Draft v2. Another tutor may not clone another tutor’s published Arrangement into a derivative publication.  
  
Admin may make objective maintenance corrections without creating a new creative iteration; audit trail retained. SA may recommend classification but human confirms.  
  
  
  
**29. Likes and content reports**  
  
Student Likes are intended across all published student-facing content.  
  
	●	account required  
	●	one Like per account/content item  
	●	Like attaches to specific content item/iteration  
	●	new iteration starts at zero  
	●	Popularity means student Likes only  
  
General written comments are deferred.  
  
Published content should support lightweight Report a problem. Report does not auto-suppress content. Admin can inspect/correct manually or via SA, sign off outcome, and reporter is notified.  
  
  
  
**30. Song Studio / Song Canvas**  
  
Flow:  
  
Create → Song → Find correct Song/recording → inspect existing published Arrangements → new Draft Arrangement  
  
A new Arrangement is substantially pre-populated rather than blank:  
  
	●	lyrics  
	●	chords over lyrics  
	●	detected sections  
	●	chord palette  
	●	strum/picking material  
	●	reference recording  
	●	useful metadata  
  
Main workspace: central lyrics/chords flow + side/tool palette for chord boxes, strum/picking and other shared tools. SA available throughout.  
  
System attempts to detect Intro/Verse/Chorus/Bridge etc.; tutor can rename, remove/hide, shorten, reorder, split/merge and repeat sections.  
  
Repeated sections can inherit from a core linked section by default, with custom variation per occurrence when needed.  
  
Harmony is an editable timeline. Tutor can replace/substitute chords, simplify/embellish, add/remove changes, alter harmonic rhythm, split one chord per bar into multiple changes, build chord solos and use different voicings by location.  
  
Chord palette is generated automatically from chords used and can offer one or several voicings depending on context. Chord Finder intelligence should be embedded.  
  
Performance Notes are free-position annotations linked to section/phrase/lyric/chord/bar and may contain formal strum/picking references or free text.  
  
Lyrics/chords remain structurally in Song flow; teaching overlays can be positioned more freely. Do not build a general DTP application for digital Canvas.  

**30a. Arrangement Builder — manual chord/lyric import (approved 31 Aug 2026, first build slice)**  

Alongside the flow above (which assumes inspecting/branching from an existing published Arrangement or a pre-populated Song source), a second, additional entry point exists for when no such source is available yet: a tutor pastes chord/lyric text directly.  

Workflow:  

  ●	tutor opens/creates an Arrangement for a Song  
  ●	a monospace plain-text paste surface accepts pasted chord/lyric text (e.g. copied from Ultimate Guitar)  
  ●	the system parses the paste into sections, lyrics, chords, and chord positions, and shows the result for review  
  ●	tutor can correct lyrics, chords, chord positions, and section structure before anything is saved  
  ●	reviewed content is saved as the Arrangement  

**Flagged relationship to the design above, not a contradiction:** this document's existing repeated-section model (line 655) has repeated sections inherit from a core linked section by default. The approved Phase 1 import architecture deliberately does not build this yet — repeated sections parsed from a paste are stored as independent sections with no reference/reuse relationship, as a reduced-fidelity MVP subset of the behaviour designed here, not a change to it. The full inheritance behaviour remains the target design for when it's built.  

Detailed parser behaviour and data shape are not specified here — see `MASTER_ARCHITECTURE.md` Section 5.5.  
  
  
  
**31. Transposition and playback**  
  
Arrangement transposition is a native dedicated control: semitone +/- and/or target key, updating chord symbols and chord boxes. Section-level modulation is supported in intended model.  
  
Course/Student remembers selected teaching key. Published Arrangement remains anchored to original recorded key.  
  
Song Canvas has immediate access to reference recording, potentially Spotify/YouTube/other integration.  
  
Track transport should support play/pause, scrub, jump, waveform/timeline, speed, pitch shift and loops.  
  
Preferred looping interaction: select lyric line/phrase/section and loop corresponding audio. Waveform/timeline is fallback/fine control.  
  
Speed: presets + slider; tempo changes without pitch.  
  
Reference audio pitch shifts by semitone. When Arrangement key changes, audio should automatically match by default. Match Arrangement resets temporary audio pitch changes.  
  
Future/intended capability: lyric/chord/section follows playback. If technically heavy, phase after core MVP.  
  
  
  
**32. Metronome and Beats**  
  
Metronome and Beats are shared tools across Song Studio, Teaching Mode, Home shortcuts and potentially Theory/Exercise.  
  
Initial Song analysis should attempt BPM and time signature detection; tutor can edit. Tap Tempo is shared across Beats/Metronome.  
  
Beats UX is simple while library is deep:  
  
Choose groove → choose sound → BPM → Play  
  
Library should be extensive/high quality across genres/feels/sounds/time signatures.  
  
Within Arrangement, Beats suggests a small set of plausible grooves from Song context. Tutor can browse full library.  
  
Grooves may offer related variations such as Simple / Standard / Busy / A / B / Fill. Live switching should occur cleanly at next musical boundary.  
  
Do not build section-by-section Beat sequencing/DAW behaviour for MVP. Arrangement simply remembers chosen Beat settings. Beats has independent simple volume control.  
  
  
  
**33. Digital Arrangement, Print View and PDF archive**  
  
Digital Arrangement is canonical and not constrained by A4.  
  
Print is a purpose-specific view of the same Arrangement. Main Song Print View is one A4 landscape page. SA/layout engine proposes compressed layout and tutor has final editorial/layout control.  
  
Beginner/Intermediate/Advanced musical versions are separate Arrangements. Digital vs Print are views of same Arrangement. Different key is dynamic state, not automatically new Arrangement.  
  
Structured changes such as transposition should flow into Print View. Substantive edits that invalidate curated layout should trigger review rather than silently destroy manual decisions.  
  
For current SOU MVP, explicitly created print files become real persistent searchable publication artefacts. Native content remains canonical, but approved PDFs are also stored.  
  
Publication/file types include Songsheet PDF, TAB Sheet PDF and Theory Sheet PDF.  
  
Only deliberately created PDFs exist in archive. Dynamic transposition does not imply PDFs for every key. Archive model may be revisited once TAB and other content become fully native/dynamic.  
  
  
  
**34. TAB**  
  
Native interactive TAB authoring is deferred.  
  
MVP TAB is primarily image-based. Tutor can upload/paste, crop, resize, position and perform Instant Alpha-style background removal/transparency.  
  
TAB may attach to Arrangement and also exist as searchable standalone TAB Sheet/publication.  
  
TAB image(s) can appear in main Song A4 landscape sheet if space permits, or be collaged into separate A4 landscape TAB Sheet PDF(s).  
  
Future native interactive TAB suite may change PDF/archive model.  
  
  
  
**35. Theory Studio**  
  
Flow:  
  
Create → Theory → title/topic → Theory Canvas → Draft → SA Proofing → Publish  
  
Theory is digital-first and should favour concise, digestible teaching nuggets rather than long textbook passages. SA should be aware of eventual print destination and may suggest splitting an over-dense subject into 2 pieces.  
  
Theory Canvas is modular and spatial/freeform, with broad shared musical/visual components such as text, chord diagrams, fretboards, scales, keyboard diagrams, notation/TAB, rhythm grids, strum/picking patterns, tables, arrows, images and highlighting.  
  
Print View is A4 landscape with simple 1- or 2-column layout. No predefined content templates required for MVP.  
  
Digital Theory may be interactive: playable chords/rhythms, root/key changes, interactive fretboard/keyboard, playable notation/TAB, short audio and video demonstrations. Print View extracts static essentials.  
  
  
  
**36. Exercise / Game**  
  
Exercise remains intentionally broad: drill, warm-up, game, technique exercise, rhythm exercise, ear training, group activity, improvisation.  
  
Conceptual distinction: Theory = understand this; Exercise = do this.  
  
Detailed Exercise creation environment is deferred. MVP may include category, a small set of example Exercises/Games, insertion into Lesson Plans, opening/running in Teaching Mode, and local non-destructive Lesson adaptation.  
  
No separate Exercise scoring/results workflow; anything worth recording belongs in Appraisal.  
  
  
  
**37. Catalog / discovery**  
  
Retain current filterable/list-based Catalog for now because it supports browsing, scanning, comparison and serendipitous discovery. Do not assume it is permanent UX.  
  
Future access may include Browse/Filter, Ask SA, contextual suggestions and richer visual discovery modes.  
  
Seasonal filtering should derive from multiple signals, not release date alone: release date, title, lyrical/theme content, cultural association, editorial/AI classification. No weighting scheme needs fixing now.  
  
  
  
**38. Student-facing content access — architectural only**  
  
Detailed Student UX is deferred.  
  
Already established principles:  
  
	●	publication status and student entitlement are separate  
	●	future top-tier subscription may access all published Arrangements/content  
	●	Course/1:1 context may surface assigned material  
	●	Student Likes can apply across published content  
	●	Student Profile is Student/Admin-controlled  
	●	Student Journey view should be simplified relative to Tutor Journey Plan  
  
Do not expand into detailed Student UI spec until students have been consulted.  
  
  
  
**39. Remote / online lessons — future direction**  
  
For MVP, Tutor Studio supports lesson context around existing call platform and tutor/student independently open relevant SOU content.  
  
Future direction may include native Studio Room, music-friendly low-latency audio/video, tutor-controlled student content view, phrase jump/rewind/loop and shared playback state. Valuable but potentially technically substantial; do not block Tutor MVP.  
  
  
  
**40. Admin product boundary**  
  
Immediate Admin MVP: functional controls required to operate Tutor Studio, not polished dedicated Admin UX.  
  
Capabilities as needed include tutor creation/invitation, Course/Student/enrolment relationships, permissions, Draft access, published content governance, reports/corrections, Admin Actions and system/account operations.  
  
  
  
**41. Tutor onboarding**  
  
Immediate requirement:  
  
Admin creates/invites Tutor → Tutor establishes account → basic profile/setup → Tutor Studio  
  
No elaborate tour/questionnaire/preferences onboarding required now.  
  
  
  
**42. Offline, security, search, settings and export**  
  
	●	**Offline:** teaching-critical prepared material should ultimately survive temporary connectivity loss; implementation deferred.  
	●	**Security/privacy:** authentication and defined permissions are immediate; deeper GDPR/retention/deletion-export/security hardening later before wider deployment.  
	●	**Global search:** deferred; contextual search where required is sufficient.  
	●	**Settings:** existing placeholder sufficient until specific needs arise.  
	●	**General sharing/export:** only PDF creation/archive required now; public links, email-from-app, bulk export and broad sharing deferred.  
	●	**Version recovery:** basic Undo + explicitly required versioning; no universal restore system yet.  
  
  
  
**43. Content / Course history**  
  
Archived content remains historically available where referenced; do not break old Lessons by deleting content.  
  
Lesson Plans normally reference current published content objects, so updates flow through unless deliberately fixed artefact is involved. Persistent published PDFs remain historical snapshots.  
  
Completed Course retains Course Plan, Lessons, Lesson Plans, Appraisals, Course Appraisal, Registers, Songs/content, Actions, linked Chats and related history.  
  
  
  
**44. Future white-label / commercial architecture**  
  
Tutor Studio may eventually support independent tutors, other schools and white-labelled implementations. Do not build this now. Where cheap, avoid hard-coding assumptions that make future separation impossible. Current SOU internal use remains proving ground.  
  
  
  
**45. Immediate MVP vs deferred**  
  
Immediate/internal product emphasis should prioritise functionality that lets SOU create native digital Songs/Arrangements, produce/preserve printable resources, plan Courses/Lessons, use Prompt Sheets, teach through Teaching Mode, capture Registers, appraise Lessons/Courses, manage Actions, accumulate teaching memory, operate multiple tutor roles safely and use SA contextually throughout.  
  
Exact engineering order is NOT defined here. It must come from current-build gap analysis.  
  
Explicitly deferred/later:  
  
	●	detailed Student UX pending student consultation  
	●	polished Admin UX  
	●	full mobile Creative Studio optimisation  
	●	detailed Exercise/Game authoring suite  
	●	native interactive TAB editor  
	●	audio-analysis Appraisal  
	●	universal global search  
	●	broad social/messaging system  
	●	student comments/reviews  
	●	native SOU booking replacement  
	●	native remote meeting platform  
	●	tutor-controlled shared student screen/content  
	●	complete offline architecture  
	●	deeper GDPR/compliance work  
	●	sophisticated real-time collaborative editing  
	●	gamification furniture  
	●	full white-label implementation  
	●	general export/sharing suite  
  
  
  
**46. Implementation-phasing principle**  
  
The design destination and implementation MVP are not the same thing.  
  
Features should be evaluated by architectural importance, operational value and development complexity/cost. Cheap future-proofing is welcome; complex features can be phased without deleting them from product design.  
  
Upcoming gap analysis must consider two lenses:  
  
**Commercial/strategic MVP**  
  
Coherent path toward broader Tutor Studio product.  
  
**September Utility**  
  
Shortest technically coherent path from current app to something SOU can use in real lesson/content production immediately.  
  
September Utility may intentionally defer substantial parts of the broader Tutor Studio destination.  
  
  
  
**47. Discovery status**  
  
The August 2026 Tutor/Admin/internal-SOU design-mapping phase is complete. Student UX is deliberately deferred.  
  
Next work:  
  
	1.	reconcile this specification against existing canonical docs  
	2.	identify contradictions/superseded assumptions  
	3.	decide canonical document structure  
	4.	update canonical documentation  
	5.	compare resolved design with current code/build  
	6.	produce current-build gap analysis  
	7.	identify September Utility path  
	8.	only then derive implementation sequence and Claude/Copilot prompts  
  
  
  
**48. Governing principles summary**  
  
	●	Tutor Studio is a distinct tutor-facing experience on the shared SOU platform.  
	●	SA is omnipresent, contextual and grounded in real system/tool truth.  
	●	Conversation-first and object-first workflows coexist.  
	●	Home should invite, not nag.  
	●	Songs lead teaching; theory/exercises are woven into musical context.  
	●	Plans preserve intention; Appraisals preserve reality.  
	●	Native digital content is canonical.  
	●	For current MVP, deliberately created PDFs are persistent searchable publication artefacts.  
	●	Drafts are private to author(s)+Admin; Published content contributes to shared SOU library.  
	●	Tutor creativity is credited; SOU owns platform-created content.  
	●	Tutors may experiment locally; shared Curriculum changes require authorised human approval.  
	●	SA may observe and propose; humans confirm consequential changes.  
	●	Teaching Mode reduces cognitive load and does not create live admin.  
	●	Structured teaching intelligence should emerge as a by-product of normal work, not research bureaucracy.  
	●	Design destination should not force every feature into v1.  
