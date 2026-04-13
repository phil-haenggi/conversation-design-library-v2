## No-Input & No-Match Escalation v1.0

**Purpose**: Design escalation strategies for the two canonical voice-specific failure modes — no-input (user says nothing) and no-match (ASR returns no usable transcription or NLU returns no confident intent) — with graduated severity and graceful exits.

**Related**: `_conversational-ai/error-recovery-flows.md`, `_conversational-ai/fallback-strategies.md`, `_voice-ux/silence-latency-management.md`, `_voice-ux/voice-conversation-repair.md`

---

**PROMPT:**

Design no-input and no-match escalation flows for {{voice_agent}} in {{domain}} on {{platform}}. Define the graduated response strategy across {{max_attempts}} attempts, including reprompting, rephrasing, modality shifts, and graceful exits.

=== ESCALATION CONTEXT ===
- Max attempts per failure: {{max_attempts}} (typically 2-3)
- Platform: {{platform}} (smart_speaker/phone_ivr/in_car/wearable)
- Escalation options: {{escalation_paths}} (human_agent/callback/text_fallback/email)
- Domain: {{domain}}
- User base: {{user_base}}
- Call deflection tolerance: {{deflection_tolerance}} (low/medium/high)

=== OUTPUT REQUIREMENTS ===

```yaml
no_input_no_match:
  ca_foundation:
    accountable_absence:
      principle: "When a question has been asked, the absence of a response is 'accountable' — the questioner is entitled to pursue a response. In conversation analysis, this is the basis for 'pursuit sequences' (Pomerantz, 1984) where speakers re-do questions that did not receive answers. Voice UX no-input handling IS a pursuit sequence. The design challenge is pursuing a response without being aggressive."
    preference_for_progressivity:
      principle: "Each re-prompt or retry is a departure from progressivity. The system should aim to return to the forward track as quickly as possible — either by getting the information through a different means or by finding a way to proceed without it."

  no_input_escalation:
    definition: "The system expected user speech but detected silence beyond the configured timeout (typically 3-8 seconds depending on context)."
    causes:
      - "User is thinking / formulating"
      - "User didn't realize it was their turn"
      - "User stepped away or is distracted"
      - "Connection or microphone issue"
      - "User doesn't know the answer"
      - "User is confused by the question"

    attempt_1:
      label: "gentle_re_prompt"
      strategy: "Repeat the question with minimal modification. Assume the user didn't hear or needs a moment."
      approach:
        - "Add a brief softener before repeating"
        - "Do NOT rephrase yet — the user may have understood but simply needs more time"
      patterns:
        after_a_question:
          - "Take your time. {{original_question}}"
          - "No rush. {{original_question}}"
          - "I'm still here. {{original_question}}"
        after_a_prompt:
          - "Whenever you're ready."
          - "Go ahead whenever you like."
      example:
        original: "What date works for you?"
        attempt_1: "Take your time. What date works for you?"
      tone: "Patient, unhurried. No indication of error."

    attempt_2:
      label: "rephrase_and_help"
      strategy: "Rephrase the question and/or offer options. The user may be confused by the original phrasing or may not know what kind of answer is expected."
      approach:
        - "Simplify the question"
        - "Offer concrete examples or options"
        - "Provide context about what kind of answer you need"
      patterns:
        - "Let me ask that differently. {{rephrased_question}}"
        - "For example, you could say something like {{example_answer}}. {{simplified_question}}"
        - "I can also help if you say {{option_1}} or {{option_2}}."
      examples:
        original: "What date works for you?"
        attempt_2: "Let me ask that differently. Do you have a specific date in mind, or would you like me to show you what's available this week?"
      tone: "Helpful, offering a way forward. Still no blame."

    attempt_3:
      label: "offer_alternatives_or_exit"
      strategy: "Assume continued silence means the user either cannot or does not want to answer. Offer alternative paths or a graceful exit."
      approach:
        - "Offer a different interaction modality (text, callback, transfer to human)"
        - "Offer to skip the current field and return to it later"
        - "Offer to save state and let the user return"
      patterns:
        - "I haven't heard anything. Would you like me to {{alternative_action}}?"
        - "If it's easier, I can send you a link to finish this online. Want me to do that?"
        - "I'm going to save your progress. You can call back anytime to pick up where we left off."
        - "Let me connect you to an agent who can help. One moment."
      example:
        original: "What date works for you?"
        attempt_3: "No worries if you're not sure right now. I can text you a link to book online, or I can connect you with someone. What would you prefer?"
      tone: "Supportive, exit-oriented. Absolute zero blame."

    beyond_attempt_3:
      action: "Graceful close with state preservation"
      pattern: "It looks like we've lost you. I've saved everything so far — just call back when you're ready and we'll pick up where we left off. Goodbye."
      design_note: "Always say goodbye. Never just disconnect silently — that feels like a dropped call."

  no_match_escalation:
    definition: "The system received user speech but could not determine intent or extract usable entities. ASR may have transcribed the audio, but NLU returned no confident match."
    causes:
      - "User utterance is out of scope"
      - "User phrasing is unexpected but in scope"
      - "ASR transcription error"
      - "User is speaking to someone else"
      - "Background noise interpreted as speech"
      - "User is rambling or providing too much context"

    attempt_1:
      label: "acknowledge_and_redirect"
      strategy: "Acknowledge that you didn't understand, take responsibility, and gently redirect. Do NOT repeat the same question verbatim — the user already answered it in a way you didn't understand."
      approach:
        - "Own the failure ('I didn't quite get that' — not 'You weren't clear')"
        - "Redirect toward the specific information you need"
        - "Offer a constrained format if the open-ended one failed (this is the least preferred strategy, however; do not teach commands/do not teach the user how to talk)"
      patterns:
        - "I didn't catch that. Could you tell me just the {{specific_field}}?"
        - "Sorry, I'm not sure I understood. Are you looking to {{option_1}} or {{option_2}}?"
        - "Hmm, I didn't get that. Try saying just the {{data_type}} — for example, {{example}}."
      example:
        original: "How can I help you?"
        user: "Well, the thing is, my neighbor told me about this plan thing and I was wondering if I could maybe, you know, change whatever I have right now..."
        attempt_1: "I want to make sure I help you with the right thing. Are you looking to change your current plan?"
        analysis: "The agent extracts the likely intent ('change plan') from the rambling input and offers a targeted understanding check rather than a generic 'I didn't understand.'"
      tone: "Apologetic (mildly), constructive. Never confused-sounding."

    attempt_2:
      label: "constrain_and_guide"
      strategy: "Narrow the interaction to a very specific, closed set of options. If open-ended didn't work, go closed-ended."
      approach:
        - "Offer a small set of explicit choices"
        - "Switch from open question to yes/no or multiple choice"
        - "Provide examples of valid responses"
      patterns:
        - "Let me try a different way. You can say {{option_1}}, {{option_2}}, or {{option_3}}."
        - "I can help with things like {{capability_1}}, {{capability_2}}, or {{capability_3}}. Which sounds closest?"
        - "Is this about {{most_likely_topic}}? Just say yes or no."
      example:
        attempt_2: "Let me help narrow it down. I can help you change your plan, check your balance, or manage add-ons. Which one?"
      tone: "Structured, guiding. Reducing cognitive load."

    attempt_3:
      label: "offer_alternatives_or_escalate"
      strategy: "The voice channel isn't working for this user right now. Offer alternative paths."
      approach:
        - "Transfer to human agent"
        - "Offer text/chat alternative"
        - "Offer callback"
        - "Provide self-service URL"
      patterns:
        - "I'm having trouble understanding. Let me connect you with a person who can help."
        - "This might be easier to handle in our app or online. Can I text you a link?"
        - "I appreciate your patience. I'm going to transfer you to an agent who can take it from here."
      example:
        attempt_3: "I'm sorry — I'm not getting this right. Let me transfer you to someone who can help. I'll pass along what we've discussed so you don't have to repeat anything."
      tone: "Honest, apologetic without groveling. Preserves user dignity."
      critical: "ALWAYS pass context to the next channel. Never make the user start over."

  cross_cutting_design_principles:
    never_blame_the_user:
      always: "I didn't catch that." / "I'm having trouble understanding."
      never: "I can't understand you." / "Your response was not recognized." / "Invalid input."
      rationale: "Blaming language is face-threatening and makes users feel incompetent. System-attribution keeps trust intact."

    vary_the_reprompt:
      principle: "Never repeat the exact same prompt twice in a row. Each attempt must feel different — different wording, different strategy, different energy."
      bad: "What date? ... What date? ... What date?"
      good: "What date works? ... Take your time — do you have a specific date, or should I suggest some? ... No worries, I can text you a link to book online."

    preserve_state_across_escalation:
      principle: "If the user is transferred to another channel, agent, or modality, ALL previously collected information must travel with them."
      implementation: "Pass conversation context (collected fields, user intent, interaction history) to the receiving channel."

    track_cumulative_failures:
      principle: "Track failures across the full conversation, not just per-question. If the user has hit no-match on 3 different questions, the voice channel may not be viable — offer escalation even if each individual question hasn't exhausted its attempts."
      threshold: "2-3 cumulative failures across different prompts should trigger a proactive escalation offer."

    tailor_timeouts_to_context:
      principle: "Not all silences should have the same timeout"
      guidelines:
        simple_yes_no: "3-4 seconds"
        factual_recall: "5-6 seconds (user may need to look something up)"
        open_ended: "6-8 seconds (user may be formulating)"
        emotional_context: "8-10 seconds (user may be upset or hesitant)"
        after_bad_news: "10+ seconds (give the user space to process)"
```

