# Dedicated Music Space Specification

**Spec ID:** `FEAT-MUSIC-SPACE-001`  
**Version:** `2.0`  
**Status:** `Ready for Development`  
**Created:** `2025-09-26`  
**Last Updated:** `2025-09-27`  
**Author(s):** `Bill Wang`

## Overview

### Problem Statement
Users who enjoy the hero page's ambient music have no way to listen to the full 'Endless Drifting' album continuously in a dedicated, low-distraction environment. The experience lacks a deeper, meditative visual component to complement the audio, limiting its potential as a "chill space" for long-term focus or relaxation.

### Goals

**Primary Goal:** Provide a persistent, long-play audio experience where users can listen to the entire music albums on a continuous loop.

**Secondary Goal:** Create a "digital zen garden" or "chill" space on the site for users who want to relax or focus on other tasks with ambient background music.

**Tertiary Goal:** Enhance the starfield background with dynamic constellation animations, paired with uplifting messages that appear at regular 60-second intervals.

### Non-Goals

- **No complex playlist management:** This is not a full-featured music player. Users will not be able to create custom playlists, reorder tracks, or upload their own music.
- **No social features:** There will be no chat or user profiles.
- **No user interaction with animations:** The constellation animations are a passive visual experience.

## User Stories

### Primary User Stories

```
As a visitor,
I want to navigate to a dedicated music space,
So that I can listen to the full album without interruption for an extended period.
```

```
As a remote worker or student,
I want to open the music space in a browser tab,
So that I have calming background music and a gentle, visually interesting animation to help me focus or relax.
```

### Edge Cases & Error Scenarios

```
As a user,
When my internet connection drops temporarily,
The player should attempt to buffer and resume playback gracefully without losing my place,
So that my listening experience is not abruptly reset.
```

```
As a user,
When I navigate away from the music space,
The original website's background music should resume control,
So that the audio experience remains consistent across the site.
```

## Requirements

### Functional Requirements

| ID | Requirement | Priority | Acceptance Criteria |
|----|-------------|----------|---------------------|
| FR-01 | Continuous Playbook | High | The full "Endless Drifting" album plays from the first to the last track and then loops back to the beginning automatically. |
| FR-02 | Traditional Music Controls | High | User has access to album selector, track list, play/pause, next/previous track, volume adjustment, and progress bar with scrubbing capability. |
| FR-03 | Dedicated URL | High | The music space is accessible via a unique and easily shareable URL, e.g., /music-space. |
| FR-04 | Minimalist UI | High | The UI consists of the starfield background with expandable/collapsible music controls overlay. |
| FR-05 | Track Information Display | Medium | The currently playing track and album information is displayed within the audio control component. |
| FR-06 | Constellation Animation Cycle | High | The background animation follows a distinct, repeating 60-second cycle: 1. Starfield appears 2. Yellow stars automatically connect one-by-one over 2 seconds 3. Constellation name and uplifting lore message appear 4. Message fades out 5. Constellation floats with label (variable duration buffer) 6. New song triggers restart of cycle |
| FR-07 | Dynamic Content Loading | High | Constellation data (star positions, connections, lore) is randomly selected from constellations.json (1000+ entries). |
| FR-08 | Audio Management | High | Override existing website background music when entering music space. Restore original audio control when navigating away. |
| FR-09 | User Interaction Gate | High | Display entry message "You have entered music space, click to enter" to comply with autoplay policies and require user interaction before starting music. |
| FR-10 | Multiple Albums Support | Medium | Support selection between multiple albums from music index.json structure. |

### Non-Functional Requirements

| ID | Requirement | Target | Measurement |
|----|-------------|--------|-------------|
| NFR-01 | Performance | Initial load < 2s. Animations must maintain smooth framerate (60fps). | Lighthouse, Browser Performance Profiler |
| NFR-02 | Accessibility | WCAG 2.1 AA | Audio controls fully keyboard-navigable and screen-reader compatible. Lore messages have sufficient color contrast. |
| NFR-03 | Browser Support | Latest 2 versions of Chrome, Firefox, Safari, Edge | Manual and automated cross-browser testing. |
| NFR-04 | Mobile Responsiveness | Touch-friendly controls on mobile devices | Controls adapt to mobile with bottom drawer UI |

## Design Specification

### User Interface

The UI will be a full-screen view of the StarFieldHero animated background, enhanced with automatic constellation formation cycles. Music controls will be presented as responsive overlays.

### Visual Design

**Background:** Reuse the StarFieldHero component's starfield animation system, including:
- Purple particle field with rotation
- Yellow star generation and connection mechanics
- Glassmorphic lore panel system
- Word-by-word text animation

