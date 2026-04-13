## Silence & Latency Management v1.0

**Purpose**: Design strategies for handling, preventing, and recovering from silences in voice interactions — including system-processing delays, user non-response, connection issues, and the perceptual management of latency.

**Related**: `_conversational-ai/setting-expectations.md`, `_conversational-ai/error-recovery-flows.md`, `_voice-ux/turn-design-progressivity.md`

---

**PROMPT:**

Design silence and latency management strategies for {{voice_agent}} operating on {{platform}} with typical processing latency of {{latency_range}}. Address both system-initiated and user-initiated silences.

=== SILENCE CONTEXT ===
- Typical system latency: {{latency_ms}} (e.g., 500ms-3s)
- User context: {{user_context}} (hands_free/driving/home/office)
- Silence tolerance: {{tolerance}} (low_for_phone_ivr/medium_for_smart_speaker/higher_for_complex_tasks)
- Domain: {{domain}}
- Failure modes: {{failure_modes}} (slow_api/asr_timeout/connection_drop)

=== OUTPUT REQUIREMENTS ===

```yaml
silence_management:
  ca_foundation:
    silence_in_conversation:
      principle: "In conversation analysis, silence is never 'nothing.' It is always interpreted by participants. A gap at a transition relevance place (TRP) — where speaker change is expected — is attributed to the person whose turn it is to speak. If the agent has just asked a question and the user is silent, the silence 'belongs' to the user. If the agent should be responding and is silent, the silence 'belongs' to the agent and will be interpreted as trouble (processing failure, confusion, disconnection)."
      key_insight: "Silences as short as 700ms are perceptible and interpretable in conversation. After ~1 second of unexpected silence, participants begin remedial action. Voice UX must account for this tight timeline."
      reference: "Jefferson (1989) established the 'standard maximum silence' at approximately 1 second — beyond which participants treat the silence as accountable."

  system_processing_silences:
    announcing_upcoming_silence:
      principle: "When the system knows it will need processing time, verbally announce the pause BEFORE it happens. This reframes the silence from 'something is wrong' to 'the system is working.'"
      rationale: "Pre-announcing a silence is a form of 'pre-sequence' (Schegloff, 2007) that sets expectations and maintains the interactional contract."
      patterns:
        brief_hold:
          tokens:
            - "One moment..."
            - "Hold on a second..."
            - "Let me look that up..."
            - "Bear with me..."
            - "Checking that now..."
          duration: "use for expected delays of 1-3 seconds"
          example:
            agent: "Hold on a second... [2s processing] Okay, I found 3 flights for you."
            analysis: "'Hold on a second' projects a brief silence. 'Okay' marks the resumption of talk and signals that the result is coming."

        longer_processing:
          tokens:
            - "This might take a moment. I'm searching for the best options..."
            - "Give me just a moment — I'm pulling up your account..."
            - "Let me run those numbers. This may take a few seconds..."
          duration: "use for expected delays of 3-10 seconds"
          follow_up: "If the delay exceeds the announced expectation, provide a progress update"
          example:
            agent: "Let me search for that. This might take a moment... [5s] Still looking — there are a lot of options... [3s] Okay, here's what I found."

        extended_processing:
          pattern: "announce + explain why + offer alternatives"
          duration: "use for delays over 10 seconds"
          example:
            agent: "I need to check a few systems for this. It might take about 15 seconds. Would you like to wait, or should I call you back with the answer?"

    verbal_fillers_during_processing:
      principle: "Use brief verbal cues during longer silences to signal continued presence and activity — mimicking how human agents say 'mm-hmm' or 'let me see' while searching."
      tokens:
        - "Alright..."
        - "Okay, let me see..."
        - "Hmm, looking at your options..."
        - "Almost there..."
      spacing: "Insert a filler every 3-5 seconds during extended processing"
      caution: "Do not overdo fillers — more than one every 3 seconds feels anxious. The goal is reassurance, not narration."

    resumption_markers:
      principle: "When resuming after a silence, use a discourse marker that signals 'I'm back and ready to continue.' This bridges the gap and re-establishes the conversational frame."
      tokens:
        - "Okay, ..."
        - "Alright, ..."
        - "Right, so ..."
        - "Here we go — ..."
        - "Got it — ..."
      example:
        agent: "Let me pull up your order... [processing] Alright, I can see your order here."

  user_silences:
    at_transition_relevance_place:
      definition: "The agent has asked a question or made a response-relevant utterance, and the user does not respond."
      timeline:
        0_to_3_secondS: "Normal processing time. Do nothing."
        3_to_6_seconds:
          action: "Check availability and/or rephrase"
          patterns:
            - "Hello?"
            - "Are you still there?"
            - "I'm still here whenever you're ready."
            - "Would you like me to repeat that?"
          rationale: "At this point the silence is clearly marked. The agent needs to determine if the user is still present. 'Hello?' is a summons — it makes a response conditionally relevant."
        6_to_10_seconds:
          action: "Rephrase the question or offer alternatives"
          patterns:
            - "Let me ask that differently. {{rephrased_question}}"
            - "If you're not sure, I can {{offer_alternative}}."
            - "Would you like me to come back to this one?"
          rationale: "The extended silence may indicate confusion, not absence. Offer a way forward."
        beyond_10_seconds:
          action: "Graceful timeout with state preservation"
          patterns:
            - "It seems like you might have stepped away. I'll be here whenever you're ready. Just say 'hello' to pick up where we left off."
            - "I haven't heard anything for a bit. I'll keep your progress saved — just speak up when you're back."
          rationale: "Preserve all conversational state. Do not force the user to start over."

    mid_utterance_silence:
      definition: "The user starts speaking but stops mid-sentence"
      approach: "Wait slightly longer than for TRP (Transition Relevance Place) silence (users may be formulating), then gently prompt"
      timeline:
        2_to_4_seconds:
          patterns:
            - "Go ahead, I'm listening."
            - "Take your time."
        4_to_8_seconds:
          patterns:
            - "Did you want to say something else, or should I go ahead with what I have?"

  regaining_attention:
    after_silence_or_disconnection:
      principle: "When resuming after a period of silence or potential disconnection, use a combination of summons + availability check + readiness signal"
      patterns:
        brief_absence:
          - "Hi — still here. Can you hear me?"
          - "Hello? I'm still here if you'd like to continue."
        longer_absence:
          - "Hi...hello. Can you hear me? We were just going over your flight options."
          - "Welcome back. We left off at choosing your seat. Want to continue?"
      structure: "summons ('hi/hello') + availability check ('can you hear me?') + context restoration ('we were going over...')"
      rationale: "The summons (Schegloff, 1968) is the canonical way to initiate or re-initiate contact in conversation. It makes a response conditionally relevant — if the user can hear, they are socially obligated to respond."

  acknowledgment_as_continuity_device:
    principle: "Brief acknowledgment tokens ('okay,' 'alright,' 'right') serve a dual function in voice UX: they confirm receipt of the user's prior turn AND they project that the agent's turn is continuing. This is invaluable for managing perceived latency."
    patterns:
      acknowledgment_plus_processing:
        example:
          user: "I'd like to check my balance"
          agent: "Okay... [1s processing] Your balance is 2,341 dollars."
          analysis: "'Okay' immediately acknowledges the request (zero perceived latency from user's perspective). The processing pause that follows is reframed as 'looking it up' rather than 'didn't hear you.'"

      acknowledgment_plus_topic_shift:
        example:
          user: "Actually, I changed my mind about the hotel"
          agent: "Alright. [pause] What would you like to change?"
          analysis: "'Alright' receives the user's change of direction. The pause gives the agent time to reorient, and the user time to formulate."

      acknowledgment_plus_escalation:
        example:
          user: "This isn't working and I'm getting frustrated"
          agent: "I understand. [pause] Let me get someone who can help you directly."
          analysis: "'I understand' validates the emotion. Pause creates space. Then the action follows."

    persona_through_acknowledgment (cf. agent persona skill, Nathan Lucy et al.):
      principle: "The choice of acknowledgment token is a powerful carrier of persona and linguistic fingerprint (see personality-consistency.md). The tokens an agent habitually uses become part of its voice identity."
      examples:
        professional_persona: ["Certainly.", "Of course.", "Understood."]
        friendly_persona: ["Got it!", "Sure thing.", "Okay, cool."]
        warm_persona: ["Of course.", "Absolutely.", "I'd be happy to."]
        efficient_persona: ["Done.", "Got it.", "On it."]
      guidance: "Choose 3-5 acknowledgment tokens that fit the agent's persona and use them consistently. Avoid mixing registers (e.g., don't alternate between 'Certainly' and 'Sure thing' — pick a lane)."

  latency_budget:
    principle: "Allocate the available silence budget wisely across the interaction"
    guidelines:
      instant_responses: "Acknowledgments, greetings, simple confirmations — under 500ms"
      brief_lookups: "Account info, simple queries — announce + 1-3s"
      complex_processing: "Search results, calculations — announce + explain + 3-10s"
      multi_system: "Cross-system lookups, complex operations — announce + offer callback for 10s+"
    perceived_vs_actual:
      principle: "Perceived latency can be reduced without reducing actual latency"
      techniques:
        - "Immediate acknowledgment token ('Okay...') buys 1-2 seconds"
        - "Pre-announcement ('Let me check...') reframes up to 5 seconds as expected"
        - "Progress narration ('Still searching...') sustains engagement for 5-10 seconds"
        - "Partial results ('I found some options, still loading more...') delivers value early"
```

