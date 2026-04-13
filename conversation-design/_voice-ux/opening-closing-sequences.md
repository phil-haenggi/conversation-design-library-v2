## Opening & Closing Sequences v1.0

**Purpose**: Design the opening and closing phases of voice interactions using the structural organization of conversational openings and closings — managing identification, greetings, first topic, pre-closings, and terminal exchanges.

**Related**: `_conversational-ai/closing-conversations.md`, `_conversational-ai/setting-expectations.md`, `_voice-ux/voice-persona-phrasebook.md`, `_voice-ux/silence-latency-management.md`

---

**PROMPT:**

Design opening and closing sequences for {{voice_agent}} in {{domain}} on {{platform}}. Ensure openings efficiently establish context and move to the first topic, and closings provide a complete, unambiguous ending.

=== SEQUENCE CONTEXT ===
- Platform: {{platform}} (smart_speaker/phone_ivr/in_car/wearable)
- Initiation: {{initiation}} (user_calls_agent/agent_calls_user/wake_word)
- Authentication: {{auth_required}} (none/pin/voice_biometric/knowledge_based)
- Domain: {{domain}}
- Persona: {{persona_archetype}}
- Average interaction length: {{avg_length}}

=== OUTPUT REQUIREMENTS ===

```yaml
opening_closing_sequences:
  ca_foundation:
    openings:
      principle: "Schegloff (1968, 1986) described telephone openings as having a systematic structure: summons-answer → identification-recognition → greetings → first topic. Voice UX openings follow this structure but must be compressed — users expect to reach the 'reason for the call' faster than in human-to-human calls."
      key_insight: "The opening is not just pleasantry — it accomplishes critical work: establishing the channel is open, identifying the parties, setting the interactional frame, and launching the first topic. Removing any step creates trouble downstream."
    closings:
      principle: "Schegloff & Sacks (1973) showed that closings are collaboratively negotiated: a pre-closing ('Is there anything else?') creates a slot for new topics; if no new topic is introduced, the terminal exchange ('Goodbye' / 'Bye') follows. Voice UX must reproduce this structure to avoid premature or abrupt termination."
      key_insight: "An abrupt closing (ending without a pre-closing) feels like being hung up on. A missing terminal exchange feels like the call dropped. Both must be present."

  opening_design:
    structure:
      step_1_channel_confirmation:
        what: "Signal that the system is active and ready"
        examples:
          inbound_call: "[ring tone → pickup] Hello..."
          wake_word: "[activation chime]"
          proactive_outbound: "[ring → user answers] Hi, this is {{agent_name}} calling from {{company}}."
        design_note: "This step is often handled by the platform (ring, chime, pickup). The agent's first words should follow immediately — no dead air."

      step_2_identification_and_greeting:
        what: "Identify the agent, optionally greet, and set the interactional frame"
        patterns:
          minimal:
            template: "{{greeting}}, this is {{agent_name}}. {{capability_frame}}"
            example: "Hi, this is Aria. I can help you with your account."
            when: "Returning user, context already established"
          standard:
            template: "{{greeting}}, I'm {{agent_name}} from {{company}}. {{capability_frame_or_how_can_i_help}}"
            example: "Hello, I'm Aria from TeleCo. How can I help you?"
            when: "New or unknown user, general entry point"
          authenticated:
            template: "{{greeting}}, {{user_name}}. {{capability_frame}}"
            example: "Hi, Sarah. What can I do for you today?"
            when: "User is already identified (logged in, caller ID, voice biometric)"
        design_principles:
          brevity: "Keep the opening under 5 seconds of speech. Users are eager to state their problem."
          first_topic_invitation: "Always end the opening with a clear invitation for the user to state their reason for calling. This is the 'anchor position' (see turn-final-placement.md) — the question at the end is what the user responds to."
          avoid_menu_trees: "Do NOT front-load a long list of options ('For billing, say billing. For technical support, say technical support...'). Instead, ask an open question and route based on the response."

      step_3_first_topic_launch:
        what: "The user states their reason, and the agent acknowledges and begins working on it"
        patterns:
          user_states_reason:
            user: "I need to change my flight"
            agent: "Sure, let me pull up your booking."
            analysis: "Immediate acknowledgment + action. No unnecessary intermediate questions."
          user_is_vague:
            user: "I have a question"
            agent: "Of course. What's your question about?"
            analysis: "One follow-up to narrow scope. Not 'What department would you like?' — that's system-oriented, not user-oriented."
          user_gives_multi_part_request:
            user: "I need to change my flight and also check on my hotel"
            agent: "Got it — I'll help with both. Let's start with the flight. What needs to change?"
            analysis: "Acknowledges both parts, sequences them, and launches the first."

    opening_anti_patterns:
      long_greeting:
        bad: "Welcome to TeleCo customer service. We value your business and are committed to providing you with the best possible experience. My name is Aria and I'm here to help you with any account-related inquiries. How may I assist you today?"
        good: "Hi, I'm Aria from TeleCo. How can I help?"
        problem: "The user zones out during the corporate copy and may miss the question."

      options_menu_as_opening:
        bad: "For account inquiries, say 'account.' For billing, say 'billing.' For technical support, say 'technical.' For all other inquiries, say 'other.'"
        good: "Hi, what can I help you with?"
        problem: "Menus impose the system's categories on the user. Open-ended NLU is almost always better."

      no_first_topic_invitation:
        bad: "Hello, this is Aria."
        problem: "The user doesn't know if it's their turn to speak. No question = no clear TRP."
        good: "Hello, this is Aria. How can I help?"

  closing_design:
    structure:
      step_1_pre_closing:
        what: "Signal that the current topic is resolved and check if the user has additional needs"
        function: "The pre-closing creates a 'possible completion point' — if the user has more business, they can introduce it here. If not, the conversation moves toward termination."
        patterns:
          standard:
            - "Is there anything else I can help with?"
            - "Anything else?"
            - "Was there anything else you needed?"
          after_complex_task:
            - "Alright, that's all taken care of. Anything else before I let you go?"
            - "All done on my end. Is there anything else?"
          proactive:
            - "Before you go — your bill is due next week. Want me to set up a reminder?"
            - "One more thing — I noticed your plan might not be the best fit anymore. Want me to look into that?"
        design_note: "The pre-closing MUST be a question with a clear TRP. The user must know it's their turn to either raise a new topic or move toward closing."

      step_2_topic_closure_confirmation:
        what: "If the main task involved an action, confirm the outcome one last time"
        patterns:
          - "So just to recap: {{summary_of_what_was_done}}."
          - "Your {{action}} is all set."
        when: "Only for consequential actions. Not needed for simple information queries."

      step_3_terminal_exchange:
        what: "The mutual goodbye that formally ends the interaction"
        patterns_by_persona:
          warm:
            - "Great, glad I could help. Have a wonderful day!"
            - "Happy to help! Take care."
            - "You're all set. Have a great one!"
          professional:
            - "Thank you for calling. Goodbye."
            - "Glad I could assist. Have a good day."
            - "You're all set. Thank you."
          casual:
            - "Cool, you're good to go. Later!"
            - "All done! Have a good one."
        design_principles:
          always_say_goodbye: "Never just disconnect. Even if the user says 'that's all,' the agent must produce a closing."
          match_user_energy: "If the user says a quick 'bye,' match with a brief 'bye.' If they're chatty, a warmer closing is appropriate."
          no_new_information_after_goodbye: "Once the terminal exchange begins, do not introduce new content. It creates confusion about whether the call is ending."

    closing_anti_patterns:
      abrupt_termination:
        bad:
          agent: "Your flight is booked."
          user: "[silence — waiting for more]"
          agent: "[disconnects]"
        problem: "No pre-closing, no terminal exchange. Feels like being hung up on."

      infinite_loop:
        bad:
          agent: "Anything else?"
          user: "No"
          agent: "Are you sure there's nothing else?"
          user: "No, I'm good"
          agent: "Okay, well if you think of anything..."
        problem: "The user said no. Accept it and close."

      post_closing_addition:
        bad:
          agent: "Thanks for calling, goodbye!"
          agent: "Oh, one more thing — your payment is due Friday."
        problem: "New information after the terminal exchange violates closing structure. The user is mentally disengaged."
        good: "Move the reminder to step_1 (pre-closing): 'Before you go — your payment is due Friday. Want me to set up a reminder? ... Alright, have a good day!'"

  special_opening_closing_cases:
    returning_user_warm_open:
      pattern: "{{personalized_greeting}} + {{context_reference}} + {{invitation}}"
      example: "Welcome back, Sarah. Last time we were setting up your travel insurance. Want to pick up where we left off, or is this about something new?"
      analysis: "Recognition builds rapport. Offering to resume respects prior investment."

    outbound_call_opening:
      pattern: "{{identification}} + {{reason_for_call}} + {{permission_check}}"
      example: "Hi Sarah, this is Aria from TeleCo. I'm calling about the plan change you requested. Is now a good time?"
      critical: "Outbound calls MUST state the reason immediately and ask for permission. Anything else feels like a spam call."

    interrupted_session_reopening:
      pattern: "{{greeting}} + {{state_restoration}} + {{continuation_invitation}}"
      example: "Hi again. We were in the middle of booking your flight — you'd chosen the 9am to London on March 15th. Want to continue from there?"
      analysis: "Full context restoration so the user doesn't have to remember where they left off."

    timeout_closing:
      pattern: "{{acknowledgment_of_silence}} + {{state_preservation}} + {{return_instructions}} + {{goodbye}}"
      example: "It seems like you might have stepped away. I've saved everything we've done so far. Just call back and say 'continue my booking' whenever you're ready. Goodbye!"
      critical: "Always say goodbye, always preserve state, always tell the user how to return."
```

