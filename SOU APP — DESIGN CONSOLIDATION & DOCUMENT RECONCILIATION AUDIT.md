#   
**SOU APP — DESIGN CONSOLIDATION & DOCUMENT RECONCILIATION AUDIT**  
  
**Date:** 25 August 2026  
**Status:** Draft consolidation audit — no implementation authorised  
  
  
  
**1. Purpose**  
  
This audit compares the resolved August 2026 design/UX decisions against the current canonical/product documents:  
  
	●	MASTER_ARCHITECTURE.md  
	●	PRODUCT_ROADMAP.md  
	●	PROJECT_FEATURE_MAP.md  
  
It identifies what remains valid, what changed, where current documents now conflict, what should become canonical, and what should not be duplicated.  
  
  
  
**2. Current governance — keep**  
  
**MASTER_ARCHITECTURE.md**  
  
Current technical truth: implemented architecture, deployment, actual schema, live systems, session history, current risks and immediate engineering work.  
  
**PRODUCT_ROADMAP.md**  
  
Forward direction: sequencing, phases, priorities, strategic opportunities and deferred future work.  
  
This division remains sound and should be preserved.  
  
  
  
**3. New canonical document recommendation**  
  
The August design exercise is too detailed to live cleanly inside either Architecture or Roadmap.  
  
Recommendation: create **PRODUCT_UX_SPEC.md** as the canonical durable definition of resolved product behaviour, UX, objects, relationships, permissions, workflows and MVP/deferred design decisions.  
  
Target governance:  
  
	●	MASTER_ARCHITECTURE.md = what exists technically now  
	●	PRODUCT_UX_SPEC.md = what the product has been designed to do  
	●	PRODUCT_ROADMAP.md = what gets built next and in what order  
  
PRODUCT_UX_SPEC.md should be referenced from Architecture and Roadmap.  
  
  
  
**4. PROJECT_FEATURE_MAP.md materially conflicts**  
  
The June 2026 Product Vision was useful, but the August Q&A superseded significant assumptions.  
  
Recommendation: do not keep it as a parallel active source of truth.  
  
Preferred options:  
  
	1.	archive it and make PRODUCT_UX_SPEC.md its successor; or  
	2.	retain filename only as a short pointer/index if code/docs depend on it.  
  
Avoid four overlapping living product documents.  
  
  
  
**5. Major supersessions / conflicts**  
  
**Tutor UI vs Admin**  
  
Earlier: one shell, Tutor as role-gated Admin.  
Now: one shared platform/data/services, but Tutor Studio is a distinct tutor-facing interface. Admin can remain utilitarian.  
  
**Home / Conversation default**  
  
Earlier: Conversation default landing page.  
Now: Home is broader and inviting: Chat/SA, Music Inspiration, Studio, subtle Schedule, Actions/Notifications, conditional Groups/Students, personal shortcuts.  
  
**Persistent VS Code-style chat panel**  
  
Earlier: docked persistent panel described as target architecture.  
Now: SA must be omnipresent/contextual, but exact panel/docking layout is deliberately not fixed. Treat VS Code panel as one UX option, not architectural certainty.  
  
**Published content visibility**  
  
Earlier: tutors largely limited to own workspace/content.  
Now: Drafts = author(s)+Admin; Published = shared SOU library visible to all tutors/admins.  
  
**Songsheet model**  
  
Earlier: core Songsheet object, roughly one per song per key.  
Now: Song → Arrangements → Digital/Print views → explicitly created PDF publications; key is dynamic.  
  
**Arrangement editing / versions**  
  
Now: meaningful creative revision creates new linked iteration; original arranger can branch own publication; other tutors cannot clone someone else’s publication into derivative credit. Admin can make maintenance corrections without new creative iteration.  
  
**PDF meaning**  
  
Roadmap principle “native digital canonical, PDF output” remains correct, but current MVP also needs persistent searchable approved PDFs as real publication artefacts. PDF is not always ephemeral/on-demand.  
  
**TAB**  
  
MVP remains image/PDF based with crop/resize/position/Instant Alpha-style background removal; standalone searchable TAB Sheet publication plus Arrangement relationship. Native TAB editor is later.  
  
**Theory**  
  
Older three-type Theory taxonomy may remain useful editorially, but new design is broader: digital-first freeform multimedia canvas, concise nugget philosophy, 1/2-column A4 Print View and intrinsic relationships. Do not let old taxonomy define architecture unless reconfirmed.  
  
**Exercise/Game**  
  
Detailed authoring environment explicitly deferred pending real usage.  
  
**Lesson planning**  
  
Older Trello-style metaphor is obsolete. New model is substantially resolved: Course Plan, Full Lesson Plan, SA-derived Short-form Plan, Lesson Hub, Teaching Mode, Appraisal and intended-vs-actual history.  
  
**Appraisal sequencing**  
  
Roadmap treats lightweight Appraisal/outcome capture as later. New design couples Appraisal tightly to a usable Lesson Planning/Teaching loop, so sequencing must be revisited after code gap analysis.  
  
**Course / Student Journey**  
  
Earlier “not yet designed.” Tutor-side Course and 1:1 Journey models are now substantially designed. Detailed Student-facing UX remains deferred.  
  