=== EXAMPLES ===

**Example 1: No-Input — Booking Flow**
- Agent: "What date works for you?"
- [silence 5s]
- Agent (attempt 1): "No rush. What date were you thinking?"
- [silence 5s]
- Agent (attempt 2): "Let me ask differently. Would you like to travel this week, next week, or do you have a specific date?"
- [silence 6s]
- Agent (attempt 3): "No worries. I can text you a link to book online whenever you're ready. Or I can save this and you can call back. Which would you prefer?"
- [silence 8s]
- Agent (close): "I'll save your progress. Just call back anytime and say 'continue my booking.' Have a good day."

**Example 2: No-Match — Open-Ended Deflection to Closed**
- Agent: "How can I help you today?"
- User: "[lengthy, unfocused explanation about a bill discrepancy]"
- Agent (attempt 1): "It sounds like you have a question about your bill. Is that right?"
- User: "Well, not exactly, it's more like the charges don't match what I was told..."
- Agent (attempt 2): "Got it. I can help with billing questions, dispute a charge, or check your payment history. Which is closest to what you need?"
- User: "Dispute it, I guess"
- Agent: "I'll help you dispute a charge. Which charge on your bill is the issue?"
- Analysis: Open → understanding check → constrained options → resolution. Progressivity restored in 3 turns.

