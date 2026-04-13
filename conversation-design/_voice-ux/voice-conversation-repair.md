## Voice Conversation Repair v1.0

**Purpose**: Design repair sequences for voice interactions that resolve misunderstandings efficiently using a graduated scale — from weaker, less disruptive repair initiators to stronger, more explicit ones — grounded in the organization of repair in conversation.

**Related**: `_conversational-ai/conversational-repair.md`, `_conversational-ai/error-recovery-flows.md`, `_conversational-ai/clarifying-questions.md`, `_voice-ux/turn-design-progressivity.md`

---

**PROMPT:**

Design voice-specific conversation repair strategies for {{voice_agent}} handling {{domain}} interactions. Apply the CA repair hierarchy (open class → restricted → partial repeat → understanding check) and address voice-specific challenges (ASR errors, ambient noise, homophones, prosodic ambiguity).

=== REPAIR CONTEXT ===
- Trouble source: {{trouble_source}} (asr_error/ambiguity/out_of_domain/user_unclear/system_confusion)
- Repair initiation: {{initiator}} (agent/user/system)
- Conversation stage: {{stage}} (opening/information_gathering/confirmation/closing)
- Error frequency: {{frequency}} (first_occurrence/repeated)
- Domain sensitivity: {{sensitivity}} (low/medium/high — e.g., medical, financial)

=== OUTPUT REQUIREMENTS ===

