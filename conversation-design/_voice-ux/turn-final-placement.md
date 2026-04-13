## Turn-Final Placement & End Focus Principle v1.0

**Purpose**: Place the most important or actionable element at the end of each agent turn — the position with the highest perceptual salience and recall in spoken interaction.

**Related**: `_voice-ux/turn-design-progressivity.md`, `_voice-ux/sequential-information-gathering.md`, `_conversational-ai/clarifying-questions.md`

---

**PROMPT:**

Apply the end focus principle to voice turn design for {{voice_agent}} handling {{use_case}}. Ensure that questions, key information, and action items are placed at the turn-final position for maximum salience.

=== PLACEMENT CONTEXT ===
- Turn type: {{turn_type}} (question/information_delivery/confirmation/action_report)
- Cognitive load: {{load}} (low/medium/high)
- Interface: {{interface}} (smart_speaker/phone_ivr/in_car/wearable)
- User state: {{user_state}} (attentive/distracted/multi_tasking)

=== OUTPUT REQUIREMENTS ===

```yaml
turn_final_placement:
  core_principle:
    end_focus:
      definition: "The turn-final position — the last element the user hears before it becomes their turn to speak — carries the highest perceptual weight in spoken interaction. This is where the listener's attention peaks, and it is the element most likely to be retained in working memory and acted upon."
      linguistic_basis: "The end focus principle (also: end-weight) is well-documented in functional linguistics (Halliday, 1967) and information structure theory. In English, new and important information gravitates toward the end of the clause. In conversation, this interacts with the turn-taking system: the turn-final element is what the next speaker orients to."
      ca_evidence: "When multiple questions are asked in a single turn, respondents overwhelmingly address the last one. This is not because they are ignoring the earlier questions — it is because the turn-final question is the one that creates the strongest sequential implicativeness (i.e., it is the one that most clearly 'demands' a response at the transition relevance place)."

  application_by_turn_type:
    questions:
      rule: "Always place the question at the very end of the turn"
      rationale: "The question is the element that makes a response conditionally relevant. Placing it last ensures (a) the user knows exactly what to answer, and (b) the question is freshest in memory."
      patterns:
        good:
          - turn: "I found three options for you. Would you like the cheapest or the fastest?"
            analysis: "Context first ('I found three options'), question last. The user hears the question right before their turn begins."
          - turn: "Your appointment is confirmed for Tuesday. Would you like a reminder?"
            analysis: "Information delivery first, question last."
          - turn: "Got it, economy class. How many passengers?"
            analysis: "Acknowledgment first, question last."
        bad:
          - turn: "Would you like the cheapest or the fastest? I found three options for you."
            problem: "Question is buried before the context. By the time the context ends, the user may have forgotten the options or started formulating an answer to a question that is no longer the last thing they heard."
          - turn: "How many passengers? And I should mention, there's a group discount for 4 or more."
            problem: "The question comes first, then additional information follows. The user doesn't know whether to answer or keep listening."

    information_delivery:
      rule: "Place the most critical or novel piece of information at the end"
      rationale: "Recency effect — the last item in a spoken sequence is recalled best."
      patterns:
        good:
          - turn: "For your March 5th trip, the total comes to 847 dollars."
            analysis: "Context frame first ('for your March 5th trip'), key information last ('847 dollars')."
          - turn: "The package has been shipped and should arrive by Thursday."
            analysis: "Action completed first, the information the user most cares about (arrival date) last."
        bad:
          - turn: "847 dollars is the total for your March 5th trip."
            problem: "The price — which the user needs to evaluate — is at the beginning, followed by context they already know."

    confirmations:
      rule: "State the information being confirmed, then place the confirmation request last"
      pattern: "{{information_summary}}. {{confirmation_question}}"
      good:
        - "Round trip to London, March 5th to 12th, economy. Does that sound right?"
        - "So you'd like to cancel the premium subscription. Is that correct?"
      bad:
        - "Is that correct — round trip to London, March 5th to 12th, economy?"
        - "Can you confirm? The order is 3 items totaling 45 dollars shipping to your home address."
        problem: "Placing the confirmation question before the information forces the user to hold the question in mind while processing the details."

    action_reports:
      rule: "Place the outcome or next step at the end"
      pattern: "{{action_taken}}. {{outcome_or_next_step}}"
      good:
        - "I've submitted your claim. You'll get a reference number by email within the hour."
        - "Your password has been reset. You can now log in with your new password."
      bad:
        - "You'll get a reference number by email. I've submitted your claim."
        problem: "The future action comes before the user knows the current action is complete — creates momentary confusion."

  compound_turns:
    rule: "When a turn must contain both information and a question, always sequence: information first, question last"
    rationale: "The question at the end serves as a clear transition relevance place (TRP) that signals 'your turn now.' If the question is followed by more information, the user doesn't know when to speak."
    pattern: "{{information}}. [micro-pause] {{question}}"
    examples:
      good:
        - "The morning flight is 450 dollars and the evening one is 380. Which would you prefer?"
        - "I see you have two accounts on file. Which one should I use for this payment?"
      bad:
        - "Which would you prefer? The morning flight is 450 and the evening is 380."
        - "Which account should I use? You have a checking and a savings on file."

  special_cases:
    lists_with_questions:
      principle: "Present list items first, then ask the question about them"
      good: "I have appointments available at 9, 11, and 2. Which works best for you?"
      bad: "Which time works for you? I have 9, 11, and 2 available."
      rationale: "The user needs to hear the options before they can make a choice."

    corrections_with_continuation:
      principle: "Correct the error first, then place the next question last"
      good: "Sorry, March 5th, not 15th. And how many passengers?"
      bad: "How many passengers? Oh, and I've corrected that to March 5th."

    caveats_and_qualifiers:
      principle: "Place caveats before the main information, not after"
      good: "Just so you know, this fare is non-refundable. The total is 450 dollars."
      bad: "The total is 450 dollars. Oh, and just so you know, it's non-refundable."
      rationale: "Placing the caveat after the price feels like a 'gotcha.' Placing it before frames the price within its conditions."

  prosodic_support:
    principle: "Use falling intonation on the turn-final element for statements (signaling completion) and rising intonation for yes/no questions (signaling that a response is expected)"
    ssml_examples:
      statement_ending:
        text: "Your balance is three hundred and forty-two dollars."
        ssml: "Your balance is <prosody pitch='-5%'>three hundred and forty-two dollars.</prosody>"
      question_ending:
        text: "Would you like to proceed?"
        ssml: "Would you like to <prosody pitch='+10%'>proceed?</prosody>"
    note: "Prosodic design reinforces the structural placement — the user hears both the position and the intonation as signals for when to respond."
```

