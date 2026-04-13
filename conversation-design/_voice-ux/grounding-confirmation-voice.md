## Grounding & Confirmation in Voice v1.0

**Purpose**: Design grounding procedures — the practices by which speakers establish mutual understanding — for voice-only interactions, where the absence of visual feedback makes explicit verbal confirmation essential.

**Related**: `_conversational-ai/confirmation-patterns.md`, `_conversational-ai/acknowledgment-responses.md`, `_voice-ux/turn-design-progressivity.md`, `_voice-ux/sequential-information-gathering.md`

---

**PROMPT:**

Design grounding and confirmation strategies for {{voice_agent}} handling {{use_case}} where {{risk_level}} consequences depend on accurate mutual understanding. Balance confirmation thoroughness against conversational efficiency.

=== GROUNDING CONTEXT ===
- Risk level: {{risk_level}} (low/medium/high/critical)
- Domain: {{domain}}
- Information type: {{info_type}} (names/numbers/addresses/preferences/actions)
- ASR confidence: {{asr_confidence}} (high/medium/low)
- Interface: {{interface}} (smart_speaker/phone_ivr/in_car/wearable)

=== OUTPUT REQUIREMENTS ===

```yaml
  confirmation_types:
    implicit_confirmation:
      definition: "The agent weaves the understood information into its next turn without explicitly asking for verification. The user can correct if the agent got it wrong, but no confirmation turn is required."
      mechanism: "Works because of the 'let it pass' principle — if the user does not correct, the information is treated as grounded."
      when_to_use:
        - "Low-stakes information (preferences, non-critical details)"
        - "High ASR confidence (>0.90)"
        - "Information that is easily reversible if wrong"
        - "To maintain conversational pace in routine interactions"
      pattern: "{{acknowledged_value_woven_into_next_action_or_question}}"
      examples:
        - user: "I'd like to fly to London"
          agent: "For your London trip, what dates work?"
          analysis: "'London' is embedded in the next question — implicitly confirmed. If the user meant 'Lisbon,' they will correct. No confirmation turn was needed."
        - user: "Make it for two people"
          agent: "Two guests. Any dietary restrictions?"
          analysis: "'Two guests' is an implicit echo-confirmation. The agent moves directly to the next question."
        - user: "Wednesday afternoon"
          agent: "Wednesday afternoon. I have slots at 2 and 4. Which do you prefer?"
          analysis: "Brief echo + immediate progression. If the user meant Thursday, they will correct here."
      cost: "Zero extra turns. Progressivity is maintained."
      risk: "If the agent misheard and the user doesn't catch the implicit confirmation, the error propagates."

    explicit_confirmation:
      definition: "The agent restates its understanding and directly asks the user to verify before proceeding."
      mechanism: "Creates an adjacency pair: confirmation request → confirmation/disconfirmation. Adds one turn but guarantees mutual understanding."
      when_to_use:
        - "High-stakes information (financial amounts, medical details, legal actions)"
        - "Moderate or low ASR confidence (<0.90)"
        - "Irreversible actions (deletions, transfers, cancellations)"
        - "Homophones, similar-sounding values, or ambiguous input"
        - "Critical identifiers (account numbers, confirmation codes, names)"
      patterns:
        yes_no_check:
          template: "{{restated_information}}. Is that right?"
          example: "Transfer 500 dollars to your savings account. Is that right?"
        value_verification:
          template: "Just to confirm — {{value}}?"
          example: "Just to confirm — that's fifty, five-zero?"
        action_confirmation:
          template: "I'm about to {{action}}. Should I go ahead?"
          example: "I'm about to cancel your subscription. Should I go ahead?"
      cost: "One extra turn (confirmation request + user yes/no)."
      benefit: "Prevents costly errors. Builds user trust for high-stakes operations."

    graduated_confirmation:
      definition: "A hybrid approach that starts with implicit confirmation and escalates to explicit only when risk increases or ambiguity is detected."
      principle: "This mirrors natural conversation — speakers don't confirm every word, but they do confirm when the stakes are high or when something sounds off."
      pattern:
        low_risk_phase: "Implicit confirmation (echo in next question)"
        medium_risk_phase: "Implicit + summary checkpoint after gathering all fields"
        high_risk_phase: "Explicit confirmation before executing"
      example_flow:
        turn_1:
          user: "I want to transfer some money"
          agent: "Sure. From which account?"
          grounding: "implicit — 'transfer' acknowledged by proceeding"
        turn_2:
          user: "From checking"
          agent: "Checking. And where to?"
          grounding: "implicit echo — 'checking' confirmed by repetition"
        turn_3:
          user: "To savings"
          agent: "Got it. How much?"
          grounding: "implicit — no echo needed, low ambiguity"
        turn_4:
          user: "Five hundred"
          agent: "So that's 500 dollars from checking to savings. Correct?"
          grounding: "explicit — full summary before irreversible action"
        turn_5:
          user: "Yes"
          agent: "Done. 500 dollars transferred."
          grounding: "action confirmation closes the sequence"

  grounding_techniques:
    echo_confirmation:
      definition: "Repeat back the critical element, optionally reformulated"
      pattern: "{{echoed_or_reformulated_value}}. {{next_action}}"
      examples:
        verbatim_echo:
          user: "March 15th"
          agent: "March 15th. And the return date?"
        reformulated_echo:
          user: "Next Tuesday"
          agent: "That's March 18th. Does that work?"
          analysis: "Reformulating 'next Tuesday' to a specific date adds precision and invites correction if the agent's calendar calculation is wrong."
        phonetic_echo:
          user: "My name is Schwartz"
          agent: "Schwartz — S-C-H-W-A-R-T-Z?"
          analysis: "Spelling out names catches ASR errors that a simple echo would not."

    digit_by_digit_confirmation:
      definition: "For numbers, read back digit by digit to disambiguate"
      when: "Phone numbers, account numbers, amounts where fifteen/fifty confusion is possible"
      pattern: "{{number}} — that's {{digit_by_digit}}?"
      examples:
        - user: "My account number is 4873921"
          agent: "4-8-7-3-9-2-1. Is that correct?"
        - user: "Fifty dollars"
          agent: "Fifty — five-zero. Right?"
        - user: "Flight at fifteen hundred"
          agent: "3pm — fifteen hundred. Got it."
          analysis: "Reformulating '1500' to '3pm' provides a different encoding that further reduces ambiguity."

    summary_confirmation:
      definition: "Restate all collected information as a complete summary before executing an action"
      when: "Before any irreversible or complex action. After gathering 3+ fields."
      pattern: "Alright, let me confirm: {{field_1}}, {{field_2}}, {{field_3}}. Should I go ahead?"
      example: "Alright, let me confirm: round trip from Boston to London, March 15th to 22nd, one passenger, economy. Should I go ahead and search?"
      design_note: "The summary serves as the 'last chance' repair opportunity. It is the user's final checkpoint before commitment. Always pause briefly after the summary to give the user time to process before asking for confirmation."

  grounding_failures_and_recovery:
    user_disconfirms:
      pattern: "Accept correction + re-ground the corrected value + resume"
      example:
        agent: "500 dollars from checking to savings. Correct?"
        user: "No, 50 dollars"
        agent: "50 dollars — five-zero. From checking to savings. Got it. Should I go ahead?"
        analysis: "Corrected value is re-grounded with digit confirmation. The rest of the summary is preserved. One extra confirmation turn for the corrected field only."

    partial_disconfirmation:
      pattern: "Fix the specific field, preserve the rest, re-confirm only the change"
      example:
        agent: "Round trip, Boston to London, March 15th to 22nd. Correct?"
        user: "March 15th to the 20th, not the 22nd"
        agent: "Changed to March 20th. Everything else the same?"
        analysis: "Only the changed field is re-grounded. 'Everything else the same?' is a lightweight confirmation that avoids repeating the entire summary."

    ambiguous_confirmation:
      problem: "User says something that could be confirmation or new information"
      examples:
        - user: "Yeah, fifty" (confirming 50? or correcting to 50?)
          strategy: "If the agent already said 50, treat as confirmation. If the agent said a different number, treat as correction."
        - user: "Mm-hmm" (weak confirmation)
          strategy: "For high-stakes actions, upgrade to explicit: 'I want to make sure — that's a yes to proceed?'"

  confirmation_anti_patterns:
    over_confirmation:
      problem: "Confirming every trivial field slows the interaction and irritates users"
      bad:
        agent: "You said London. Is that correct?"
        user: "Yes"
        agent: "And you said March 15th. Is that correct?"
        user: "Yes"
        agent: "And economy class. Is that correct?"
        user: "YES"
      good: "Use implicit confirmation for each field, then a single explicit summary at the end"

    confirmation_without_content:
      problem: "Asking 'Is that correct?' without stating what is being confirmed"
      bad: "Is that correct?"
      good: "500 dollars to savings. Is that correct?"
      rationale: "The user needs to hear the content to verify it. A bare 'is that correct?' after a pause forces the user to recall what was said."

    double_confirmation:
      problem: "Confirming the same information twice"
      bad:
        agent: "Boston to London. Correct?"
        user: "Yes"
        agent: "Great. So just to confirm, that's Boston to London?"
      good: "Confirm once, then move on."

    confirmation_as_interrogation:
      problem: "Rapid-fire confirmation feels like a police interview"
      bad: "Is the date correct? Is the amount correct? Is the account correct?"
      good: "Transfer 500 dollars from checking to savings on March 15th. All good?"
```

