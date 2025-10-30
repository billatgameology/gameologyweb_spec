# Multiplayer Game UI/UX Patterns

**Spec ID:** `ui-multiplayer-001`  
**Version:** `1.0`  
**Status:** `Approved`  
**Created:** `2025-10-29`  
**Last Updated:** `2025-10-29`  
**Author(s):** `Bill W.`  

## Overview

This specification defines the common UI/UX patterns and design elements that should be consistent across all multiplayer games in Gameology. These patterns ensure a cohesive user experience and reduce cognitive load when players switch between different multiplayer games.

## Purpose

- Create a consistent, recognizable experience across all multiplayer games
- Reduce implementation time by reusing proven patterns
- Ensure accessibility and mobile-first design principles
- Maintain clear authentication requirements and user flows

## Scope

This specification applies to all multiplayer games including:
- Tic Tac Toe
- Pixel Match
- Any future multiplayer game implementations

## Common Design Elements

### 1. Page Header & Title

**Visual Structure:**
- Large, bold game title at the top
- Optional game icon/emoji before the title
- Short, descriptive subtitle beneath the title (one sentence maximum)
- Clean, uncluttered appearance

**Rules:**
- Game title should be clear and prominent
- Subtitle should be concise and describe the core gameplay
- Remove verbose generic phrases like "Real-time multiplayer strategy game"
- No status badges or mode indicators in the header (e.g., "Ready to Play")

**Examples:**
- ✅ "Pixel Match" with subtitle "Find the matching symbol - click it on BOTH cards to score!"
- ✅ "Multiplayer Tic Tac Toe" (title only, no subtitle needed)
- ❌ "Real-time multiplayer strategy game" (too generic and verbose)
- ❌ Including a "Ready to Play" badge (unnecessary clutter)

### 2. Authentication Gate

**Requirement:**
All multiplayer games MUST require user authentication before allowing gameplay. No guest mode is permitted.

**Unauthenticated State Display:**
When a user who is not logged in visits a multiplayer game page, they see:

1. A card with the heading "Sign In to Start Playing"
2. A brief description: "You need to be logged in to create or join games"
3. A short benefit statement explaining why login is required (e.g., "Track your wins, losses, and play with friends!")
4. A prominent "Sign In to Play" button

**Login Flow:**
- Clicking the sign-in button redirects to the login page
- After successful login, the user is automatically returned to the game page they came from
- The game automatically uses the user's display name from their profile
- No manual name entry is required

**Rules:**
- No guest mode for multiplayer games (authentication is required)
- Always redirect users back to the game after login
- Display clear benefits of signing in
- Use the user's profile display name automatically (no manual entry)

### 3. Lobby Interface

**Visual Structure:**
When a logged-in user is in the game lobby, the interface contains:

**Top Section - Create New Game:**
1. Section heading: "Create New Game"
2. Game-specific setup options (e.g., deck selection for Pixel Match)
3. Large, prominent "🆕 Create New Game" button with emoji icon

**Middle Section - Visual Divider:**
- Horizontal line with centered text "or join existing" between the sections

**Bottom Section - Join Game:**
1. Section heading: "Join Game with Game ID" (single combined heading)
2. Text input field with placeholder "Enter game ID"
3. Large, prominent "🔗 Join Game" button with emoji icon (blue/primary color)

**Error Display:**
- If there's an error, it appears at the very top in a red/destructive color box
- Clear, actionable error messages

