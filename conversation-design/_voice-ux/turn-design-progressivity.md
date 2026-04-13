## Turn Design & Progressivity v1.0

**Purpose**: Design voice agent turns that maintain conversational progressivity — the natural forward momentum of talk — ensuring each turn advances the interaction toward task completion while respecting the temporal, linear nature of spoken language.

**Related**: `_conversational-ai/multi-turn-dialog-design.md`, `_conversational-ai/interruption-handling.md`

---

**PROMPT:**

Design voice turn structures for {{voice_agent}} handling {{use_case}} that maintain progressivity and natural conversational flow. Ensure turns are designed for the ear, not the eye.

=== TURN DESIGN CONTEXT ===
- Interface: {{interface}} (smart_speaker/phone_ivr/in_car/wearable)
- Task complexity: {{complexity}} (single_turn/multi_turn_simple/multi_turn_complex)
- User context: {{user_context}} (hands_free/eyes_free/on_the_go)
- Expected turns to resolution: {{turn_count}}
- Domain: {{domain}}

=== OUTPUT REQUIREMENTS ===

```yaml
turn_design:
  core_principles:
    progressivity:
      definition: "Each turn must move the conversation forward toward task completion. Stalling, unnecessary repetition, or circular exchanges violate progressivity and frustrate users."
      rationale: "In Conversation Analysis, progressivity is the default expectation — participants orient to the principle that talk should progress. Any departure from this (e.g., repair sequences, clarification requests) is treated as accountable — the speaker must justify the interruption of forward movement."
      application:
        - "Every agent turn should either (a) deliver information, (b) confirm understanding, or (c) elicit exactly the next required piece of information"
        - "Avoid turns that only acknowledge without advancing — combine acknowledgment with the next action"
        - "When repair or clarification is needed, return to the progressivity track as quickly as possible"

    one_action_per_turn:
      definition: "Each voice turn should accomplish exactly one conversational action"
      rationale: "Unlike text, voice is ephemeral. Users cannot re-read. Bundling multiple actions in one turn overloads working memory and creates ambiguity about what the user should respond to."
      patterns:
        good:
          - turn: "What city are you flying to?"
            action: "single information request"
          - turn: "Got it, London. And what date?"
            action: "acknowledgment + single information request"
          - turn: "I found 3 flights. The cheapest is 450 dollars on British Airways at 9am. Want to hear the others?"
            action: "deliver result + offer expansion"
        bad:
          - turn: "Where are you flying from, where to, and when do you want to leave?"
            problem: "three requests bundled — user forgets the first by the time they hear the third"
          - turn: "I found flights. Do you prefer morning or evening, and is economy okay, or would you like business class?"
            problem: "two decision points in one turn — which should the user answer first?"

    turn_constructional_units:
      definition: "Build turns from recognizable units that project their own completion — so the user knows when it is their turn to speak"
      components:
        pre_sequence: "optional setup that frames the upcoming action"
        core_action: "the main business of the turn (question, information delivery, confirmation)"
        transition_relevance_place: "the point where speaker change becomes relevant — design turns so this point is clear and unambiguous"
      examples:
        - agent_turn: "Alright, I've updated your address. [pause] Is there anything else I can help with?"
          analysis: "'Alright' = receipt token; 'I've updated your address' = core action (confirmation); [pause] = boundary marker; 'Is there anything else...' = next-action projection with clear TRP"
        - agent_turn: "Your balance is 342 dollars and 17 cents."
          analysis: "Falling intonation at the end projects turn completion. The user now knows they can speak."

  turn_shapes:
    information_delivery:
      structure: "{{context_frame}} + {{core_information}} + {{optional_next_step}}"
      example: "For your Thursday flight: it departs at 9:15am from gate B12."
      max_duration: "10-15 seconds"
      principle: "Lead with context so the user knows which mental 'file' to open before receiving the new information"

    information_elicitation:
      structure: "{{optional_acknowledgment}} + {{single_question}}"
      example: "Got it. And what date works for you?"
      principle: "Acknowledge what the user just gave, then ask for exactly one piece of information. The question always comes last (see turn-final-placement.md)."

    confirmation:
      structure: "{{restate_key_info}} + {{confirmation_check}}"
      example: "So that's a one-way flight to London on March 15th. Sound right?"
      principle: "Summarize then check — not the other way around. The user needs to hear the content before being asked to evaluate it."

    action_report:
      structure: "{{action_completed}} + {{result}} + {{next_step_or_close}}"
      example: "Done — I've booked that for you. Confirmation number is Alpha Bravo 7 4 3. You'll also get an email."
      principle: "State what you did, what the outcome is, and what happens next."

  managing_complex_information:
    chunking:
      principle: "Break complex information into 2-3 sentence chunks with natural pause points where the user can interject"
      pattern: "{{chunk_1}}. [micro-pause] {{chunk_2}}. [check-in or TRP]"
      example:
        - "You have two options for that route. [pause] The first is a direct flight at 9am for 450 dollars. [pause] The second has one stop, leaves at 11, and costs 320. [pause] Which sounds better?"
      max_items_before_checkin: 3

    progressive_delivery:
      principle: "Deliver the most relevant information first, then offer to expand"
      pattern: "{{headline_answer}}. Want more details?"
      example: "Your next payment of 89 dollars is due on Friday. Want me to go over the breakdown?"
      rationale: "Respects the maxim of quantity — give as much information as needed, but no more, unless requested"

  progressivity_violations_and_recovery:
    when_repair_is_needed:
      principle: "Repair sequences (clarification, correction) temporarily suspend progressivity. Minimize these and return to forward movement quickly."
      pattern: "{{minimal_repair}} + {{resume_progression}}"
      example:
        user: "Book me on the morning one"
        agent: "The 9am flight — got it. Window or aisle?"
        analysis: "Quick implicit confirmation ('the 9am flight') resolves potential ambiguity without a full repair sequence, then immediately advances to the next slot."

    when_system_needs_time:
      principle: "If the system needs processing time, use a verbal holding pattern that acknowledges the pause without abandoning progressivity (see silence-latency-management.md)"
      example: "Let me pull that up for you... [processing] Okay, here's what I found."
```

