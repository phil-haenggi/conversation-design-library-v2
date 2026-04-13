## Cross-modal: Voice-to-Screen Handoff v1.0

**Purpose**: Design seamless transitions between voice and visual modalities — handling cases where a task started in voice is better completed on screen, or where visual output supplements a voice interaction.

**Related**: `_conversational-ai/handoff-to-human.md`, `_voice-ux/no-input-no-match-escalation.md`, `_voice-ux/opening-closing-sequences.md`

---

**PROMPT:**

Design voice-to-screen handoff patterns for {{voice_agent}} where {{handoff_trigger}} requires transitioning from voice to a visual interface. Ensure context preservation, clear user communication, and minimal friction.

=== HANDOFF CONTEXT ===
- Trigger: {{handoff_trigger}} (complex_information/user_request/voice_failure/authentication/legal_requirement)
- Target modality: {{target}} (companion_app/sms_link/email/smart_display/web_browser)
- Context to transfer: {{context_fields}}
- Return to voice: {{return_expected}} (yes/no/optional)
- Platform: {{platform}}

=== OUTPUT REQUIREMENTS ===

```yaml
voice_to_screen_handoff:
  design_principles:
    why_handoff:
      principle: "Voice is excellent for hands-free interaction, quick queries, and linear tasks. But some tasks are poorly suited to voice: comparing multiple options, reviewing long lists, entering complex data (emails, addresses), viewing images or maps, or signing legal documents. Rather than forcing these into voice, gracefully hand off to a visual modality."
      key_insight: "A well-designed handoff is not a failure — it is the system being smart about choosing the right modality for the task at hand. Frame it as an upgrade, not a fallback."

    context_must_travel:
      principle: "Everything the user has said, every field already collected, and the current point in the flow must transfer to the visual interface. The user must NEVER re-enter information they already provided by voice."
      implementation: "Deep link to the exact point in the flow where the user left off, pre-populated with all collected data."

    announce_before_sending:
      principle: "Always tell the user what you are about to send and to which channel BEFORE sending it. Unexpected messages feel intrusive."

  handoff_patterns:
    proactive_agent_initiated:
      when: "The agent recognizes the task would be better served visually"
      triggers:
        - "Comparing 3+ options (flights, plans, products)"
        - "Reviewing terms and conditions or legal documents"
        - "Entering structured data (email addresses, physical addresses, credit card numbers)"
        - "Viewing maps, images, or charts"
        - "Long lists (5+ items)"
      pattern: "{{reason_for_handoff}} + {{what_you_will_send}} + {{where}} + {{user_consent}}"
      examples:
        - agent: "I found 5 flights that work. This might be easier to compare on screen. Can I send the options to your phone?"
          user: "Sure"
          agent: "Sent. You'll see all 5 flights with prices and times. Once you pick one, just let me know or tap to book in the app."
        - agent: "I'll need your email address for this. It might be easier to type that in. Want me to send you a link to enter it?"
          user: "Yeah, do that"
          agent: "Done — check your messages. The rest of the form is pre-filled with what you've told me."
        - agent: "There are some terms to review before I can activate this. I'll send them to your app so you can read through at your own pace. Sound good?"

    user_requested:
      when: "The user explicitly asks for visual output"
      triggers:
        - "Can you send me that?"
        - "I'd rather do this online"
        - "Can I see that on my phone?"
      pattern: "{{confirmation}} + {{action}} + {{what_to_expect}}"
      examples:
        - user: "Can you just send me a link?"
          agent: "Absolutely. I'll text you a link with everything pre-filled. You'll just need to confirm and pay."
        - user: "I want to see the options on screen"
          agent: "Sure. I'm sending the 3 options to your app now. They should pop up in a second."

    failure_driven:
      when: "Voice interaction has failed (repeated no-match, ASR difficulty, complex data entry)"
      pattern: "{{honest_framing}} + {{alternative_offer}} + {{context_preservation}}"
      examples:
        - agent: "I'm having trouble getting the spelling right. Let me text you a link where you can type it in — everything else is already filled in."
        - agent: "This part works better on screen. I'll send a link to your phone — you'll pick up right where we left off."
      design_note: "Frame the modality switch as practical, not as failure. 'This works better on screen' is better than 'I can't understand you.'"

  handoff_mechanics:
    pre_handoff_communication:
      required_elements:
        - what: "What you are sending (link, options, form, document)"
        - where: "Where it's going (text message, app notification, email)"
        - why: "Why the handoff helps (easier to compare, easier to type, required for legal reasons)"
        - continuity: "What happens next (they continue there, or they come back to voice)"
      example: "I'll text you a link with the 3 flight options. You can compare prices and times, and book right from there. If you'd rather come back to me, just say so."

    during_handoff:
      confirm_delivery:
        pattern: "{{delivery_confirmation}} + {{instruction}}"
        example: "Sent! You should see it in a moment. Let me know if it doesn't come through."
      handle_delivery_failure:
        pattern: "{{acknowledge}} + {{alternative}}"
        example: "Hmm, it doesn't look like it went through. Want me to try again, or would email work better?"

    post_handoff:
      if_voice_continues:
        pattern: "Continue the voice interaction with the remaining tasks"
        example: "Alright, the flight comparison is on your phone. While you look that over — did you also need a hotel?"
      if_voice_ends:
        pattern: "{{summary}} + {{where_to_continue}} + {{closing}}"
        example: "I've sent everything to your app. You can finish booking there. If you need anything else, just call back. Have a good day!"
      if_user_returns_from_screen:
        pattern: "{{welcome_back}} + {{status_check}}"
        example: "Welcome back. Did you find what you needed on screen, or would you like to continue here?"

  screen_to_voice_handoff:
    when: "The user transitions from a visual interface to voice mid-task"
    pattern: "{{context_restoration}} + {{continuation}}"
    examples:
      - agent: "I can see you started a booking on the app — a flight to London on March 15th. Want me to finish that up for you?"
      - agent: "Looks like you were comparing hotel options. Want me to walk you through the top choices?"
    critical: "Same principle — context must travel. The user should never have to repeat what they already entered on screen."

  companion_screen_patterns:
    smart_display:
      principle: "When a smart display is available, use it as a complement to voice — not a replacement. Voice handles the interaction; the screen shows supporting visuals."
      examples:
        - "Voice: 'Here are your three flight options.' Screen: [shows comparison table with prices, times, airlines]"
        - "Voice: 'Your package is on its way.' Screen: [shows tracking map]"
      design_rule: "The voice interaction must be fully functional without the screen. The screen adds value but must never be required — the user may not be looking at it."

    phone_screen_during_call:
      principle: "If the user is on a phone call, they may or may not be able to look at their screen simultaneously"
      approach: "Send information via text/push notification that the user can review AFTER the call, or explicitly ask 'Can you check your screen right now?'"

  handoff_anti_patterns:
    context_loss:
      bad: "I'll send you a link to start the booking process."
      problem: "User already provided destination, dates, passengers — 'start' implies they have to redo everything."
      good: "I'll send a link with your London trip details pre-filled. You just need to choose a flight and pay."

    unexplained_handoff:
      bad: "[sends text message without warning]"
      problem: "User didn't consent and doesn't know what was sent or why."
      good: "I'm going to text you a link with the options. Sound okay?"

    forced_handoff:
      bad: "You need to complete this step in our app. Please go to..."
      problem: "The user called because they want voice interaction. Forcing them to an app feels like a runaround."
      good: "This step works best in the app, but I can also walk you through it here if you prefer. What would you like?"

    dead_end_handoff:
      bad: "I've sent you a link. Goodbye."
      problem: "No confirmation of delivery, no instructions, no option to continue by voice."
      good: "I've sent you a link. Let me know if it arrives. And if you'd rather finish this by voice, I'm happy to do that too."
```