**Rules:**
- NO separate card header with "Start Playing" title
- NO subtitle like "Create a new game or join an existing one"
- NO manual "Your Name" input field (uses user's profile display name automatically)
- Create Game section always comes first
- Join Game section always comes second
- Section headings are simple and clear
- "Join Game with Game ID" is ONE heading (not separate "Game ID" label + heading)
- Both buttons are large for easy clicking/tapping on mobile
- Join Game button uses the primary/blue color (not outline style)
- Button icons: 🆕 for Create, 🔗 for Join

### 4. Game ID Display & Sharing

**Visual Structure:**
When in an active game, the Game ID is displayed with sharing tools:

1. A badge showing the Game ID (e.g., "ID: abc123")
2. A copy button (clipboard icon) next to the ID
3. A QR code button next to the copy button
4. Visual feedback when copied (checkmark icon appears)

**QR Code Dialog:**
When the QR code button is clicked:
- A dialog/modal opens with a scannable QR code
- The Game ID is displayed below the code
- A copy button is available in the dialog
- Instructions explain that scanning joins the game directly

**Rules:**
- Game ID is always visible and easy to find during active games
- Copy-to-clipboard functionality is always available
- QR code generation for easy mobile device joining
- Success feedback shows when ID is copied (icon changes to checkmark)
- QR code includes the full game URL for automatic joining

### 5. Error Handling

**Visual Appearance:**
- Errors appear in a red/destructive colored box with light red background
- Text is clear and easy to read
- Box has a subtle border to stand out
- Positioned prominently at the top of the relevant section

**Message Content:**
- Clear, concise error messages
- Explain what went wrong
- Guide users toward resolution when possible
- Examples:
  - "Please enter a game ID"
  - "Game not found. Please check the game ID."
  - "Please set your display name in your profile"

**Behavior:**
- Errors clear automatically when the user takes corrective action
- Errors don't persist after the issue is resolved
- Only one error displays at a time (most recent/relevant)

### 6. Loading States

**Visual Appearance:**
When an action is in progress (creating game, joining game, etc.):

1. Button shows a spinning animation icon
2. Text changes to action-specific loading message
3. Button is disabled/grayed out to prevent double-clicks
4. Loading text is clear about what's happening

**Loading Text Examples:**
- "Creating..." (when creating a new game)
- "Joining..." (when joining an existing game)
- "Starting..." (when starting a match/rematch)

**Rules:**
- Always show loading state for actions that take time
- Use spinner icon with descriptive text
- Disable button during loading to prevent multiple submissions
- Loading text should be action-specific, not generic
- Remove loading state immediately when action completes

### 7. Mobile Responsiveness

**Design Principles:**
- All multiplayer games must be designed mobile-first
- Touch targets (buttons, inputs) are large enough for fingers
- Text is readable on small screens
- Layout adapts gracefully from phone to tablet to desktop

**Specific Requirements:**
- Lobby interfaces use centered, narrow containers (approximately 448-672px max width)
- Button sizes are large for easy tapping on mobile devices
- Text sizes scale appropriately (larger on desktop, readable on mobile)
- Elements stack vertically on mobile devices
- Adequate spacing between interactive elements to prevent mis-taps
- No horizontal scrolling required on mobile devices

**Responsive Behavior:**
- Titles scale from smaller on mobile to larger on desktop
- Cards and containers adapt width to screen size
- Touch areas remain easily tappable across all screen sizes

### 8. Navigation

**Global Navigation:**
- Use the standard site header for global navigation (home, games menu, profile, etc.)
- Header remains consistent across all game pages

**In-Game Navigation:**
- Provide a "Back to Lobby" button during active games
- Button is clearly visible and accessible at all times during gameplay

**Removed Elements:**
- No redundant bottom navigation buttons (like "Back to Games" or "Dashboard")
- These functions are handled by the global header
- Keeps the interface clean and focused on gameplay

**Rules:**
- Don't duplicate navigation that's already in the header
- Provide clear exit paths from active games
- Keep navigation simple and unobtrusive during gameplay

### 9. Player Join Notifications

**Purpose:**
Show existing players when a new player joins the game session, creating social presence and excitement.

**When to Display:**
- When a new player successfully joins an existing game
- Only shown to players already in the game (not to the joining player)
- Appears immediately after join is successful

**Visual Structure:**
- Fixed position popup at top center of viewport
- No dark background overlay (does not block interaction)
- Green/success themed card with border and subtle shadow
- Welcome icon (e.g., 👋 wave emoji)
- Player name prominently displayed
- Brief subtitle (e.g., "Get ready to play")
- Auto-dismisses after 4 seconds
- Smooth slide-in animation from top

**Design Specifications:**
- Position: `fixed top-4 left-1/2 -translate-x-1/2 z-50`
- Background: Solid color (light green-100 in light mode, green-800 in dark mode)
- Border: 2px solid green border (green-300 light, green-700 dark)
- Text: High contrast (green-900/green-100 for main text)
- Shadow: `shadow-lg` for elevation
- Animation: Slide in from top, duration 300ms
- No modal overlay or backdrop
- Does not block user interaction with page

**Content:**
- Icon: Wave emoji or similar friendly gesture
- Primary text: "[Player Name] has joined!"
- Secondary text: "Get ready to play"
- Font: Bold for name, regular for subtitle

**Behavior:**
- Appears immediately when player joins
- Remains visible for 4 seconds
- Dismisses automatically (no close button needed)
- Can be shown multiple times if multiple players join
- Does not interrupt gameplay or page interaction

**Variant: Opponent Left Notification**
- Uses same popup pattern and position
- Yellow/warning theme instead of green (yellow-100/yellow-800 background)
- Icon: Wave emoji 👋
- Primary text: "Opponent left the game"
- Secondary text: "Returning to lobby..."
- Auto-returns user to lobby after 3 seconds
- Applies when opponent leaves during end-game phase

### 10. Ready-Up System

**Game Types & Player Requirements:**

**Fixed Player Count Games (e.g., Tic Tac Toe = exactly 2 players):**
- Ready-up system appears when exact player count is met
- All required players must be present before ready-up is available

**Flexible Player Count Games (e.g., 2-4 players):**
- Ready-up system appears as soon as minimum players join
- Game can start with minimum players if all ready up
- No need to wait for maximum player count
- Example: In a 2-4 player game, if 2 players join and both click Ready, game starts immediately

**When to Display:**

1. When minimum required players have joined (for flexible count games)
2. When exact required players have joined (for fixed count games)
3. After all present players have successfully entered the game session
4. Before the actual gameplay begins

**Visual Structure:**

**Instruction Message:**
- A prominent message box appears after minimum/required players join
- Fixed count: "All players are here! Click Ready to start the game"
- Flexible count (minimum met): "Minimum players ready! Click Ready to start, or wait for more players to join"
- Flexible count (maximum met): "Maximum players reached! Click Ready to start the game"
- Uses an informative color (blue/info, not error or warning)
- Positioned prominently so all players see it

**Player List with Ready Status:**
- Display list of all players currently in the game
- Each player's display name is shown
- Visual indicator (green checkmark ✓) appears next to players who have clicked Ready
- Players who haven't clicked Ready have no checkmark or a waiting indicator
- Your own name may be highlighted to make it easy to identify
- List updates in real-time as new players join (for flexible count games)

**Ready Button:**
- Large, prominent "Ready" button available to all players
- Button shows loading/disabled state after clicking
- May change to "Waiting for others..." after user clicks
- Once clicked, cannot be un-readied (prevents trolling)
- If new player joins a flexible count game, existing ready states are maintained

**Game Start Trigger:**

**For Fixed Count Games:**
- Game starts when ALL players have clicked Ready
- No partial starts allowed

**For Flexible Count Games:**
- Game starts when ALL currently present players have clicked Ready
- Even if below maximum player count
- Example: 2-4 player game with 2 players → both click Ready → game starts
- Example: 2-4 player game with 3 players → all 3 click Ready → game starts
- Once game starts, no more players can join that session

**Rules:**
- Ready system appears based on minimum player requirement
- For flexible count: game can start before maximum is reached
- All present players must ready up before game starts
- Ready status is visible to all players in real-time
- System prevents game start with any player not ready
- Clear visual feedback for ready/not ready states
- New joins during ready-up reset or maintain ready states (game-specific)

**Example Flow (Tic Tac Toe - Fixed 2 Players):**
1. Player 1 creates game → waits in lobby
2. Player 2 joins → notification "Player 2 has joined" shows to Player 1
3. Message appears: "All players are here! Click Ready to start the game"
4. Player list shows both names with no checkmarks
5. Player 1 clicks Ready → green checkmark ✓ appears next to Player 1's name
6. Player 2 sees the checkmark and clicks Ready
7. Green checkmark ✓ appears next to Player 2's name
8. Game automatically starts since all players are ready

**Example Flow (Pixel Match - 2-4 Players Flexible):**
1. Player 1 creates game → waits in lobby
2. Player 2 joins → notification "Player 2 has joined"
3. Message appears: "Minimum players ready! Click Ready to start, or wait for more players to join"
4. Player 1 clicks Ready → checkmark appears
5. Player 2 clicks Ready → checkmark appears
6. Game starts with 2 players (minimum met, all ready)

**Alternate Flow (Pixel Match - Waiting for More):**
1. Player 1 creates game → waits in lobby
2. Player 2 joins → ready message appears
3. Player 1 clicks Ready → checkmark appears
4. Player 3 joins → notification "Player 3 has joined"
5. Player list shows Player 1 (✓ ready), Player 2 (waiting), Player 3 (waiting)
6. Player 2 clicks Ready → checkmark appears
7. Player 3 clicks Ready → checkmark appears
8. Game starts with 3 players (all ready)

### 11. End-of-Game Flow & Play Again System

**When a Game Round Ends:**

After a game finishes (win, loss, or draw), players see a game result modal with:
- Result title (e.g., "You Won! 🎉", "You Lost 😢", "It's a Draw! 🤝")
- Game statistics (total moves, game time, etc.)
- Player list showing who won
- Two action buttons: "Play Again" and "Back to Lobby"

**Play Again Request System:**

**Initial State:**
- "Play Again" button shows normal text: "Play Again"
- "Back to Lobby" button available as alternative

**When First Player Clicks Play Again:**
- That player's button changes to: "Waiting for others..."
- That player's button becomes disabled (can't un-request)
- Other players see their button update to: "Play Again (1/2)" for 2-player games
- Other players see their button update to: "Play Again (2/4)" for 4-player games (showing count)
- Button dynamically updates as more players request rematch