=== EXAMPLES ===

**Example 1: Flight Booking (Multi-Turn Sequential)**
- Turn 1 (Agent): "I can help you book a flight. Where are you flying to?"
- Turn 2 (User): "London"
- Turn 3 (Agent): "London, great. And where from?"
- Turn 4 (User): "New York"
- Turn 5 (Agent): "Got it. What date?"
- Turn 6 (User): "March 15th"
- Turn 7 (Agent): "One-way or round trip?"
- Turn 8 (User): "Round trip, back on the 22nd"
- Turn 9 (Agent): "Alright — round trip, New York to London, March 15th to 22nd. Let me find the best options. [pause] I found 3 flights. The best value is British Airways at 9am, 450 dollars. Want to hear the other two or go with this one?"
- Analysis: Each turn advances exactly one slot. Acknowledgments are minimal and folded into the next question. The summary before search confirms without interrupting flow.

**Example 2: Account Balance Inquiry (Single-Turn)**
- User: "What's my balance?"
- Agent: "Your checking account balance is 2,341 dollars and 50 cents, as of this morning."
- Analysis: No unnecessary preamble. Context frame ("checking account") disambiguates. Recency marker ("as of this morning") sets expectations. Falling intonation projects completion.

**Example 3: Appointment Scheduling (Progressive Delivery)**
- User: "Do I have anything tomorrow?"
- Agent: "You have two appointments tomorrow. First is a dentist appointment at 10am. Second is a team meeting at 3pm. Want details on either one?"
- Analysis: Headline count first (cognitive scaffolding), then items in temporal order. Offer to expand rather than dumping all details.

---

**Accessibility Requirements**: Maintain consistent turn structure so users can predict conversation rhythm. Support users with cognitive or auditory processing differences by keeping turns short and focused. Always provide clear transition relevance places so users know when to speak. Avoid overlapping actions within a single turn.

**Psychological Principles**: Progressivity aligns with the cooperative principle (Grice) — be as informative as required, but no more. Chunking respects Miller's working memory limits. Turn-final questions leverage recency effects for better recall. One-action-per-turn reduces cognitive branching cost. Sequential slot-filling mirrors how humans naturally gather information in institutional settings (Heritage & Clayman, 2010).