=== EXAMPLES ===

**Example 1: Low-Stakes — Implicit Grounding Throughout**
- User: "Set a timer for 10 minutes"
- Agent: "10-minute timer started."
- Analysis: The echo ("10-minute") implicitly confirms. No explicit check needed — the user will hear the timer and notice immediately if wrong. Zero extra turns.

**Example 2: High-Stakes — Graduated to Explicit**
- User: "Transfer money to my daughter's account"
- Agent: "Sure. How much?" (implicit grounding of intent)
- User: "Two hundred"
- Agent: "200 dollars. To your daughter Sarah's account ending in 4872?" (explicit — high stakes)
- User: "Yes"
- Agent: "Done. 200 dollars transferred to Sarah's account."
- Analysis: Implicit for the intent, explicit for amount + recipient. The agent adds identifying detail (name, last 4 digits) to prevent wrong-recipient errors.

**Example 3: ASR Ambiguity — Phonetic Echo**
- User: "Book under the name Nguyen"
- Agent: "Nguyen — N-G-U-Y-E-N?"
- User: "That's right"
- Agent: "Got it. And the first name?"
- Analysis: Names are high-error for ASR. Spelling confirmation is essential. Without it, the booking could end up under a completely different name.

**Example 4: Partial Disconfirmation Recovery**
- Agent: "So that's 3 nights at the Grand Hotel, checking in Friday, checking out Monday. Sound right?"
- User: "Sunday, not Monday"
- Agent: "Updated to Sunday checkout — that's 2 nights instead of 3. Everything else the same?"
- User: "Yes"
- Analysis: The agent corrects the field AND flags the downstream impact (2 nights, not 3). Then uses a lightweight "everything else the same?" rather than repeating the full summary.