=== EXAMPLES ===

**Example 1: Complete Inbound Call — Open to Close**
- Opening:
  - Agent: "Hi, I'm Aria from TeleCo. How can I help?"
  - User: "I want to check my data usage"
  - Agent: "Sure, let me pull that up."
- [... task interaction ...]
- Closing:
  - Agent: "You've used 12 of your 50 gigs this month. Is there anything else?"
  - User: "Nope, that's it"
  - Agent: "Great. Have a good day!"
- Analysis: Clean four-part structure — greeting+invitation → task → pre-closing → terminal exchange. Under 3 seconds for the opening, under 4 for the closing.

**Example 2: Outbound Call with Permission Check**
- Agent: "Hi, this is Aria from TeleCo. I'm calling because your plan renewal is coming up and there's a better option available. Do you have a couple of minutes?"
- User: "Sure, go ahead"
- Agent: "Great. Your current plan is 50 gigs for 45 dollars a month..."
- Analysis: Identification + reason + permission. The user knows who, why, and whether they need to commit time.

**Example 3: User Raises New Topic at Pre-Closing**
- Agent: "Your flight is booked. Anything else?"
- User: "Actually, yeah — can you check my hotel too?"
- Agent: "Of course. Which hotel?"
- [... second task ...]
- Agent: "All done. Anything else?"
- User: "No, I'm good"
- Agent: "Great, glad I could help with both. Have a good day!"
- Analysis: The pre-closing worked as designed — it surfaced a second topic. The second pre-closing confirms completion. Terminal exchange references both tasks.

---

**Accessibility Requirements**: Openings must clearly identify the agent (for users relying on audio-only identification). Capability framing helps users with cognitive differences understand what the system can do. Pre-closings must be unambiguous questions so the user knows it's their turn. Terminal exchanges must use recognizable closing tokens ("goodbye," "take care") so users know the call is ending — especially important for users with auditory processing differences who might miss subtle cues.

**Psychological Principles**: Primacy effect (first impressions) — the opening sets the tone for the entire interaction. Recency effect — the closing is the last experience and disproportionately affects satisfaction ratings. Zeigarnik effect — unresolved tasks linger; clean closings with summaries create cognitive closure. Schegloff's (1968) summons-answer sequence — openings must establish mutual availability before proceeding. Schegloff & Sacks (1973) — closings are collaborative achievements, not unilateral decisions.