**Student Platform**  
  
Existing Student ideas remain directional only. Detailed Student UX should not be considered settled until student consultation.  
  
**Booking**  
  
Near-term: keep Sesami, build native SOU Schedule surface and integration abstraction; native booking engine later.  
  
**Social/community**  
  
Forum/general messaging is not near-term Tutor MVP. Revisit later with Student/social architecture.  
  
  
  
**6. Roadmap sequencing requires reconciliation, not blind replacement**  
  
Current Roadmap spine:  
  
Studio stabilisation → legacy PDF extraction → Digital Songsheet Builder → Curriculum/Lesson integration → Pedagogy → Appraisal → Progression → Intelligence  
  
August design changes dependency picture:  
  
	1.	Lesson planning, Appraisal and Teaching Mode now form one coherent operating loop.  
	2.	Multi-account/role/permission groundwork becomes more important for Tutor Studio.  
	3.	Song/Arrangement model is richer than old minimal Songsheet Builder description.  
	4.	Persistent PDF publication/archive remains operationally important.  
	5.	Student UI is explicitly not immediate MVP.  
	6.	September Utility may differ from commercial MVP sequence.  
  
Therefore do not rewrite Roadmap until resolved-design vs current-build gap analysis is complete.  
  
  
  
**7. Current-build gap analysis — required next**  
  
Inspect at minimum:  
  
**Identity/auth/permissions**  
  
	●	current tutors table  
	●	hardcoded tutor_id=1  
	●	login/session model  
	●	roles  
	●	Course/Student permission needs  
  
**Chat/SA**  
  
	●	current ConversationWorkspace  
	●	contextual relationships  
	●	tool system/audit trail  
	●	context retrieval architecture  
	●	Chat metadata/search  
  
**Content**  
  
	●	current songs table  
	●	mapping to new Arrangement entity  
	●	current PDF/material fields  
	●	R2 storage  
	●	chord library  
	●	theory/TAB assets  
  
**Course/Lesson/Student**  
  
	●	existing tables, if any  
	●	Course  
	●	cohort/enrolment  
	●	Lesson  
	●	Course Plan  
	●	Lesson Plan  
	●	Appraisal  
	●	Register  
	●	Action  
	●	Student Journey/Journey Plan  
	●	playlist relationship  
  
**Studio UI**  
  
	●	current Admin shell  
	●	feasibility of distinct Tutor Studio shell  
	●	responsive foundation  
	●	SA omnipresence  
  
**Print/PDF**  
  
	●	current PDF serving/storage  
	●	generation capabilities  
	●	publication records  
	●	key-specific archived generated PDFs  
  
**Integrations**  
  
	●	Sesami  
	●	Spotify playlists  
	●	external calendar push  
	●	WhatsApp/email notifications  
	●	reference audio feasibility  
  
Only after this comparison should development sequencing be set.  
  
  
  
**8. September Utility lens**  
  
As soon as consolidation + current-build gap analysis exist, ask:  
  
> **What is the shortest technically coherent path from the current build to something Matthew can use for real September lesson/content production?**  
  
Possible thin path hypothesis:  
  
minimum Tutor auth/context → Song/Arrangement creation → A4 print/PDF archive → Course/Lesson Plan → Short-form Prompt Sheet  
  
This is only a hypothesis and must be validated by the gap analysis.  
  
Do not assume Student accounts, social features, native booking, full notification infrastructure or white-label architecture are prerequisites for internal utility.  
  
  
  
**9. Proposed document governance after consolidation**  
  
**MASTER_ARCHITECTURE.md**  
  
Owns implemented technical state, actual schema/APIs, deployment, repo rules, AI/tool-integrity implementation, immediate engineering items and session log.  
  
**PRODUCT_UX_SPEC.md**  
  
Owns resolved product behaviour, Tutor Studio UX principles, product-level objects/semantics, roles/permissions, workflows, content lifecycle and MVP/deferred decisions.  
  
**PRODUCT_ROADMAP.md**  
  
Owns sequencing, phases, dependency order, September Utility lane if adopted, strategic future and anti-scope-creep rules.  
  
**PROJECT_FEATURE_MAP.md**  
  
Archive or reduce to pointer/index. Do not maintain as fourth overlapping source of product truth.  
  
  
  
**10. Recommended update order**  
  
	1.	approve consolidated PRODUCT_UX_SPEC.md  
	2.	archive/supersede PROJECT_FEATURE_MAP.md  
	3.	update MASTER_ARCHITECTURE.md governance references only, plus current-state corrections discovered during code audit  
	4.	perform current-build gap analysis  
	5.	update PRODUCT_ROADMAP.md sequencing from evidence  
	6.	add Architecture session-log entry documenting consolidation  
	7.	only then produce implementation prompts  
  
This order minimises drift.  
  
  
  
**11. Consolidation status**  
  
	●	Design extraction: **complete draft**  
	●	Conflict/supersession audit: **complete draft**  
	●	Canonical document recommendation: **made, awaiting approval**  
	●	Current-build code gap analysis: **not yet performed**  
	●	Roadmap rewrite: **not yet performed**  
	●	Implementation prompts: **not authorised / not begun**  