**When All Players Click Play Again:**
- New game session is created automatically
- Players are moved to the new game session with a new session ID
- Game state resets to "waiting" (ready-up phase)
- All player positions are maintained (or swapped for fairness - game specific)
- Ready-up system activates immediately

**If Not All Players Want to Play:**
- Players can click "Back to Lobby" at any time
- No penalty for declining
- **2-Player Games:** If one player leaves, the remaining player sees a notification "Opponent left the game" and is automatically returned to lobby
- **Multi-Player Games (3+ players):** Game session ends when any player leaves
- Remaining players see updated count if someone leaves before all have left
- Clear notification when session ends due to player departure

**Visual Feedback:**
- Clear counter showing "X/Y" where X = players who want to play, Y = total players
- Button state changes clearly visible
- Loading state while new game is being created
- Dismissible notification when opponent/player leaves

**Example Flow (2-Player Game):**
1. Game ends → both see result modal
2. Player 1 clicks "Play Again" → button shows "Waiting for others..." (disabled)
3. Player 2 sees button update to "Play Again (1/2)"
4. Player 2 clicks "Play Again"
5. New game session created automatically
6. Both players see ready-up interface (Section 10)
7. Follow ready-up flow from Section 10

**Example Flow (2-Player Game - One Player Leaves):**
1. Game ends → both see result modal
2. Player 1 clicks "Play Again" → button shows "Waiting for others..." (disabled)
3. Player 2 sees button update to "Play Again (1/2)"
4. Player 2 clicks "Back to Lobby" instead
5. Player 1 sees notification: "Opponent left the game"
6. Player 1 is automatically returned to lobby
7. Result modal closes automatically

