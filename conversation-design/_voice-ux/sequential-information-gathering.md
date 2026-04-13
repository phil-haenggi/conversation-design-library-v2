## Sequential Information Gathering v1.0

**Purpose**: Gather required information from users one question at a time in a natural, conversational sequence — avoiding the "form-filling" anti-pattern where multiple fields are requested simultaneously.

**Related**: `_conversational-ai/clarifying-questions.md`, `_conversational-ai/disambiguation-prompts.md`, `_voice-ux/turn-design-progressivity.md`

---

**PROMPT:**

Design sequential information-gathering flows for {{voice_agent}} collecting {{data_points}} from users via voice for {{use_case}}.

=== GATHERING CONTEXT ===
- Required fields: {{required_fields}}
- Optional fields: {{optional_fields}}
- Field dependencies: {{dependencies}} (e.g., return_date depends on trip_type)
- Domain: {{domain}}
- User patience profile: {{patience}} (task_focused/conversational/time_pressured)

=== OUTPUT REQUIREMENTS ===

```yaml
sequential_gathering:
  core_principle:
    one_question_at_a_time:
      rationale: "In natural conversation, speakers ask one question per turn and wait for the answer before asking the next. This is not merely a UX heuristic — it reflects a fundamental structural property of talk-in-interaction. Adjacency pairs (question-answer) are the basic building block of interaction. Stacking multiple questions violates this structure and creates trouble for the respondent: which question to answer first? Did I forget one?"
      ca_evidence: "Sacks, Schegloff & Jefferson (1974) demonstrated that conversation operates on a one-at-a-time turn-taking system. When multiple questions are asked in a single turn, respondents overwhelmingly answer only the last one (the one freshest in memory — see turn-final-placement.md)."

  question_sequencing:
    ordering_principles:
      natural_narrative_order:
        principle: "Sequence questions in the order a user would naturally think about the task"
        example:
          domain: "travel_booking"
          natural_order: ["destination", "origin", "date", "passengers", "class"]
          rationale: "People think 'I want to go to London' before 'I'm leaving from New York' — destination is the motivating desire"
        anti_pattern:
          order: ["passenger_count", "class", "origin", "destination", "date"]
          problem: "Bureaucratic order that follows internal data schema, not user mental model"

      dependency_aware:
        principle: "Ask dependent questions only after their prerequisites are resolved"
        example:
          - ask: "One-way or round trip?"
            then_if_round_trip: "And when would you like to come back?"
            then_if_one_way: "[skip return date, move to next field]"
          - ask: "Is this for yourself or someone else?"
            then_if_someone_else: "What's their name?"
            then_if_self: "[skip, use known name]"

      easy_first:
        principle: "Start with low-cognitive-load questions to build momentum and rapport before asking harder ones"
        example:
          easy_start: "What city are you heading to?"
          harder_later: "Do you have any airline preference, or should I find the best price?"

      leverage_prior_answers:
        principle: "Use information from previous answers to narrow subsequent questions or skip them entirely"
        example:
          - user_said: "I need to fly to London next Friday"
            extracted: ["destination: London", "date: next Friday"]
            skip: "destination question, date question"
            next_ask: "And where are you flying from?"

  acknowledgment_bridging:
    principle: "Each new question should be bridged with a brief acknowledgment of the previous answer. This creates conversational coherence and signals that the information was received."
    patterns:
      minimal_receipt:
        tokens: ["Got it.", "Okay.", "Right.", "Sure."]
        usage: "routine, low-stakes fields"
        example: "Got it. And what date?"

      echo_confirmation:
        pattern: "{{echoed_value}}. {{next_question}}"
        usage: "when the value is important or ambiguous"
        example: "London Heathrow. And when are you looking to fly?"

      implicit_confirmation:
        pattern: "{{reframed_value_in_next_question}}"
        usage: "to advance without adding an extra turn"
        example:
          user: "March 15th"
          agent: "For your March 15th flight — morning or afternoon departure?"
          analysis: "The date is woven into the next question, confirming receipt without a separate confirmation turn"

      appreciative_receipt:
        tokens: ["Great.", "Perfect.", "Thanks."]
        usage: "sparingly — overuse feels performative. Best for final field or complex input."
        example: "Perfect, that's everything I need. Let me search for flights."

  handling_user_volunteered_information:
    principle: "When users provide multiple pieces of information unprompted, acknowledge all of them and skip the corresponding questions"
    example:
      user: "I need a round trip from Boston to Chicago, leaving Friday, back Sunday"
      agent: "Got it — round trip, Boston to Chicago, Friday to Sunday. How many passengers?"
      analysis: "The agent extracts all volunteered slots, confirms them in a brief summary, and jumps to the first un-filled slot"
    anti_pattern:
      agent: "What city are you flying to?"
      problem: "Ignores everything the user just said. Forces the user back to the beginning. Violates progressivity."

  pacing_and_rhythm:
    between_questions:
      principle: "Allow a micro-pause (~300-500ms) between the acknowledgment and the next question. This gives the user a beat to process."
      pattern: "{{acknowledgment}}. [300ms pause] {{next_question}}"

    dont_rush:
      principle: "Avoid machine-gunning questions. Even if the system can process instantly, maintain a natural conversational pace."
      bad: "London. Date? March 15th. Passengers? One. Class?"
      good: "London, great. [pause] And what date works for you?"

    signal_progress:
      principle: "For longer gathering sequences (5+ fields), periodically signal progress to prevent user fatigue"
      example: "Almost there — just two more things. Do you have a seat preference?"

  error_recovery_in_sequence:
    misheard_value:
      principle: "Correct the specific field without restarting the sequence"
      example:
        agent: "March 15th. And how many passengers?"
        user: "Wait, I said March 5th, not 15th"
        agent: "Sorry about that — March 5th. So, how many passengers?"
        analysis: "Corrects the field, then resumes from where the sequence left off"

    user_changes_mind:
      principle: "Accept the change, update the slot, and continue forward"
      example:
        agent: "One-way to London on March 5th. How many passengers?"
        user: "Actually, make it round trip"
        agent: "Sure, round trip. When would you like to come back?"
        analysis: "The change introduces a new dependency (return date), which the agent handles before continuing"

  completion_and_summary:
    principle: "After gathering all required fields, provide a complete summary and request confirmation before acting"
    pattern: "Alright, let me confirm: {{summary_of_all_fields}}. Should I go ahead?"
    example: "Alright, let me confirm: round trip from Boston to London, March 5th to 12th, one passenger, economy. Should I search for flights?"
    rationale: "This is the repair 'last chance' — a pre-sequence to the action that lets the user catch any errors before commitment"
```

