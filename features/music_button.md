# Music Button Specification

**Spec ID:** `music-button-v2`  
**Version:** `2.0`  
**Status:** `Implemented`  
**Created:** `2025-09-28`  
**Last Updated:** `2025-09-28`  
**Author(s):** `Gameology Development Team`

## Overview

### Problem Statement
Users need a consistent, intuitive way to control background music across the Gameology web application. The music system must respect browser autoplay policies, provide clear visual feedback, and maintain user preferences across sessions while being accessible and performant.

### Goals
1. **Primary**: Provide a 3-state music button system that respects user preferences and browser policies
2. **Secondary**: Maintain consistent music playback state across page navigation
3. **Tertiary**: Ensure accessibility compliance and smooth user experience

### Non-Goals
- Music visualization or equalizer features
- Multiple track selection within this component
- Volume control (handled separately)
- Playlist management

## User Stories

### Primary User Stories
```
As a first-time visitor,
I want to see a clear music note icon,
So that I understand I can enable background music if desired.

As a returning user with "play on start" preference,
I want music to automatically start playing when I visit,
So that I don't have to manually enable it each time.

As a user controlling music,
I want to easily toggle between play and pause states,
So that I can control my listening experience without confusion.
```

### Edge Cases & Error Scenarios
```
As a user on a browser with strict autoplay policies,
When automatic playback is blocked,
I should see the music note state instead of an error,
So that I can manually initiate playback.

As a user with JavaScript disabled,
When I interact with the music button,
I should see graceful degradation without crashes,
So that the rest of the application remains functional.
```

## Requirements

### Functional Requirements
| ID | Requirement | Priority | Acceptance Criteria |
|----|-------------|----------|-------------------|
| FR-01 | 3-state button system (music note → play/pause toggle) | High | Button displays correct icon for each state |
| FR-02 | User preference persistence | High | Settings saved to localStorage and respected on revisit |
| FR-03 | Browser autoplay compliance | High | No console errors from blocked autoplay attempts |
| FR-04 | State synchronization across components | Medium | All music controls reflect current playback state |
| FR-05 | Keyboard accessibility | Medium | Button operable via keyboard navigation |

### Non-Functional Requirements
| ID | Requirement | Target | Measurement |
|----|-------------|--------|-------------|
| NFR-01 | Performance | <100ms state transition | User interaction timing |
| NFR-02 | Accessibility | WCAG 2.1 AA compliance | Screen reader compatibility |
| NFR-03 | Browser Support | Modern browsers with ES6+ | Cross-browser testing |

## Design Specification

### User Interface

#### Visual Design
- **Initial State**: Music note icon (♪) - indicates music is available
- **Playing State**: Pause icon (⏸) - indicates music is playing and can be paused
- **Paused State**: Play icon (▶) - indicates music is paused and can be resumed
- **Button Style**: Consistent with header icon styling, hover states included
- **Responsive Behavior**: Maintains size and position across screen sizes

#### Interaction Design
- **First Visit**: Music note → Click → Play (with user interaction context)
- **Subsequent Clicks**: Play ↔ Pause toggle only
- **State Persistence**: Remembers user preference for future visits
- **Error Handling**: Graceful fallback for autoplay failures

### Technical Architecture

#### Frontend Components
```
MusicToggleButton
├── useMusic (Context Hook)
├── Icon Components (Music, Play, Pause)
└── localStorageManager
```

#### Data Models
```typescript
interface MusicPreference {
  hasInteracted: boolean;
  shouldAutoPlay: boolean;
  lastVisitState: 'initial' | 'playing' | 'paused';
}

interface MusicButtonState {
  currentState: 'initial' | 'playing' | 'paused';
  isLoading: boolean;
  hasUserInteracted: boolean;
}
```

#### State Management
- **MusicContext**: Global music state management
- **localStorage**: User preference persistence
- **MusicService**: Singleton for audio control

### Dependencies
- React 18+ - Component framework
- Next.js 15+ - Application framework
- Lucide React - Icon library
- Web Audio API - Browser audio control

## Success Criteria

### Definition of Done
- [x] All functional requirements implemented
- [x] All acceptance criteria met
- [x] Performance requirements satisfied
- [x] Accessibility requirements met
- [x] Cross-browser compatibility verified
- [x] Unit tests written and passing
- [x] Integration tests written and passing
- [x] Documentation updated
- [x] Code reviewed and approved

### Key Performance Indicators
| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| State Transition Time | <100ms | Performance monitoring |
| User Preference Retention | 100% | localStorage persistence testing |
| Autoplay Success Rate | 90%+ | Error tracking and user interaction detection |

## Testing Strategy

### Unit Tests
- State transition logic
- User preference persistence
- Error handling scenarios