**Example 3: Cumulative Failure — Proactive Escalation**
- [User has hit no-match on 2 previous questions in the conversation]
- Agent: "What's the account holder's name?"
- User: "[unclear response]"
- Agent: "I'm sorry — I've been having some trouble today. This might go faster with one of my colleagues. Let me connect you, and I'll pass along everything we've covered so far. One moment."
- Analysis: Rather than retry the same failing pattern, the agent proactively escalates after cumulative failures. Honest framing ("I've been having some trouble") preserves user dignity.

**Example 4: Contextual Timeout — After Bad News**
- Agent: "I'm sorry, but that flight has been cancelled."
- [silence 8s — user processing]
- Agent: "Take your time. When you're ready, I can help you find an alternative flight or arrange a refund."
- Analysis: The agent uses a longer timeout and a gentler re-prompt because the context (bad news) warrants giving the user space.

---

**Accessibility Requirements**: No-input handling must accommodate users with speech onset delays, stuttering, or motor speech difficulties — extend timeouts generously and never rush. No-match handling must not penalize users with accents, non-native pronunciation, or atypical speech patterns — offer spelling, keypad, and text alternatives early. Escalation to human agents must preserve all context to avoid re-explanation, which is especially burdensome for users with communication difficulties.

**Interactional & Psychological Principles**: Learned helplessness (Seligman, 1975) — repeated failures with no clear path forward lead to disengagement; graduated escalation prevents this. Attribution theory (Weiner, 1985) — attributing failure to the system (not the user) preserves self-efficacy and willingness to try again. Reactance theory (Brehm, 1966) — users who feel trapped (no exit, no alternatives) become frustrated; always offer exits. The "2-3 strike" convention is widely recognized in voice UX (Turunen & Hakulinen, 2003) and matches user expectations for when the system should try something different.