=== EXAMPLES ===

**Example 1: Restaurant Reservation**
- Agent: "I'll help you make a reservation. What restaurant?"
- User: "Chez Louis"
- Agent: "Chez Louis, nice choice. What date?"
- User: "This Saturday"
- Agent: "Saturday. And what time?"
- User: "7:30"
- Agent: "Got it. For how many people?"
- User: "Four"
- Agent: "Alright — Chez Louis, this Saturday at 7:30 for four. Let me check availability."
- Analysis: Six turns (3 agent, 3 user adjacency pairs) + summary. Natural narrative order. Minimal acknowledgments keep pace brisk.

**Example 2: User Volunteers Multiple Fields**
- User: "I need a table for two at that Italian place on Main Street, Friday night"
- Agent: "Got it — the Italian place on Main Street, Friday evening for two. Any time preference, or should I take the first available?"
- Analysis: Agent acknowledges all volunteered info, skips to the one remaining gap. No redundant questions.

**Example 3: Insurance Claim (Longer Sequence with Progress Signal)**
- Agent: "I'll help you file a claim. First, what type of claim is this — auto, home, or other?"
- User: "Auto"
- Agent: "Auto claim. When did the incident happen?"
- User: "Yesterday morning"
- Agent: "Okay. Can you briefly describe what happened?"
- User: "Someone rear-ended me at a red light"
- Agent: "Got it, a rear-end collision. Were there any injuries?"
- User: "No, everyone's fine"
- Agent: "Glad to hear that. Just a couple more details — do you have the other driver's insurance information?"
- Analysis: "Just a couple more details" signals progress. Empathetic response ("Glad to hear that") is appropriate for the domain without derailing progressivity.

---

**Accessibility Requirements**: One question at a time is inherently more accessible — it reduces cognitive load, supports users with working memory differences, and allows screen reader / voice-only users to process each field independently. Never bundle multiple questions. Always allow the user to go back and correct a previous answer without restarting.

**Psychological Principles**: Adjacency pair structure (Schegloff & Sacks, 1973) is the fundamental unit of conversational interaction. Sequential gathering mirrors the "stepwise progression" observed in institutional talk (e.g., medical intake, service encounters). The Zeigarnik effect means users are motivated to complete a sequence once started — progress signals reinforce this. Cognitive load theory (Sweller) supports one-item-at-a-time processing in the auditory channel.