### Integration Tests
- Music service integration
- Context provider functionality
- Cross-component state synchronization

### End-to-End Tests
- First-time user workflow
- Returning user with preferences
- Browser autoplay policy compliance

### Manual Testing Checklist
- [ ] First visit shows music note icon
- [ ] Click starts music and switches to pause icon
- [ ] Subsequent clicks toggle between play/pause
- [ ] Preferences persist across browser sessions
- [ ] Works with keyboard navigation
- [ ] Screen reader announces state changes
- [ ] No console errors on autoplay restrictions

## Implementation Plan

### Phases
1. **Phase 1**: Core 3-state button implementation ✅
   - Music note initial state
   - Play/pause toggle functionality
   - Basic state management

2. **Phase 2**: User preference system ✅
   - localStorage integration
   - Visit detection and preference loading
   - Autoplay compliance

### Timeline
| Phase | Start Date | End Date | Dependencies |
|-------|------------|----------|--------------|
| Phase 1 | 2025-09-25 | 2025-09-26 | MusicContext, Icon components |
| Phase 2 | 2025-09-26 | 2025-09-28 | Browser API testing, localStorage manager |

## Risks & Mitigation

| Risk | Impact | Probability | Mitigation Strategy |
|------|--------|-------------|-------------------|
| Browser autoplay restrictions | High | High | User interaction detection, graceful fallback |
| localStorage availability | Medium | Low | Feature detection and fallback to session state |
| Performance on mobile | Medium | Medium | Debounced state updates, optimized re-renders |

## Security Considerations

- No sensitive data stored in localStorage
- User preferences are client-side only
- No external music service authentication required
- XSS protection through React's built-in sanitization

## Accessibility Considerations

- ARIA labels for screen readers
- Keyboard navigation support (Tab, Enter, Space)
- Visual focus indicators
- State announcements for assistive technology
- High contrast mode compatibility

## Implementation Details

### Current Implementation Status
The music button has been successfully implemented with the following key features:

#### 3-State System
1. **Initial State** (`musicButtonState: 'initial'`)
   - Displays music note icon
   - Shown on first visit or when user preference is "ask"
   - Single click transitions to playing state

2. **Playing State** (`musicButtonState: 'playing'`)
   - Displays pause icon
   - Music is actively playing
   - Click transitions to paused state

3. **Paused State** (`musicButtonState: 'paused'`)
   - Displays play icon  
   - Music is paused but ready to resume
   - Click transitions to playing state

#### Unified Music System Architecture
The music button is part of a **unified music system** that provides consistent playback across the entire application:

**Global Music Context**
- Single source of truth for all music state
- Shared between header button, music-space, and any future music components
- Event-driven synchronization ensures all UI elements reflect current state

**Cross-Page Consistency**
- Music continues playing when navigating between pages
- State persists across page transitions
- Music-space acts as an enhanced interface for the same music system

**Browser Autoplay Compliance**
- User interaction detection through event handlers
- Graceful fallback when autoplay is blocked
- No console errors from premature play attempts

#### Integration with Music Space
When users navigate to the music-space page:
- Music continues playing seamlessly (no interruption)
- Music-space provides rich visual controls for the same audio
- Constellation animations can be enhanced to react to the playing music
- All controls (header button, music-space player) affect the same audio stream

#### User Preference Logic
```typescript
// On app initialization
const checkUserPreference = () => {
  const preference = localStorageManager.getMusicPreference();
  if (preference?.shouldAutoPlay && preference?.hasInteracted) {
    // User has interacted before and prefers autoplay
    setMusicButtonState('playing');
  } else {
    // First time or user prefers to be asked
    setMusicButtonState('initial');
  }
};
```

## Open Questions

- [x] Should the button remember last state (play/pause) or preference (autoplay/ask)?
- [x] How to handle multiple tabs with the same music playing?
- [x] What happens when music fails to load?

## References & Resources

- [Web Audio API Documentation](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)
- [Autoplay Policy Best Practices](https://developer.chrome.com/blog/autoplay/)
- [WCAG 2.1 Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)
- [React Context Patterns](https://react.dev/learn/passing-data-deeply-with-context)

---

## Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-09-25 | Development Team | Initial requirements and basic implementation |
| 2.0 | 2025-09-28 | Development Team | Complete specification with implemented solution |

---

## Appendices

### Appendix A: State Transition Diagram
```
[Initial] --click--> [Playing]
    ↑                    ↓
    |                   click
    |                    ↓
    └-- user pref --- [Paused]
```

### Appendix B: Code Examples
```typescript
// Example usage in header component
<MusicToggleButton 
  className="music-toggle-btn"
  aria-label="Toggle background music"
/>
```