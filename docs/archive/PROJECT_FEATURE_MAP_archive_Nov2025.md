# SOU Platform/App Feature Map

This document summarizes the overall architecture and feature set for the SOU app, based on the provided chart. Use this as a living reference for planning, development, and future feature refinement.

---

## Architecture Overview

The SOU Platform/App is structured around three primary user interfaces (Tutor UI, Admin UI, End-user UI) plus a community/support component (Genius Bar & Forum). All components integrate with a central Song Database and Music Theory resources.

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
