## Barge-In & Overlap Management v1.0

**Purpose**: Handle simultaneous speech, user interruptions (barge-in), and overlapping talk in voice interactions — supporting user agency while preserving conversation coherence.

**Related**: `_conversational-ai/interruption-handling.md`, `_voice-ux/turn-design-progressivity.md`, `_voice-ux/prosodic-design-intonation.md`

---

**PROMPT:**

Design barge-in and overlap management strategies for {{voice_agent}} on {{platform}}. Define when to yield to the user, when to hold the floor, and how to recover after interruptions.

=== BARGE-IN CONTEXT ===
- Barge-in support: {{barge_in_enabled}} (full/partial/none)
- Platform: {{platform}} (smart_speaker/phone_ivr/in_car/wearable)
- Typical utterance length: {{agent_utterance_length}} (short/medium/long)
- Domain: {{domain}}
- Interaction style: {{style}} (transactional/informational/conversational)

=== OUTPUT REQUIREMENTS ===

```yaml
barge_in_management:
  ca_foundation:
    overlap_in_conversation:
      principle: "In natural conversation, overlap (simultaneous speech) occurs regularly — it is not a system failure but a structural feature of turn-taking. Sacks, Schegloff & Jefferson (1974) identified that most overlap occurs at or near transition relevance places (TRPs), where speaker change is legitimately possible. Voice UX should treat barge-in as a natural conversational event, not an error."
      types_of_overlap:
        transitional: "User begins speaking near a TRP — this is orderly, expected overlap"
        recognitional: "User recognizes where the agent's turn is going and responds early — a sign of engagement, not rudeness"
        competitive: "User speaks to actively interrupt — often due to impatience, urgency, or the agent saying something wrong"
        accidental: "Background noise, side conversation, or unintended activation"

  barge_in_detection_and_response:
    immediate_yield:
      when:
        - "User speaks during agent's turn with clear intent (not background noise)"
        - "User's input is a correction ('No,' 'Wait,' 'Actually')"
        - "User's input answers a question the agent is still framing"
      action: "Stop speaking immediately. Process user input. Resume or redirect."
      example:
        agent: "I found three options. The first is a direct flight at—"
        user: "Just the cheapest one"
        agent: "[stops] The cheapest is 320 dollars on United, with one stop. Want me to book it?"
        analysis: "The agent yields immediately and addresses the user's shortcut. No 'As I was saying' — the user's barge-in overrides the agent's plan."

    hold_the_floor:
      when:
        - "Agent is delivering critical information (confirmation numbers, legal disclaimers, safety instructions)"
        - "Agent is mid-word or mid-number and stopping would cause confusion"
      action: "Complete the critical unit, then yield"
      example:
        agent: "Your confirmation number is Alpha Bravo 7—"
        user: "[speaks]"
        agent: "—4 3. Sorry, did you say something?"
        analysis: "The agent finishes the confirmation number (critical unit), then yields. Stopping mid-number would force a repeat."
      design_note: "Use this sparingly. Holding the floor when the user is trying to speak feels controlling."

    noise_filtering:
      when: "Detected audio is likely background noise, cough, or side conversation — not directed at the agent"
      action: "Continue speaking. Do not acknowledge or stop."
      challenge: "Distinguishing user-directed speech from ambient noise is primarily a system/ASR concern, but conversation design can help by providing recovery if the system guesses wrong"
      recovery: "If the agent continues but the user was actually speaking, the user will typically try again — and the agent should yield on the second attempt"

  post_barge_in_recovery:
    user_correction:
      pattern: "Accept correction + resume with corrected information"
      example:
        agent: "I'll book you on the 9am—"
        user: "The 11am"
        agent: "The 11am flight. Got it. Window or aisle?"

    user_shortcut:
      pattern: "Skip ahead to what the user wants"
      example:
        agent: "Let me walk you through your three options. First—"
        user: "Whatever's cheapest"
        agent: "The cheapest is 320 dollars, one stop, arriving at 4pm. Want that one?"

    user_answers_early:
      pattern: "Accept the early answer without remarking on the timing"
      example:
        agent: "Next I'll need your—"
        user: "March 15th"
        agent: "March 15th, got it. And how many passengers?"
        analysis: "The user anticipated the question. The agent doesn't say 'I was about to ask that' — just accepts and moves on."

    user_topic_change:
      pattern: "Acknowledge the switch + save state if needed"
      example:
        agent: "For your hotel, I found three options—"
        user: "Actually, can we go back to the flight? I want to change the time."
        agent: "Sure, let's go back to the flight. What time would you prefer?"

    user_abort:
      pattern: "Stop immediately + confirm cancellation"
      example:
        agent: "I'm now processing your—"
        user: "Stop. Cancel that."
        agent: "Stopped. Nothing has been processed. What would you like to do instead?"

  designing_for_interruptibility:
    chunk_information:
      principle: "Design long utterances with natural barge-in points — places where the user can reasonably interrupt without losing critical information"
      pattern: "{{key_info}} [barge-in-safe point] {{supporting_detail}} [barge-in-safe point] {{next_step}}"
      example:
        agent: "Your total is 847 dollars. [safe] That includes the base fare of 720 and 127 in taxes. [safe] Should I confirm the booking?"
        analysis: "If the user barges in after 'dollars,' they have the key info. If they wait, they get the breakdown. If they wait longer, they get the question."

    barge_in_safe_information_structure:
      principle: "Do NOT simply invert information order to 'front-load' key content — this conflicts with the end-focus principle (see turn-final-placement.md), which places the most salient element at turn-final position for maximum recall. Instead, break long utterances into short, self-contained chunks, each of which individually follows end-focus. This way, even if the user barges in after chunk 1, that chunk has already delivered its key payload at its own end."
      reconciliation_with_end_focus: "End-focus is the default for all turn design. Barge-in resilience is achieved not by abandoning end-focus, but by ensuring that each chunk is a complete informational unit with its own end-focused structure. The user can interrupt between chunks without losing the critical content of any completed chunk."
      pattern: "{{chunk_1_with_end_focused_key_info}}. [barge-in-safe boundary] {{chunk_2_with_end_focused_detail}}."
      examples:
        good:
          turn: "Your flight lands at 3:15. [safe] You're arriving at Terminal B, gate 42."
          analysis: "Chunk 1 delivers the answer to the user's likely core question (arrival time) at chunk-final position. Chunk 2 adds detail, also end-focused (gate number). If the user barges in after chunk 1, they have the essential answer. If they hear both, each chunk respects end-focus independently."
        bad_front_loading:
          turn: "3:15 is when your flight lands, Terminal B, gate 42."
          problem: "Violates end-focus — the most important element (arrival time) is at the start where it has the least perceptual salience and recall. This trades recency-effect benefits for barge-in protection, which is a net loss: most turns are NOT interrupted, so optimizing every turn for barge-in degrades the common case."
        bad_inverted:
          turn: "Coming in to Terminal B, gate 42, your flight is expected to arrive at 3:15."
          problem: "Follows end-focus for the whole turn, but as a single long unit it is not barge-in safe — if the user interrupts after 'gate 42,' they still don't have the arrival time."
      design_rule: "When in doubt, chunk shorter. A turn made of two end-focused chunks is both barge-in safe AND optimized for recall. Front-loading within a chunk is almost never the right trade-off."

    offer_explicit_interruption_points:
      principle: "For long information sequences, build in explicit check-in points"
      pattern: "{{chunk}}. Want to hear more, or is that enough?"
      example: "You have 5 transactions this week. The three largest are a 200-dollar charge at Amazon, 150 at Whole Foods, and 89 at the electric company. Want me to go through the rest?"

  platform_specific_considerations:
    smart_speaker:
      barge_in: "Usually supported via wake word or continuous listening"
      challenge: "Distinguishing household ambient speech from commands"
      guidance: "Support barge-in liberally — smart speaker interactions should feel conversational"

    phone_ivr:
      barge_in: "Often supported via DTMF (keypress) and/or speech"
      challenge: "Legacy systems may not support speech barge-in well"
      guidance: "Announce barge-in capability: 'You can interrupt me at any time.' Support both speech and keypress."

    in_car:
      barge_in: "Critical for safety — drivers must be able to interrupt immediately"
      challenge: "Road noise, passengers speaking"
      guidance: "Aggressive barge-in detection. Short utterances. Frequent check-in points."

    wearable:
      barge_in: "Limited by small microphone and processing power"
      challenge: "False activations from movement"
      guidance: "Keep agent utterances very short to reduce barge-in need"
```

