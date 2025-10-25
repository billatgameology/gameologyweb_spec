# Pixel Match Mini Web Game Specification

**Spec ID:** `game-pixel-match`  
**Version:** `1.0`  
**Status:** `Draft`  
**Created:** `2025-10-25`  
**Last Updated:** `2025-10-25`  
**Author(s):** `Bill W.`  
**Game URL:** `/play/pixel-match`

## Game Overview

### Game Concept
Pixel Match is a fast-paced visual puzzle game where players compete to find the single matching icon between two cards. The game's unique feature is the AI-powered Deck Builder, allowing players to create, manage, and play with their own custom-generated icons.

### Game Category
- [ ] Strategy
- [x] Puzzle 
- [x] Action/Reflex
- [ ] Chance/Luck
- [x] Memory
- [ ] Classic Arcade

### Target Audience
- **Primary Age Group**: All ages
- **Gaming Experience**: Casual
- **Session Length**: 1-5 min
- **Device Preference**: Cross-platform (Mobile-first design)

## Game Mechanics

### Core Gameplay Loop
`[Game Setup]` → `[Start Hosting]` → `[Lobby]` → `[All Players Ready]` → `[Deal Cards]` → `[Players search for match]` → `[First player clicks the correct match]` → `[Award Point & Show Stats]` → `[Deal New Cards]` → `[End of Game]`

### Game Rules
1.  **Objective**: To be the player with the most points at the end of the game by being the fastest to spot the matching icon between two cards each round.
2.  **Setup**: The player who initiates the game is the "Host". On the setup screen, the Host selects their desired Icon Deck. Each card will always have 8 icons. After selecting the deck, a "Start Hosting" button becomes active (e.g., changing from grey to green).
3.  **Lobby & Joining**: Clicking "Start Hosting" takes the Host to a lobby screen. This screen displays player slots (1-4 players), a button to copy the game session ID, and a button to show a QR code for joining. As other players join, their display names fill the player slots.
4.  **Starting a Round**: The Host can start the game at any point. The start button's behavior depends on the player count:
    *   **1 Player**: The button reads "Play Alone" and starts the game immediately on click.
    *   **2+ Players**: The button reads "Start Game". When a player clicks it, they enter a "ready" state, and the button's text updates to show the ready count (e.g., "2 of 3 players ready"). Players who are not ready will see a message like "Other players are waiting". The game begins only when all players in the session are ready.
    *   **Late Joiners**: If a player joins while others are in a "ready" state, the game will not start until the new player also readies up.
5.  **Player Actions**: Players can click or tap on any icon on the two displayed cards.
6.  **Turn Structure**: The game is real-time. All players view the same two cards simultaneously and race to find the match.
7.  **Win Conditions**: The first player to correctly click the matching icon wins the round and is awarded one point.
8.  **Lose Conditions**: There is no losing condition, only not winning a round. Incorrect clicks can optionally be penalized with a short cooldown.
9.  **Special Rules**: With 8 symbols per card, the game requires 57 unique icons total. If the player's selected custom deck has fewer than 57 icons, it will be automatically supplemented with icons from the "Default House Deck" to make the game playable.

### Game States
| State | Description | Player Actions | Transitions |
|-------|-------------|----------------|-------------|
| Home | The main landing page of the game. | Navigate to Play, Library | → Game Setup, Icon Library |
| Game Setup | The "Play" screen where the Host configures game settings. | Select Deck, Click "Start Hosting" | → Lobby |
| Lobby | Staging area where players gather before the game starts. | Copy Session ID, Show QR Code, Click Ready/Start Game | → Playing |
| Playing | Active gameplay where two cards are shown. | Click/Tap an icon | → Round Over |
| Round Over | A player has found a match. | Start Next Round | → Playing |
| Game Over | All rounds are complete or a score limit is reached. | Play Again, View Scores, Return to Home | → Game Setup, Home |
| Icon Library | User manages their master icon collection and builds decks. | View all icons, Filter by deck, Add/Remove icons from decks, Rename decks, Create new icon | → Create Icon |
| Create Icon | User generates a new icon using AI. | Enter text prompt, Generate, Save to Library | → Icon Library |

### Scoring System
- **Base Points**: 1 point is awarded to the player who wins a round.
- **High Score**: The system will track total wins and fastest match times.

## User Interface Specification

### Visual Layout
The application is divided into four main pages:
1.  **Home Page**: Central navigation to Play or Manage Library.
2.  **Library & Deck Builder Page**: Shows all user-generated icons in a full grid layout. Deck tabs (1, 2, 3, 4) at the top allow filtering to view icons in a specific deck. Icons display small badges indicating which deck(s) they belong to. Each deck name can be customized.
3.  **Lobby/Staging Page**: A screen showing a list of 1-4 player slots, with controls for the host to start the game and for all players to share the game session.
4.  **Play Page**: A top control panel for game settings and a main area below to display the game cards side-by-side.