**Example 5: Over-Confirmation Anti-Pattern Fix**
- Bad flow (6 turns for 3 fields):
  - Agent: "London?" User: "Yes" Agent: "March 15th?" User: "Yes" Agent: "Economy?" User: "Yes"
- Good flow (4 turns for 3 fields):
  - Agent: "London. March 15th. Economy. [pause] Shall I search?"
  - User: "Yes" (or corrects the one wrong field)
- Analysis: Batched implicit confirmation + single action prompt. Respects the user's time.

---

**Accessibility Requirements**: Grounding is inherently accessibility-friendly — it verifies understanding, which benefits all users. For users with hearing impairments, always echo critical values verbally (do not rely on display). For users with cognitive differences, prefer incremental grounding (one field at a time) over long summaries. For users with speech differences, allow extra time after confirmations and accept non-verbal confirmation (keypad press) as alternative.

**Interactional & Psychological Principles**: Common ground theory (Clark, 1996) — communication succeeds when participants maintain shared understanding. The "least collaborative effort" principle (Clark & Wilkes-Gibbs, 1986) — speakers minimize the total effort required to achieve grounding. Confirmation bias risk — once an agent states a value, users may accept it even if slightly wrong; use reformulation and digit-by-digit to break this bias. Trust calibration — explicit confirmation on high-stakes actions builds trust proportional to risk.