=== EXAMPLES ===

**Example 1: Banking — Balance + Offer**
- Bad: "Would you like to make a transfer? Your balance is 2,341 dollars."
- Good: "Your balance is 2,341 dollars. Would you like to make a transfer?"
- Why: The question at the end creates a clear response point. The user has the information they need (balance) before being asked to act on it.

**Example 2: Healthcare — Appointment Options**
- Bad: "Which one works for you? Dr. Smith has openings Monday at 10, Wednesday at 2, and Friday at 9."
- Good: "Dr. Smith has openings Monday at 10, Wednesday at 2, and Friday at 9. Which one works for you?"
- Why: The user needs to hear the options before choosing. Turn-final question leverages recency.

**Example 3: Retail — Order Confirmation**
- Bad: "Is that right? Two pairs of running shoes, size 10, in black, shipping to your home address."
- Good: "Two pairs of running shoes, size 10, in black, shipping to your home address. Is that right?"
- Why: The user processes the summary and is immediately prompted for confirmation while all details are fresh.

**Example 4: Complex Turn — Information + Caveat + Question**
- Bad: "Do you want to add insurance? It costs 35 dollars and is non-refundable. Your rental is confirmed for Saturday."
- Good: "Your rental is confirmed for Saturday. You can add insurance for 35 dollars — it's non-refundable though. Want to add it?"
- Why: Logical flow: confirmation → offer → caveat → question. Each element builds on the previous, and the question comes last.

---

**Accessibility Requirements**: Turn-final placement is especially critical for users with auditory processing differences, limited working memory, or attention difficulties. Consistent placement of questions at turn-end creates a predictable rhythm that reduces cognitive effort. Avoid burying actionable elements in the middle of turns.

**Psychological Principles**: Recency effect (Murdock, 1962) — the last item in a sequence is recalled most accurately. End-weight principle (Quirk et al., 1985) — longer and more complex elements naturally gravitate to sentence-final position in English. Sequential implicativeness (Schegloff, 2007) — the turn-final element sets up the strongest expectation for what comes next. Transition relevance places (Sacks, Schegloff & Jefferson, 1974) — speakers orient to turn-final elements as the signal for speaker change.