=== EXAMPLES ===

**Example 1: User Corrects Mid-Utterance**
- Agent: "I've scheduled your appointment for Tuesday at—"
- User: "Wednesday"
- Agent: "Wednesday. What time?"
- Analysis: Clean yield, accept correction, advance. No dwelling on the error.

**Example 2: Impatient User Skips Ahead**
- Agent: "Before we proceed, I should let you know about our cancellation policy. If you cancel within 24 hours—"
- User: "I know, I know. Just book it."
- Agent: "Booking now. [pause] Done. Your confirmation number is Alpha Bravo 7-4-3."
- Analysis: Agent respects the user's desire to skip. Does not insist on reading the policy. (If legally required, handle differently — e.g., "I do need to mention one thing quickly: cancellations within 24 hours are fully refundable. Alright, booking now.")

**Example 3: User Answers Before Question Is Fully Asked**
- Agent: "And for the return, when would you like to—"
- User: "The 22nd"
- Agent: "March 22nd. Perfect."
- Analysis: Recognitional overlap — the user knew what the question was going to be. Agent completes smoothly.

**Example 4: Hold Floor for Critical Information**
- Agent: "Important: your reference number is—"
- User: "[coughs]"
- Agent: "—B-K-7-4-9-2. Do you need me to repeat that?"
- Analysis: Agent finishes the critical unit. Then checks if the interruption was intentional.

---

**Accessibility Requirements**: Barge-in support is an accessibility feature — it respects user agency and accommodates different processing speeds. Users with motor speech disorders may have delayed or atypical barge-in timing — systems should have generous overlap windows. Ensure barge-in is available in all modalities (speech and keypress for phone). Never penalize users for interrupting (e.g., by restarting from the beginning).

**Psychological Principles**: User agency and perceived control (Skinner, 1996) — the ability to interrupt increases perceived control and satisfaction. Conversational alignment (Pickering & Garrod, 2004) — smooth barge-in handling signals that the agent is a competent conversational partner. Frustration threshold (Norman, 2013) — inability to interrupt a long agent utterance is one of the top voice UX frustrations. Competitive overlap resolution (Schegloff, 2000) — in natural conversation, the interrupter typically 'wins' the floor; voice UX should mirror this by yielding.