=== EXAMPLES ===

**Example 1: Quick Lookup with Acknowledgment Bridge**
- User: "What's the status of my order?"
- Agent: "Let me check... [2s] Your order shipped yesterday and is expected Thursday."
- Analysis: "Let me check" fills the gap immediately. 2 seconds of silence is acceptable because it was announced. Resumption with direct answer maintains progressivity.

**Example 2: User Non-Response After Question**
- Agent: "Would you like the window or aisle seat?"
- [3 seconds of silence]
- Agent: "No rush — window or aisle?"
- [5 more seconds]
- Agent: "If you're not sure, I can assign the best available. Would that work?"
- Analysis: Graduated response. First: gentle prompt with repetition. Second: offers an escape hatch. Never pressures.

**Example 3: Extended Processing with Progress Updates**
- User: "Find me the cheapest round trip to Tokyo in April"
- Agent: "Okay, let me search for that. Flights to Tokyo in April — this might take a moment... [4s] I'm comparing a few airlines... [3s] Alright, I found some good options. The cheapest is 845 dollars on ANA, departing April 7th."
- Analysis: Immediate acknowledgment → expectation setting → progress filler → resumption marker → answer. The 7-second total delay feels much shorter because of the verbal scaffolding.

**Example 4: Regaining Attention After Silence**
- Agent: "What date works for you?"
- [12 seconds of silence]
- Agent: "Hello? I'm still here. We were picking a date for your appointment. Want to continue, or should I save your spot and you can call back?"
- Analysis: Summons → presence signal → context restoration → two clear options (continue or save state). Respects user autonomy.

