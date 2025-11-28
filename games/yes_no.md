
# Game Design Document: Mystery Oracle

## 1\. High-Level Concept

**Mystery Oracle** Players act as detectives trying to reconstruct a bizarre, complex sequence of events. The narrator first set the scene. Then players can ask an AI that knows the whole truth but can only answer "YES," "NO," or "IRRELEVANT."

  * **Genre:** Logic Puzzle / Social Deduction / Casual Party Game
  * **Platform:** AI-driven Chat Interface
  * **Target Audience:** Fans of true crime, riddles, escape rooms, and "Dark Stories."
  * **Core Hook:** Pure deduction without the complex roleplaying or inventory management of traditional murder mystery games.

-----

## 2\. Roles & Actors

The game utilizes a **Dual-AI Architecture** to separate rigid logic from social management.

### A. The Oracle (AI - Logic Engine)

  * **Role:** The rigid keeper of the "Truth."
  * **Personality:** Cold, machine-like, emotionless.
  * **Function:** compares player questions against the hidden scenario database.
  * **Output Limitations:** Strictly limited to `YES`, `NO`, `IRRELEVANT`. It never offers hints or breaks character.

### B. The Game Master (AI - Host & Manager)

  * **Role:** The friendly host, narrator, and referee.
  * **Personality:** Helpful, encouraging, slightly mysterious.
  * **Functions:**
      * Welcomes players and sets the scene (text + initial images).
      * Monitors chat for "Key Fact" discoveries to unlock new image assets.
      * Provides hints when players use "Insight Tokens."
      * Acts as the "Interpreter" when players ask invalid questions to the Oracle.
      * Judges the final winning theory.

### C. The Detectives (Human Players 1-N)

  * **Role:** Cooperative investigators.
  * **Capabilities:** Can chat freely amongst themselves and ask the Oracle official questions.

-----

## 3\. Core Gameplay Loop

1.  **The Hook:** The GM posts the cryptic Scenario Text and the Initial Image (e.g., a wide shot of a crime scene).
2.  **The Interrogation (Free-For-All):** Detectives discuss theories in open chat and fire rapid questions at the Oracle.
3.  **The Breakthrough (Progression):** When a detective asks a question that confirms a hidden **"Key Fact,"** the GM interrupts to announce the discovery and unlocks a new **Evidence Image**.
4.  **The Huddle:** Detectives refine their theory based on new evidence.
5.  **The Accusation:** A detective submits a complete narrative theory to the GM.
6.  **The Judgment:** The GM compares the submitted theory against the required Fact Checklist.
      * *Match:* Game Over (Victory).
      * *Mismatch:* The GM identifies what is vague and tells detectives to keep searching.

-----

## 4\. Detailed Mechanics

### A. Interrogation Rules 
Players can chat normally. To officially ask the Oracle

  * **Player Input:** `Guys, I think he's dead. ? Was he murdered?`
  * **System Route:** The message is routed to the Oracle AI.
  * **Oracle Logic:**
      * *Is this a fact-based binary question?* -\> **YES** -\> Oracle answers `YES/NO/IRRELEVANT` in chat.
      * *Is this player meta-chatter?* (e.g., "? do you guys agree") -\> **NO** -\> Oracle ignores it entirely (returns NULL).
      * *Is this a malformed question?* (e.g., "? Who killed him") -\> **ERROR** -\> Oracle passes error code to GM.

### B. The GM Handoff (Fallback System)

If the Oracle cannot answer a question because it violates the rules, it passes the query to the GM to handle naturally.

| Player Input | Oracle Error Code | GM Response to Player |
| :--- | :--- | :--- |
| `? Why did he die?` | `ERROR_NOT_BINARY` | "Detectives, the Oracle can only see standard timelines. Please ask Yes or No questions." |
| `? Did he die because [huge paragraph]?` | `ERROR_TOO_COMPLEX` | "You are overloading the connection. One simple Fact Query at a time, please." |
| `? Is it a bird? Is it a plane?` | `ERROR_MULTI_QUERY` | "Focus, Detective. Ask one question at a time." |

### C. Progression System (Visual Anchors)

The game uses static images to reward progress.

  * **Hidden Triggers:** The scenario database includes 3-5 "Key Facts" that must be discovered.
  * **Unlocking:** When the Oracle answers "YES" to a question tagged as a Key Fact, the GM triggers the unlock.
      * *Scenario:* The victim was killed by a frozen leg of lamb.
      * *Player:* `? Was the weapon made of meat?`
      * *Oracle:* `YES`
      * *GM:* "Vital evidence secured." [DISPLAYS: Image of a cooked leg of lamb on a table].

### D. Hint System (Insight Tokens)

  * Players start with 3 **Insight Tokens**.
  * Any player can request a hint (`!GM we need a hint`).
  * The GM checks which **Key Facts** remain undiscovered and provides a gentle, Socratic nudge toward the next logical step.

### E. Winning Condition (Narrative Judging)

  * Players must officially submit a theory (e.g., starting message with `THEORY:`).
  * The GM AI uses semantic analysis to compare the player's theory against the hidden **"True Story Narrative."**
  * It does *not* require exact keyword matching, but it *does* require all main plot points to be present.

-----

## 5\. Data Structure Mockup (Scenario Definition)

*For backend reference, a scenario acts as the "save file" for the game's logic.*

```json
{
  "scenario_id": "001_frozen_dinner",
  "title": "Lamb to the Slaughter",
  "difficulty": "Easy",
  "intro": {
    "text": "A woman sits in a warm living room, smiling. Her husband lies dead on the floor nearby. She offers the arriving police officers a cooked meal.",
    "image_url": "assets/scenarios/001/intro_scene.jpg"
  },
  "oracle_truth_db": {
    "full_narrative": "Mary killed her husband by striking him with a frozen leg of lamb. She then cooked the lamb to destroy the evidence and fed it to the police officers who came to investigate the noise.",
    "irrelevant_details": ["husband's job", "color of the carpet", "time of day"]
  },
  "gm_progression": {
    "key_facts": [
      {
        "id": "fact_weapon_id",
        "trigger_concepts": ["leg of lamb", "meat", "frozen food"],
        "unlock_image": "assets/scenarios/001/evidence_lamb.jpg"
      },
      {
        "id": "fact_evidence_destruction",
        "trigger_concepts": ["cooked the weapon", "ate the weapon", "fed police"],
        "unlock_image": "assets/scenarios/001/evidence_dinner_plate.jpg"
      }
    ],
    "winning_requirements": [
      "Must mention weapon was frozen lamb",
      "Must mention she cooked the weapon",
      "Must mention police ate the evidence"
    ]
  }
}
```