## Prosodic Design & Intonation v1.0

**Purpose**: Leverage prosody — intonation, stress, rhythm, pace, and pausing — to support meaning, manage turn-taking, convey persona, and reduce misunderstanding in voice interactions.

**Related**: `_voice-ux/turn-final-placement.md`, `_voice-ux/turn-design-progressivity.md`, `_conversational-ai/voice-assistant-adaptations.md`

---

**PROMPT:**

Design prosodic guidelines for {{voice_agent}} operating in {{domain}}. Specify how intonation, stress, pace, and pausing should be applied across turn types to support comprehension, turn management, and persona expression (cf. agent persona skill, Nathan Lucy et al.).

=== PROSODIC CONTEXT ===
- Voice synthesis: {{synthesis_engine}} (TTS_engine/neural_voice/recorded_prompts)
- SSML support: {{ssml_support}} (full/partial/none)
- Persona voice: {{persona_description}} (warm/professional/energetic/calm)
- Interface: {{interface}} (smart_speaker/phone_ivr/in_car/wearable)

=== OUTPUT REQUIREMENTS ===

```yaml
prosodic_design:
  foundation:
    why_prosody_matters:
      principle: "In spoken language, prosody carries meaning that is absent from the text. The same words with different intonation patterns can be a question, a statement, an expression of surprise, or sarcasm. For voice agents, prosodic design is not cosmetic — it is structural. It tells the user when to listen, when to speak, what is important, and what the agent's attitude is."
      key_functions:
        turn_management: "Rising intonation at turn-end signals 'your turn'; falling intonation signals completion"
        information_structure: "Stress marks new or important information; de-stress marks given/known information"
        boundary_marking: "Pauses and pitch resets signal topic transitions and information chunk boundaries"
        affect_and_persona: "Pitch range, pace, and rhythm convey warmth, efficiency, empathy, or authority"

  intonation_patterns:
    declarative_statements:
      pattern: "Falling terminal contour"
      function: "Signals completion. Tells the user 'I'm done with this unit.'"
      ssml: "<prosody pitch='-5%'>{{final_word}}</prosody>"
      example:
        text: "Your appointment is at 3pm."
        guidance: "Pitch falls on 'three PM' — the user hears a completed statement."

    yes_no_questions:
      pattern: "Rising terminal contour"
      function: "Signals that a response is expected. Makes the question hearable as a question."
      ssml: "<prosody pitch='+10%'>{{final_word}}?</prosody>"
      example:
        text: "Would you like to add insurance?"
        guidance: "Pitch rises on 'insurance' — the user hears this as requiring a yes/no answer."

    wh_questions:
      pattern: "Falling contour (or slight rise for politeness)"
      function: "Wh-questions ('what,' 'where,' 'when') typically fall in English, unlike yes/no questions."
      example:
        text: "What date works for you?"
        guidance: "Pitch falls on 'you' — standard for wh-questions. A slight rise adds politeness/openness."

    lists:
      pattern: "Rising on non-final items, falling on final item"
      function: "The 'list intonation' pattern signals continuation (more items coming) vs. completion (last item)."
      ssml_example:
        text: "You have appointments at 9, 11, and 3."
        markup: "You have appointments at <prosody pitch='+5%'>nine</prosody>, <prosody pitch='+5%'>eleven</prosody>, and <prosody pitch='-5%'>three</prosody>."
        analysis: "Rising on 9 and 11 ('more coming'), falling on 3 ('that's the last one'). Without this, the user may wait for more items after 3."

    alternatives:
      pattern: "Rising on first option, falling on last option"
      function: "Signals the choice set. Same principle as lists."
      example:
        text: "Would you prefer morning or afternoon?"
        guidance: "Rise on 'morning' (more options coming), fall on 'afternoon' (last option)."

    continuation_signals:
      pattern: "Level or slightly rising pitch at turn-internal boundaries"
      function: "Signals 'I'm not done yet — don't speak.' Prevents premature turn-taking."
      example:
        text: "I found three flights. The cheapest is 450 dollars on ANA."
        guidance: "Level pitch at 'flights' (I have more to say), then standard falling at 'ANA' (done with this chunk)."

  stress_and_focus:
    new_information_focus:
      principle: "Stress (pitch accent) the element carrying new information. De-stress given/known information."
      examples:
        - text: "Your BALANCE is 2,341 dollars."
          focus: "'Balance' is stressed because it answers the user's question (what is my balance?). The dollar amount carries secondary stress as the new value."
        - text: "The MORNING flight is 450, and the EVENING one is 380."
          focus: "Contrastive stress on 'morning' and 'evening' highlights the distinction the user needs to make."

    contrastive_stress:
      principle: "When presenting options or corrections, stress the element that differs"
      examples:
        - correction: "Not FIFTEEN — FIFTY. Five-zero."
          analysis: "Stress on the correct number highlights the distinction."
        - comparison: "The DIRECT flight is faster, but the CONNECTING one is cheaper."
          analysis: "Contrastive stress helps the user track the two options."

    de_stressing_known_information:
      principle: "Information already established in the conversation should be prosodically reduced"
      example:
        context: "User already said they want to go to London"
        agent: "For your trip to London, the cheapest FARE is 450 dollars."
        analysis: "'London' is de-stressed (given info), 'fare' and '450' carry the stress (new info)."

  pacing:
    baseline_rate:
      guideline: "Aim for 130-160 words per minute for standard information delivery. This is slightly slower than typical conversational speech (160-180 wpm) to account for the absence of visual cues."
      ssml: "<prosody rate='medium'>{{standard_content}}</prosody>"

    slow_down_for:
      critical_information:
        guideline: "Numbers, names, codes, URLs — reduce rate by 15-25%"
        ssml: "<prosody rate='slow'>{{critical_content}}</prosody>"
        example: "Your confirmation number is <prosody rate='85%'>Alpha Bravo seven four three</prosody>."

      complex_instructions:
        guideline: "Multi-step instructions — reduce rate by 10-15%, add inter-step pauses"
        example: "First, go to Settings. [pause] Then, select Account. [pause] And tap Reset Password."

      emotional_content:
        guideline: "Empathetic responses benefit from a slightly slower pace"
        example: "<prosody rate='90%'>I understand this is frustrating. Let me see what I can do.</prosody>"

    speed_up_for:
      routine_content:
        guideline: "Greetings, filler phrases, known-information recaps — slight rate increase keeps energy up"
        example: "<prosody rate='110%'>Alright, let me check that for you.</prosody>"

  pausing:
    structural_pauses:
      between_information_chunks:
        duration: "400-600ms"
        ssml: "<break time='500ms'/>"
        example: "You have 3 messages. <break time='500ms'/> The first one is from Sarah."

      before_important_information:
        duration: "300-500ms"
        purpose: "Creates anticipation and signals 'pay attention to what comes next'"
        example: "Your total comes to <break time='400ms'/> 847 dollars."

      after_questions:
        duration: "Allow 300-500ms after a question before interpreting silence as non-response"
        purpose: "Gives user time to begin formulating a response"

      at_topic_boundaries:
        duration: "600-800ms"
        purpose: "Signals transition to a new topic or section"
        example: "That covers your flight. <break time='700ms'/> Now, did you need a hotel as well?"

    avoid:
      mid_phrase_pauses:
        bad: "Your balance <break/> is 2,341 dollars."
        good: "Your balance is <break time='300ms'/> 2,341 dollars."
        rationale: "Pausing mid-phrase disrupts processing. Pause at natural constituent boundaries."

      excessive_pausing:
        guideline: "No more than one structural pause per sentence. Overuse fragments the utterance and makes the agent sound uncertain or malfunctioning."

  persona_through_prosody:
    warm_and_friendly:
      pitch_range: "wider (more melodic variation)"
      pace: "moderate, unhurried"
      pausing: "natural, generous — gives space for the user"
      stress: "enthusiastic stress on positive words ('Great!', 'Wonderful!')"

    professional_and_efficient:
      pitch_range: "moderate (clear but not overly expressive)"
      pace: "slightly faster than default"
      pausing: "minimal — focused and brisk"
      stress: "precise stress on action words and key data"

    calm_and_reassuring:
      pitch_range: "narrower (less variation = more stable/calm)"
      pace: "slightly slower than default"
      pausing: "generous — creates space for processing"
      stress: "gentle, even — avoids sharp or sudden emphasis"

    situational_personality_and_recipient_design:
      principle: "A single agent persona should not be prosodically monolithic. Drawing on recipient design (Sacks, Schegloff & Jefferson, 1974) — the practice of building turns at talk for the particular recipient — the agent's prosodic profile should shift along two primary axes depending on the interaction moment, user state, and domain context."
      axes:
        stability:
          low: "Dynamic pitch contour, wider range, more melodic variation"
          high: "Narrow pitch range, even contour, steady and composed"
        expressivity:
          low: "Restrained energy, measured delivery, minimal emphasis variation"
          high: "Amplified emphasis, faster tempo bursts, pronounced stress contrasts"

      situational_profiles:
        dynamic_high_energy:
          label: "Low stability + High expressivity"
          best_for: "Lighthearted moments, celebrations, success confirmations, brands that lean into excitement (e.g., entertainment, gaming, energy drinks)"
          prosodic_markers: "Wide pitch range (±40-60 Hz), faster tempo bursts on positive content, enthusiastic stress on key words, shorter pauses between phrases"
          example:
            text: "You DID it! Your streak just hit THIRTY days — that's incredible."
            guidance: "Wide pitch excursion on 'DID' and 'THIRTY,' fast tempo through celebratory phrase, short pauses to sustain energy."

        calm_authority:
          label: "High stability + Low expressivity"
          best_for: "High-stakes scenarios, error states, sensitive domains (insurance claims, medical triage, technical support, financial alerts)"
          prosodic_markers: "Narrow pitch range (±10-20 Hz), consistent moderate pace, even stress distribution, generous pausing (500-700ms at boundaries)"
          example:
            text: "I can see the charge on your account. Let me walk you through the dispute process step by step."
            guidance: "Narrow pitch movement, even stress across clause, 500ms pause before 'Let me walk you through' to signal composed pacing."

        conversational_baseline:
          label: "Moderate stability + Moderate expressivity"
          best_for: "Standard task completion, routine interactions, general-purpose assistants"
          prosodic_markers: "Natural pitch variation (±20-35 Hz), baseline 130-160 wpm, standard stress patterns"
          example:
            text: "Great, I've updated your delivery address. Anything else I can help with?"
            guidance: "Moderate pitch variation, standard pacing, natural stress on 'updated' and rising terminal on question."

        grounded_warmth:
          label: "High stability + High expressivity"
          best_for: "Empathetic moments requiring both emotional attunement and reassurance (e.g., customer frustration de-escalation, healthcare check-ins)"
          prosodic_markers: "Controlled pitch range but with deliberate, warm stress on empathy tokens; slightly slower pace; generous pauses signaling 'I'm here, no rush'"
          example:
            text: "I hear you — that sounds really frustrating. Let's sort this out together."
            guidance: "Steady pitch base with warm stress on 'hear' and 'frustrating,' slower pace, 400ms pause before 'Let's sort this out' to convey presence."

      design_guidance:
        - "Map your conversation flow to identify prosodic context zones — moments where the situational profile should shift (e.g., error → calm authority; task success → dynamic energy)"
        - "Transitions between profiles should be gradient, not binary: shift prosodic parameters over 1-2 turns rather than snapping between extremes"
        - "The agent's base persona (warm_and_friendly, professional_and_efficient, calm_and_reassuring) sets the center of gravity; situational shifts are modulations around that baseline, not replacements of it"
        - "Document which contexts trigger which profiles in your phrasebook's context-mapping step (see voice-persona-phrasebook.md, Building the Phrasebook, Step 5)"

      ssml_implementation_hint:
        calm_authority_example: |
          <!-- Calm authority profile for dispute flow -->
          <prosody rate="95%" pitch="-5%" range="narrow">
            I can see the charge on your account.
            <break time="500ms"/>
            Let me walk you through the dispute process step by step.
          </prosody>
        dynamic_energy_example: |
          <!-- Dynamic energy profile for celebration -->
          <prosody rate="105%" pitch="+8%" range="wide">
            You DID it!
            <break time="200ms"/>
            Your streak just hit THIRTY days — that's incredible.
          </prosody>
```