**Example 5: Acknowledgment Tokens Carrying Persona**
- Warm/friendly agent:
  - User: "I need to change my flight"
  - Agent: "Of course! Let me pull that up... [2s] Alright, I've got your booking here."
- Efficient/professional agent:
  - User: "I need to change my flight"
  - Agent: "Got it. One moment... [2s] I have your booking. What would you like to change?"
- Analysis: Same function, different persona. "Of course!" vs "Got it." — the acknowledgment token is a micro-expression of character.

---

**Accessibility Requirements**: Users with hearing impairments using voice interfaces via captions or hearing aids need explicit verbal signals for processing pauses — never rely on audio tones alone. Users with anxiety or cognitive differences may interpret silence as failure — always explain pauses. Timeout warnings must be clear and offer state preservation. Never force users to repeat information after a timeout.

**Interactional & Psychological Principles**: Jefferson's (1989) "standard maximum silence" of ~1 second sets the baseline for acceptable gaps. The "accountability" of silence (Heritage, 1984) means every silence is interpreted — unmanaged silence generates negative attributions. Pre-announcements (Schegloff, 2007) reduce uncertainty by setting expectations. Acknowledgment tokens serve as "continuers" (Schegloff, 1982) that maintain the interactional channel. The mere exposure effect means consistent acknowledgment token choice reinforces persona recognition over time.