**Example Flow (4-Player Game - Partial Agreement):**
1. Game ends → all four see result modal
2. Player 1 clicks "Play Again" → shows "Waiting for others..."
3. Players 2-4 see "Play Again (1/4)"
4. Player 2 clicks "Play Again" → shows "Waiting for others..."
5. Players 3-4 see "Play Again (2/4)"
6. Player 3 clicks "Back to Lobby" → leaves session
7. Players 1-2-4 see "Play Again (2/3)" (denominator updates)
8. Player 4 clicks "Back to Lobby" → session ends
9. Players 1-2 automatically return to lobby

### 12. Game Start Messages

**All Players Ready Confirmation:**

When all players have clicked Ready and the game is about to start:

**Display Sequence:**
1. Ready-up interface disappears
2. A confirmation card appears prominently in the center
3. Card displays: "All players ready! Game begins..." with animated countdown or immediate transition
4. Card uses success colors (green background/border)
5. Card appears for 2-3 seconds before game starts
6. Optional: Brief animation (fade in, scale up) for emphasis

**Visual Structure:**
- Large text: "All players ready!"
- Secondary text: "Game begins..." or "Starting game..."
- Success/green color scheme
- Center of game area
- May include animated icon (e.g., ✓ checkmark, game icon)

**Turn-Based Games - First Turn Announcement:**

For turn-based games (like Tic Tac Toe), immediately after the "All players ready" message:

**Display:**
1. A second card appears (or the ready card transitions)
2. Card announces: "[Player Name] goes first!" or "Your turn!" / "Opponent's turn!"
3. May include player's symbol (e.g., "X goes first!")
4. Card appears for 2-3 seconds
5. Uses informative color (blue/primary)
6. Then transitions to game board with turn indicator visible

