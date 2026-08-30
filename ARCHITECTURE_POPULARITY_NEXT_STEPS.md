# Popularity & Progressive Discovery: Next Implementation Steps (MVP Roadmap)

## 1. Session Datastore (In Progress)
- File: `materials-server/searchSessionStore.js`
- Purpose: Hold working set for a user's multi-filter search; avoid refetch for each refinement.
- Actions Remaining:
  - Integrate into `searchService.advancedSearch(criteria)` to create/retrieve session ID.
  - Return `sessionId` in response meta; frontend stores it.
  - Add endpoint `POST /api/search/session/:id/enrich` to trigger specific enrichment modules.

## 2. Enrichment Modules (Planned)
- Key modules: genres/tags, audio features, chart history, popularity refresh.
- Pattern: `enrichXYZ(session, subsetKeys)` returning status map.
- Concurrency: Limit to 3 parallel external API groups to respect rate limits.

## 3. Popularity Aggregator (Stub Complete)
- File: `materials-server/popularityAggregator.js`
- TODO: Replace placeholder metric fabrication with real API integrations.
- Endpoint to add: `POST /api/popularity/refresh/:canonicalKey` -> forces metrics refresh.

## 4. Admission Policy
- File: `materials-server/admissionPolicy.js`
- Integrate with endpoint `GET /api/popularity/recommendations` (songs above threshold not yet in CSV; show promote list).
- Future: `POST /api/popularity/promote/:canonicalKey` -> writes to CSV with generated ID (needs canonical title/artist mapping).

## 5. Threshold & Promotion Flow (UI)
- Add Promote button next to catalog entries meeting threshold.
- Confirmation dialog shows component breakdown + rationale.
- Post-promotion: refresh admin song table and remove from recommendations list.

## 6. Risk & Performance
- Latency sources: Spotify search, Last.fm track.getInfo, chart aggregation.
- Mitigations: session caching, batch fetch, score memoization, opportunistic refresh outside user path.
- SLA Target: <1200ms initial response (artist only), incremental enrichment updates streaming via WebSocket or polling.

## 7. AI Assist Integration
- Use existing `aiAssistant` module to map broad natural queries ("classic upbeat 80s pop") to structured filter seed (year range + minPopularity + genre).
- Implement classification + expansion: prompt LLM to output JSON { years:[...], genres:[...], minScore, archetype }.
- Guardrails: Validate each field against allowed enumerations before applying.

## 8. Data Quality Loop
- After promotion, schedule background popularity refresh (t+1 day) to catch rapid velocity decay.
- Build discrepancy audit: peakPosition mismatches across sources -> manual review queue.

## 9. Logging & Observability
- Add structured logs per enrichment stage: { sessionId, stage, durationMs, externalCalls }.
- Aggregate daily metrics for rate-limit planning.

## 10. Future Scalability
- Replace in-memory Map with Redis when sessions >500 or multi-instance deployment.
- Background worker (BullMQ) for popularity refresh tasks.

## Implementation Order (Short)
1. Wire session ID returns & basic createSession in advancedSearch.
2. Recommendations endpoint + promotion (write to CSV) stub.
3. Real Spotify + Last.fm integrations in aggregator.
4. Frontend promotion UI.
5. AI assist structured query expansion.
6. Background refresh scheduler.

## Open Questions
- Duplicate canonicalization refinement (need robust title variation handling?).
- Multi-artist collaborations identity merging policy?
- Handling re-releases (2023 remaster vs original) in scoring age computation.