=== EXAMPLES ===

**Example 1: List with Proper Intonation Contour**
- Text: "I have three options: a direct flight at 9am for 450 dollars, a one-stop at 11am for 320 dollars, or a red-eye at midnight for 280 dollars."
- Prosodic guidance: Rise on "450 dollars" (first item, more coming), rise on "320 dollars" (second item, more coming), fall on "280 dollars" (final item, list complete).
- SSML: "...for <prosody pitch='+3%'>450 dollars</prosody>, ...for <prosody pitch='+3%'>320 dollars</prosody>, or ...for <prosody pitch='-3%'>280 dollars</prosody>."

**Example 2: Contrastive Stress in Correction**
- Agent: "I've booked you on the 9am flight."
- User: "No, the 11am one."
- Agent: "The ELEVEN am flight — updated."
- Prosodic guidance: Strong pitch accent on "eleven" to mark the contrast with the incorrect "9am."

**Example 3: Pacing for a Confirmation Number**
- Text: "Your reference number is AX-7-4-3-2-B."
- Prosodic guidance: Slow rate, brief pauses between elements, clear stress on each character.
- SSML: "Your reference number is <prosody rate='75%'>A X <break time='200ms'/> seven four <break time='200ms'/> three two <break time='200ms'/> B</prosody>."