```yaml
voice_repair:
  ca_foundation:
    repair_organization:
      definition: "Repair is the set of practices by which participants in conversation address and resolve problems of speaking, hearing, and understanding. It is organized along two dimensions: who initiates the repair (self vs. other) and who carries it out (self vs. other)."
      preference_structure: "In ordinary conversation, self-initiated self-repair is preferred over other-initiated repair. However, in voice UX, the agent frequently needs to initiate repair on the user's talk (other-initiation) because of ASR limitations. The design challenge is to do this as non-disruptively as possible."
      key_principle: "Use the weakest repair initiator that will get the job done. Stronger repair initiators are more disruptive to progressivity and should only be escalated to when weaker forms fail."
      reference: "Schegloff, Jefferson & Sacks (1977) — The Preference for Self-Correction in the Organization of Repair in Conversation"

  agent_initiated_repair_hierarchy:
    overview: "The following hierarchy moves from weakest (least disruptive) to strongest (most disruptive) forms of agent-initiated repair. Always start at the weakest appropriate level and escalate only if needed."

    level_1_open_class:
      strength: "weakest"
      definition: "A general indicator that something in the prior turn was problematic, without specifying what. Gives the user maximum latitude to self-repair."
      tokens:
        - "Sorry?"
        - "Hm?"
        - "Pardon?"
        - "Come again?"
        - "Say that again?"
        - "I didn't quite catch that."
        - "Sorry, one more time?"
      when_to_use:
        - "First occurrence of ASR failure (no confident transcription)"
        - "Entire utterance was unclear (noise, mumbling, too quiet)"
        - "You have no specific element to query"
      example:
        user: "[inaudible or garbled input]"
        agent: "Sorry, I didn't catch that. Could you say that again?"
      design_note: "Open class repair is the gentlest but also the least efficient — the user must repeat everything. Use it only when you genuinely have no idea what was said."

    level_2_question_word_plus_partial_repeat:
      strength: "moderate"
      definition: "A question word (what, which, where, when) combined with a partial repeat of the trouble-source turn. This locates the problem for the user — they know which part to repair — without suggesting a candidate answer."
      pattern: "{{question_word}} + {{element_of_prior_turn}}"
      tokens:
        - "What do you mean {{element}}?"
        - "Which {{category}} was that?"
        - "What was the {{field}} again?"
        - "Where did you say?"
        - "When was that?"
      examples:
        - user: "I want to change my phone number thing"
          agent: "What do you mean phone number? Change it on your account, or update the one we have for you?"
          analysis: "'What do you mean phone number?' locates the ambiguity (is 'phone number thing' about changing a device or updating contact info) and offers two candidate resolutions."

        - user: "Add 100GB to my plan"
          agent: "What do you mean 100GB? Your current plan has a 50GB cap — did you want to add 100GB on top of that, or switch to the 100GB plan?"
          analysis: "'What do you mean 100GB?' isolates the problematic element. The agent then provides specific candidate interpretations grounded in the user's account context."

        - user: "Move it to the other one"
          agent: "Which other one — your savings account or your investment account?"
          analysis: "Partial repeat ('the other one') + question word ('which') + candidate options."

        - user: "I was charged twice on the fifteenth...or the sixteenth...for the thing"
          agent: "What was the charge for?"
          analysis: "The agent identifies the most consequential gap ('the thing') and targets that, rather than asking about the ambiguous date which matters less for locating the transaction."

      when_to_use:
        - "You understood most of the utterance but one element is unclear"
        - "The user used a vague or ambiguous term"
        - "ASR returned a partial transcription"

    level_3_candidate_understanding_check:
      strength: "strong"
      definition: "The agent offers a specific candidate interpretation of what the user meant and asks them to confirm or deny it. This is more presumptive than Level 2 — it proposes an answer rather than asking an open question."
      pattern: "You mean {{candidate_interpretation}}?"
      tokens:
        - "You mean {{candidate}}?"
        - "So you're saying {{interpretation}}?"
        - "Just to check — you want to {{action}}?"
        - "Did you mean {{option_A}} or {{option_B}}?"
        - "So that's {{value}}, is that right?"
      examples:
        - user: "Turn off the roaming"
          agent: "You mean the data roaming, whether it's on or off?"
          analysis: "The agent formulates a specific understanding check. 'Whether it's on or off' shows the agent is confirming the scope of the request (toggle data roaming) rather than acting on a potentially wrong interpretation."

        - user: "Cancel it"
          agent: "You mean cancel the order you just placed?"
          analysis: "Candidate understanding check grounded in conversational context (recent order). Much more efficient than open class ('Cancel what?')."

        - user: "I said the blue one"
          agent: "The blue Samsung Galaxy, right?"
          analysis: "Upgrades 'the blue one' to a specific product reference and checks it."

        - user: "Put fifty on there"
          agent: "So that's fifty dollars to your prepaid balance?"
          analysis: "Resolves 'there' to a specific account and 'fifty' to a currency amount."

      when_to_use:
        - "You have a strong hypothesis about what the user meant"
        - "The user's utterance is ambiguous but you can narrow it based on context"
        - "Confirming a specific interpretation is faster than asking an open question"
        - "High-stakes domain where acting on wrong interpretation would be costly"

    level_4_explicit_correction_request:
      strength: "strongest"
      definition: "Direct request for the user to provide specific information, often after previous repair attempts have failed. This is the most disruptive to progressivity but may be necessary."
      pattern: "I need you to {{specific_request}}."
      tokens:
        - "Could you spell that for me?"
        - "Can you give me the exact {{field}}?"
        - "Let me try once more — what was the {{specific_element}}?"
        - "I want to make sure I get this right. What is the {{field}} exactly?"
      examples:
        - agent: "Could you spell the city name for me? I want to make sure I've got the right one."
        - agent: "Can you give me the account number digit by digit?"
        - agent: "I want to make sure I get this right. What's the name on the reservation?"
      when_to_use:
        - "Previous repair attempts at lower levels have failed"
        - "ASR keeps misrecognizing a proper name or number"
        - "The information is critical (financial, medical, legal)"
        - "Homophones or similar-sounding options make confirmation unreliable"

  repair_escalation_protocol:
    principle: "Start weak, escalate only as needed. Each failed repair attempt should move one level up the hierarchy."
    example_sequence:
      turn_1:
        user: "[unclear input about account]"
        agent: "Sorry, I didn't quite catch that."
        level: "Level 1 — open class"
      turn_2:
        user: "I said change the thing on my account"
        agent: "What thing on your account — your password, your email, or something else?"
        level: "Level 2 — question word + partial repeat + options"
      turn_3:
        user: "The email"
        agent: "You want to update the email address on your account?"
        level: "Level 3 — candidate understanding check"
      turn_4:
        user: "Yes"
        agent: "Got it. What's the new email address?"
        resolution: "Repair complete, progressivity resumed"

    max_repair_attempts:
      guideline: "After 3 failed repair attempts on the same trouble source, pivot strategy"
      options:
        - "Offer to try a different input modality: 'Would it be easier to type that in? I can send you a link.'"
        - "Offer to transfer: 'Let me connect you to someone who can help with this.'"
        - "Offer to skip and return: 'Let's come back to that. What about {{next_field}}?'"

  user_initiated_repair:
    user_corrects_agent:
      principle: "Accept corrections immediately, confirm the corrected version, and move on. Do not over-apologize or explain why the error occurred."
      patterns:
        - user: "No, I said Boston, not Austin"
          agent: "Boston, got it. [continues]"
          analysis: "Minimal receipt of correction + immediate resume. No 'I'm sorry I misheard' — that stalls progressivity."
        - user: "Not the savings account, the checking account"
          agent: "Checking account. I'll transfer from there."
          analysis: "Echo-corrects and moves forward."
      avoid:
        - "I apologize for the confusion, I thought you said Austin. Let me correct that to Boston. So you wanted to fly to Boston, is that correct?"
          problem: "Three turns of repair work for what should be a one-turn correction. Violates progressivity."

    user_requests_repeat:
      principle: "Repeat the relevant information concisely — do not re-deliver the entire prior turn."
      patterns:
        - user: "Wait, how much was that?"
          agent: "450 dollars."
          analysis: "Answers the specific repeat request without re-stating the entire turn."
        - user: "Sorry, say that again?"
          agent: "Your flight arrives at 3:15pm at Terminal B."
          analysis: "Repeats the core information, slightly slower. Does not say 'As I was saying...'"
        - user: "What were those times again?"
          agent: "9am, 11am, and 2pm."
          analysis: "Just the requested data."

  voice_specific_repair_challenges:
    homophones_and_near_homophones:
      problem: "Words that sound the same or similar cause frequent ASR errors"
      strategy: "Use disambiguating context or ask for spelling"
      examples:
        - pair: "15 / 50 (fifteen / fifty)"
          repair: "Did you say fifteen, one-five, or fifty, five-zero?"
        - pair: "B / D / E / G / P / T / V (letter confusion)"
          repair: "Was that B as in Bravo, or D as in Delta?"
        - pair: "gray / grey / great"
          repair: "Can you spell that for me?"

    background_noise_repair:
      principle: "When noise is detected, address it directly rather than repeatedly asking the user to repeat"
      patterns:
        - "I'm having a little trouble hearing you — it sounds like there's some background noise. Could you speak up a bit or move somewhere quieter?"
        - "The connection seems a bit noisy. Let me try to work with what I've got — did you say {{best_guess}}?"

    prosodic_ambiguity:
      problem: "In voice, the same words with different intonation can mean different things"
      examples:
        - utterance: "Tuesday?" (rising = question) vs "Tuesday." (falling = statement)
          strategy: "When prosody is ambiguous, treat as the more common interpretation in context, but confirm"
        - utterance: "Fine." (could be satisfied or resigned)
          strategy: "In high-stakes contexts, check: 'Just to make sure — you're happy with that option?'"
```