**Controls:** 
- **Desktop:** Expandable/collapsible overlay with traditional music player interface
- **Mobile:** Bottom drawer that slides up for full controls, collapses to minimal "now playing" bar
- **Auto-collapse:** After inactivity timeout on mobile
- **Swipe gestures:** Up to expand, down to collapse on mobile

**Animation Sequence (60-second cycles):**
1. **Starfield Base:** Purple particles with rotation (adapted from StarFieldHero)
2. **Star Connection:** Yellow stars appear and auto-connect one-by-one over 2 seconds
3. **Lore Panel:** Constellation name and uplifting lore message appear using glassmorphic panel
4. **Message Fade:** Lore panel auto-dismisses after reading time
5. **Constellation Float:** Constellation remains visible with label for variable buffer duration
6. **Cycle Restart:** New song triggers immediate restart of sequence

### Technical Architecture

```
ConstellationMusicSpace/
├── StarFieldBackground (adapted from StarFieldHero)
├── ConstellationAnimator (5-phase cycle manager)
├── MusicPlayer (traditional controls)
│   ├── AlbumSelector (dropdown/grid)
│   ├── TrackList (for selected album)
│   ├── PlayerControls (play/pause/next/prev/volume/progress)
│   └── ResponsiveDrawer (mobile/desktop layouts)
├── LorePanel (adapted from StarFieldHero)
└── AudioManager (override site music, restore on exit)
```

### Music Integration

**Timing Logic:**
- Each constellation cycle = 60 seconds fixed duration
- Song duration ÷ 60 = number of full constellation cycles per song
- Remainder time extends the final "constellation floating with label" phase
- New song start = immediate restart of constellation cycle
- Example: 3:13 song = 3 full cycles (3 minutes) + 13 second extended final phase

**Data Sources:**
- Music metadata: `/public/music/index.json` and `/public/music/{album-id}/metadata.json`
- Audio files: `/public/music/{album-id}/*.m4a`
- Constellation data: `constellations.json` (random selection from 1000+ entries)

### Interaction Design

**Entry Flow:**
1. User navigates to `/music-space`
2. Starfield loads immediately
3. Entry gate displays: "You have entered music space, click to enter"
4. User click triggers music start and constellation cycle
5. Defaults to first track of first album at 50% volume

**Audio Management:**
- Override any existing site background music on entry
- Restore original site audio control on navigation away
- Graceful buffering and resume for network interruptions

### Responsive Behavior

**Desktop:**
- Expandable/collapsible music control overlay
- Full traditional music player interface
- Hover states and smooth transitions

**Mobile:**
- Bottom drawer UI pattern
- Swipe gestures (up/down) for expand/collapse
- Auto-collapse after inactivity
- Touch-friendly control sizes
- Minimal "now playing" bar when collapsed

## Success Criteria

### Definition of Done

- [x] **Phase 1: Core Component Setup & Starfield Background** ✅
  - [x] Created ConstellationMusicSpace component structure
  - [x] Adapted StarFieldBackground from StarFieldHero  
  - [x] Set up basic routing and page structure at /music-space
  - [x] Implemented entry gate with user interaction requirement
- [ ] **Phase 2: Music Player Infrastructure** 🚧
  - [ ] Create AudioManager for site music override/restore
  - [ ] Build traditional music player controls with album/track selection
  - [ ] Implement responsive mobile drawer UI
- [ ] **Phase 3: Constellation Animation System** 🔄
  - [ ] Build ConstellationAnimator with 5-phase cycle
  - [ ] Implement automatic star connection animation
  - [ ] Adapt LorePanel from StarFieldHero
  - [ ] Add random constellation selection from constellations.json
- [ ] **Phase 4: Music-Constellation Synchronization** 🔄
  - [ ] Implement 60-second constellation timing logic
  - [ ] Add variable buffer duration for song sync
  - [ ] Handle new song triggers for cycle restart
- [ ] **Phase 5: Polish & Optimization** 🔄
  - [ ] Accessibility improvements
  - [ ] Performance optimization
  - [ ] Cross-browser testing
  - [ ] Final UI/UX polish

## Technical Decisions

- **Audio Format:** M4A files in `/public/music/{album-id}/` directories
- **Constellation Selection:** Truly random from constellations.json (no filtering)
- **Initial Volume:** Default to 50%
- **Mobile Drawer:** Auto-collapse after inactivity with minimal "now playing" display
- **Timing Cycle:** Fixed 60-second constellation cycles with variable buffer extension
- **Component Architecture:** New component (not extending StarFieldHero) with shared systems