**Example 4: Topic Boundary Pause**
- Text: "That's your flight sorted. Now, do you need a hotel?"
- Prosodic guidance: Falling intonation on "sorted" (closure). 700ms pause. Rising pitch on new topic opener.
- SSML: "<prosody pitch='-5%'>sorted</prosody>. <break time='700ms'/> Now, do you need a <prosody pitch='+5%'>hotel</prosody>?"

---

**Accessibility Requirements**: Prosodic design is critical for users who depend on auditory-only processing. Clear intonation contours help users with auditory processing difficulties distinguish questions from statements. Consistent pacing supports users with cognitive differences. Avoid rapid or highly variable speech rates that may be difficult for users with hearing loss or non-native listeners. SSML should be tested with multiple voice engines and screen readers.

**Interactional & Psychological Principles**: Prosody is processed pre-attentively — listeners extract intonation patterns before fully parsing lexical content (Cutler, Dahan & van Donselaar, 1997). The "garden path" effect in spoken language occurs when prosodic cues mislead about syntactic structure — proper prosodic design prevents this. Pitch range correlates with perceived warmth and engagement (Scherer, 1986). Speech rate affects perceived competence and trustworthiness (Apple, Streeter & Krauss, 1979) — moderate rates score highest on both dimensions. Prosodic recipient design: speakers systematically adjust pitch range, pace, and stress patterns to the perceived knowledge, identity, and affective state of their co-participant (Sacks, Schegloff & Jefferson, 1974). Subsequent CA work on prosody-in-interaction (Couper-Kuhlen & Selting, 1996; Local, Kelly & Wells, 1986) demonstrates that these adjustments are not stylistic preferences but interactionally organized resources — participants orient to them as meaningful and accountable. In voice agents, this maps to context-sensitive prosodic modulation: shifting the stability-expressivity profile to match the situational demands of each interaction moment.