### Game Board/Play Area
- **Dimensions**: Each card is a 3x3 grid capable of displaying up to 9 icons.
- **Visual Style**: Clean, modern, and minimalist, consistent with the `shadcn/ui` component library.
- **Interactive Elements**: Icons on the cards are buttons. Hover effects will indicate they are clickable.
- **Visual Feedback**: On a correct match, the matching icon is highlighted, and a success indicator is shown. On an incorrect click, a subtle shake or color change provides feedback.

### Control Scheme
#### Desktop Controls
- **Mouse**: Click to select an icon.

#### Mobile Controls
- **Touch**: Tap to select an icon.

### UI Components
| Component | Purpose | States | Responsive Behavior |
|-----------|---------|--------|-------------------|
| Game Cards | Display the icons for the current round (8 icons per card). | Active, disabled | Scale proportionally. Stack vertically on narrow screens. |
| Deck Selector | Dropdown to choose between the 4 user decks and the default deck. | Default, disabled | Standard form element behavior. |
| Icon Library | Grid display of all user-created icons with deck membership badges. | Default, empty state, filtered by deck | Grid columns adjust based on screen width. |
| Deck Tabs | Buttons to switch between viewing all icons or filtering by Deck 1, 2, 3, or 4. | All Icons, Deck 1, Deck 2, Deck 3, Deck 4 | Stack or scroll horizontally on mobile. |
| Deck Name Editor | Inline text input to rename each deck. | Default, editing | Standard text input behavior. |
| Deck Membership Badges | Small visual indicators on icons showing which deck(s) (1, 2, 3, 4) they belong to. | None, 1 deck, multiple decks | Scale down on smaller icons. |
| Modal Dialogs | Show round/game over stats, errors, or confirmations. | Open, closed | Full-screen on mobile. |
| Style Selector | Dropdown on the "Create Icon" page for choosing the icon's visual style. | Default, disabled | Standard form element behavior. |

## Core Feature Details

### 1. AI Icon Generation
- **Flow**: Users navigate to the "Create Icon" page. The UI will feature a dropdown menu to select a visual style and a text input box for the core subject of the icon. The final prompt sent to the AI is a combination of the selected style and the user's text. The backend then calls an image generation AI, uploads the resulting image to cloud storage, and saves the icon's metadata (prompt, image URL) to the user's database collection.
    - **Example Styles**:
        - "a simple, clean, minimalist icon of ${prompt}, cartoon style, on a white background"
        - "a pixel art icon of ${prompt}"
        - "a flat, 2D geometric icon of ${prompt}"
- **Technology**: A Next.js API route will handle the request to prevent exposing API keys. It will call a service like Google's Imagen 3 to generate the image.
- **Persistence**: Generated icons are stored in a user-specific `iconLibrary` collection in Firestore, with the image file itself in Firebase Storage.

### 2. Deck Building
- **Concept**: Users have four customizable "Deck Slots" (Deck 1, Deck 2, Deck 3, Deck 4). Each deck can be renamed to help users organize their collections (e.g., "Animals", "Fantasy", "Space Theme").
- **Library View**: The Deck Building page displays all icons generated by the user in a full grid layout. Icons are not automatically added to any deck upon creation - they exist in the user's master library first.
- **Deck Indicators**: If an icon has been added to one or more decks, visual indicators (e.g., small badges or colored dots labeled "1", "2", "3", "4") appear on top of the icon to show which deck(s) it belongs to.
- **Deck Selection**: Clicking on a deck tab (Deck 1, 2, 3, or 4) will filter the view to show only the icons currently in that deck, making it easy to review and manage each deck's contents.
- **Adding/Removing Icons**: Users can click on any icon in the library to add it to the currently selected deck. If the icon is already in that deck, clicking will remove it.
- **Data Model**: Each deck is a document in a `decks` collection in Firestore, containing the deck's name and an array of `iconIds` that reference documents in the `iconLibrary`.
- **Gameplay Integration**: On the "Play" page, the user selects one of their decks. The game logic fetches the icon data for that deck. Since the game always uses 8 symbols per card (requiring 57 unique icons), if the deck has fewer than 57 icons, the "Smart Deck-Filling" logic will pull unique icons from the `DEFAULT_HOUSE_DECK` to meet the required count.

### 3. Game Logic
- **Card Generation**: The `generateSpotItDeck(8)` algorithm, based on finite projective planes, is used to create the card structure with 8 symbols per card. It outputs an array of cards, where each card is an array of symbol IDs.
- **Symbol Mapping**: The generic symbol IDs from the algorithm are mapped to the actual `iconId` strings from the active deck (user's custom deck + fillers).
- **Matching**: When a player clicks an icon, the game checks if that icon's ID exists in the symbol arrays of both cards currently in play. If so, it's a match.