=== EXAMPLES ===

**Example 1: Graduated Repair — Telecom Account Change**
- User: "I want to change the thingy"
- Agent [L1]: "Sorry, what was that?"
- User: "The thingy, you know, the data thing"
- Agent [L2]: "What do you mean — your data plan or your data roaming setting?"
- User: "The roaming"
- Agent [L3]: "You mean the data roaming, whether it's on or off?"
- User: "Yeah, turn it off"
- Agent: "Done — data roaming is now off. Anything else?"
- Analysis: L1 (open class) → L2 (question word + partial repeat) → L3 (candidate understanding check) → resolution. Three repair turns to go from "the thingy" to a specific action. Each level added specificity.

**Example 2: Immediate Level 3 — High Confidence Context**
- Context: User has been discussing their recent order
- User: "Cancel it"
- Agent: "You mean cancel order number 7-8-4-2 that you placed yesterday?"
- User: "Yes"
- Agent: "Cancelled. You'll get a refund within 3-5 business days."
- Analysis: The agent skips Levels 1 and 2 because context provides a strong candidate. Going straight to L3 is efficient and respects progressivity.

**Example 3: User-Initiated Repair — Correction**
- Agent: "I've set that up for Austin, Texas."
- User: "No, Boston."
- Agent: "Boston — updated. And was that a one-way trip?"
- Analysis: Minimal correction receipt + immediate resume. One turn, not three.

**Example 4: Number Disambiguation**
- User: "Transfer fifty to savings"
- Agent: "Fifty dollars — that's five-zero — to your savings account?"
- User: "No, fifteen"
- Agent: "Fifteen, one-five. Got it — transferring 15 dollars to savings now."
- Analysis: Number disambiguation using digit-by-digit confirmation. When corrected, the agent uses the same technique to confirm the corrected number.

**Example 5: Repair Failure → Pivot**
- Agent: "What's the name on the reservation?"
- User: "[unclear name]"
- Agent: "Sorry, I didn't catch that. Could you say the name again?"
- User: "[still unclear]"
- Agent: "I'm still having trouble with the name. Could you spell it for me?"
- User: "S-C-H-W-A-R-T-Z"
- Agent: "Schwartz. Got it."
- Analysis: L1 (open class) → L4 (explicit correction with modality shift to spelling). The agent skips L2/L3 because the problem is auditory, not semantic.

---

**Accessibility Requirements**: Repair sequences must be patient and non-judgmental. Users with speech impairments, accents, or non-native pronunciation may trigger more ASR errors — the repair system must never make them feel at fault. Offer alternative input modalities (spelling, keypad, text fallback) early for users who struggle with voice recognition. Never use blame-attributing language ("I can't understand you"). Always frame as the system's limitation ("I'm having trouble with that").

**Psychological Principles**: The repair preference hierarchy (Schegloff, Jefferson & Sacks, 1977) reflects a universal preference for minimal disruption. Face-threat mitigation (Brown & Levinson, 1987) — stronger repair forms are more face-threatening to the user (implying their talk was deficient), which is why we start weak. The "progressivity bias" (Stivers & Robinson, 2006) — participants prefer to move forward; repair is tolerated only because it serves eventual understanding. Response design (Heritage, 1984) — the form of the repair initiator shapes the form of the repair itself, so choosing the right level directly affects repair efficiency.