=== EXAMPLES ===

**Example 1: Flight Comparison — Voice to App**
- Agent: "I found 5 flights to London on March 15th. Prices range from 320 to 680 dollars. Rather than reading all 5, want me to send the comparison to your phone? It's easier to see side by side."
- User: "Yes please"
- Agent: "Sent to your TeleCo app. You'll see all 5 with times, stops, and prices. Tap any one to book, or call me back if you want help choosing."
- Analysis: Clear reason (easier to compare), consent obtained, sent to specified channel, instructions for next steps, and an exit back to voice.

**Example 2: Data Entry Failure — Graceful Handoff**
- Agent: "What's your email address?"
- User: "j-dot-smith-at-g-mail..."
- Agent: "I'm getting j-smith at gmail. Let me send you a quick form to type it in — everything else is already filled out."
- User: "Okay"
- Agent: "Texted. Just enter your email and hit submit — the rest is done."
- Analysis: The agent's ASR failed on the email. Rather than three rounds of "was that j or g?", a modality switch solves it in one move. Framed as practical, not as failure.

**Example 3: Legal Requirement — Terms Review**
- Agent: "Before I activate the new plan, there are some terms to review. I'm required to have you read through them. I'll send them to your app — take your time, and when you accept, I'll activate the plan right away."
- User: "Fine"
- Agent: "Sent. Once you accept in the app, your new plan will be active immediately. Anything else while you're here?"
- Analysis: Legal requirement makes voice-only impossible. Agent explains why, sends to app, and continues the voice interaction while the user reviews.

**Example 4: Screen-to-Voice Return**
- [User started booking on app, then calls in]
- Agent: "Hi, I can see you were looking at flights to London in the app. Want me to help finish the booking?"
- User: "Yeah, I couldn't decide between the two morning flights"
- Agent: "The 9am direct is 450 dollars and gets you there at 9pm. The 11am with a stop is 320 and arrives at 11pm. The direct one saves you 4 hours. Want me to book the 9am?"
- Analysis: Full context from app is available. Agent provides the comparison the user struggled with visually, adds a helpful framing ("saves you 4 hours"), and offers a recommendation.

---

**Accessibility Requirements**: Handoff links must be accessible (WCAG compliant destination pages). SMS links must work with screen readers. Always offer to continue by voice as an alternative — some users cannot use screen interfaces. Ensure pre-filled forms are labeled correctly for assistive technology. Never force a modality switch on users who depend on voice due to visual or motor impairments.

**Interactional & Psychological Principles**: Channel continuity (Rosen & Salomon, 2007) — context preservation across channels is the single biggest factor in cross-channel satisfaction. Effort justification — users who have already invested effort in a voice interaction will abandon if forced to restart on screen. Choice architecture (Thaler & Sunstein, 2008) — offering the handoff as an option (not a requirement) respects autonomy. Peak-end rule — a smooth handoff at the end of a voice interaction lifts overall experience ratings.