**Visual Structure:**
- Large text: "[Player Name] goes first!"
- Player symbol or avatar if available
- Informative/blue color scheme
- Brief appearance (2-3 seconds)
- Smooth transition to active game board

**Turn Indicator During Play:**
- Persistent, smaller turn indicator remains visible during gameplay
- Shows current player's turn
- Updates in real-time as turns change
- Positioned near game board or in sidebar

**Example Flow (Turn-Based Game):**
1. Both players click Ready
2. Card appears: "All players ready! Game begins..." (2 seconds)
3. Card transitions to: "Player X goes first!" (2 seconds)
4. Game board becomes active
5. Turn indicator shows "Your turn!" or "Opponent's turn!"
6. Players take turns until game ends

**Example Flow (Real-Time Game):**
1. All players click Ready
2. Card appears: "All players ready! Game begins..." (2 seconds)
3. Game starts immediately with all players able to act
4. No turn announcement needed

**Rules:**
- Always show "All players ready" confirmation before starting
- For turn-based games, always announce who goes first
- Keep messages brief and clear (2-3 seconds each)
- Use appropriate colors (green for ready, blue for turn info)
- Smooth animations and transitions
- Messages should feel exciting, not like a delay
- Consider adding sound effects for enhanced feedback

## Implementation Checklist

When creating a new multiplayer game, ensure:

**Authentication & User Identity:**
- [ ] Authentication gate is implemented (no guest mode)
- [ ] User's `displayName` is used (no manual name input)
- [ ] Login redirect returns user to the game

**Lobby Interface:**
- [ ] Lobby follows standard layout (Create → Divider → Join)
- [ ] Card has no redundant header/title
- [ ] "Join Game with Game ID" as section heading (not separate label)
- [ ] Both action buttons are large size
- [ ] Join Game button is primary/blue color (not outline)

**Player Experience:**
- [ ] Player join notifications show for existing players
- [ ] For fixed-player games: Ready-up system is implemented
- [ ] Player list shows ready status with green checkmarks
- [ ] Game starts automatically when all players are ready
- [ ] Clear messaging when waiting for players or ready-up
- [ ] "All players ready" confirmation card displays before game starts
- [ ] Turn-based games show "[Player] goes first" announcement
- [ ] Play Again button shows request count (X/Y players)
- [ ] New game session created when all players request rematch
- [ ] Ready-up system re-activates after Play Again rematch

**Error Handling & Feedback:**
- [ ] Error handling displays prominently
- [ ] Loading states on all async actions
- [ ] Success feedback for actions (copy, ready, etc.)

**Game Session:**
- [ ] Game ID sharing with copy and QR code
- [ ] "Back to Lobby" button during active games

**Design & Responsiveness:**
- [ ] Clean header without clutter (no status badges)
- [ ] Mobile-responsive design
- [ ] Large touch targets for mobile

## Benefits

1. **Consistency**: Players immediately understand how to start/join games
2. **Speed**: Developers can reference this pattern for new games
3. **Quality**: Prevents common UX mistakes and inconsistencies
4. **Maintenance**: Updates to patterns propagate to all games
5. **Accessibility**: Standard patterns are easier to make accessible
6. **Social Presence**: Join notifications and ready system create engagement
7. **Fair Start**: Ready-up system ensures all players are prepared before game begins
8. **Clear Transitions**: Game start messages create excitement and set expectations
9. **Rematch Flow**: Smooth play-again system keeps players engaged

## Future Considerations

- Spectator mode patterns
- Tournament/bracket UI patterns
- Voice chat integration patterns
- Replay/recording UI patterns
- Achievement/trophy display patterns
- Player disconnection/reconnection handling
- Mid-game player replacement for dropped players
- Timeout/AFK detection and handling
- Game pause/resume functionality (for appropriate game types)

## Related Specifications

- `game-pixel-match` - Pixel Match game specification
- `ui-auth-001` - Authentication flow patterns
- `ui-responsive-001` - Mobile-first design guidelines (future)

## Version History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 1.0 | 2025-10-29 | Initial specification | Bill W. |

---

**Notes:**
- This specification should be referenced when creating any new multiplayer game
- Deviations from these patterns should be documented and justified
- Pattern updates should be communicated to all game developers
