---
version: "2.1"
date: 2026-04-14
---

# Conversation Design Prompt Library

> **Production-ready prompt templates** for designing human-centered conversational AI experiences and accessible, localized content.

## Purpose

This library provides prompt templates for three core disciplines in conversation design:

- **Conversational AI** — Dialog patterns for chatbots, agents, and various other conversational user interfaces
- **Voice UX** — Guidelines for voice-first and voice-only conversational interfaces, grounded in interactional principles from Conversation Analysis and Linguistics
- **Accessibility & Localization** — Inclusive design and international adaptation for conversational experiences

Each prompt includes comprehensive variable definitions, structured output formats (JSON/YAML), example implementations, and WCAG 3.0 Level AA accessibility requirements.

This library is a starting point, not a finished product. Prompts will be revised, expanded, and restructured as patterns are tested against use cases and new research surfaces.

---

## Library Contents

### Conversational AI (30 prompts)

Design natural(-like), coherent, and resilient conversation flows:

| Category | Prompts |
|----------|---------|
| **Dialog Flow** | [Multi-Turn Dialog Design](conversation-design/_conversational-ai/multi-turn-dialog-design.md), [Task-Oriented Dialog](conversation-design/_conversational-ai/task-oriented-dialog.md), [Context Management](conversation-design/_conversational-ai/context-management.md), [Topic Switching](conversation-design/_conversational-ai/topic-switching.md) |
| **Voice & Personality** | [Voice & Tone Calibration](conversation-design/_conversational-ai/voice-tone-calibration.md), [Personality Consistency](conversation-design/_conversational-ai/personality-consistency.md), [Sentiment Adaptation](conversation-design/_conversational-ai/sentiment-adaptation.md), [Response Length Optimization](conversation-design/_conversational-ai/response-length-optimization.md) |
| **Error Handling** | [Fallback Strategies](conversation-design/_conversational-ai/fallback-strategies.md), [Error Recovery Flows](conversation-design/_conversational-ai/error-recovery-flows.md), [Conversational Repair](conversation-design/_conversational-ai/conversational-repair.md), [Disambiguation Prompts](conversation-design/_conversational-ai/disambiguation-prompts.md) |
| **User Engagement** | [Proactive Suggestions](conversation-design/_conversational-ai/proactive-suggestions.md), [Re-engagement Prompts](conversation-design/_conversational-ai/reengagement-prompts.md), [Feedback Collection](conversation-design/_conversational-ai/feedback-collection.md), [Small Talk Design](conversation-design/_conversational-ai/small-talk-design.md) |
| **Conversation Management** | [Confirmation Patterns](conversation-design/_conversational-ai/confirmation-patterns.md), [Clarifying Questions](conversation-design/_conversational-ai/clarifying-questions.md), [Acknowledgment Responses](conversation-design/_conversational-ai/acknowledgment-responses.md), [Closing Conversations](conversation-design/_conversational-ai/closing-conversations.md) |
| **Transparency & Trust** | [Setting Expectations](conversation-design/_conversational-ai/setting-expectations.md), [Transparent Limitations](conversation-design/_conversational-ai/transparent-limitations.md), [Apologizing Effectively](conversation-design/_conversational-ai/apologizing-effectively.md), [Handoff to Human](conversation-design/_conversational-ai/handoff-to-human.md) |
| **Advanced Patterns** | [Memory References](conversation-design/_conversational-ai/memory-references.md), [Interruption Handling](conversation-design/_conversational-ai/interruption-handling.md), [Multi-Language Switching](conversation-design/_conversational-ai/multi-language-switching.md), [Voice Assistant Adaptations](conversation-design/_conversational-ai/voice-assistant-adaptations.md), [Chitchat Boundaries](conversation-design/_conversational-ai/chitchat-boundaries.md), [Question Answering Patterns](conversation-design/_conversational-ai/question-answering-patterns.md) |

### Voice UX (12 prompts)

Design voice-first and voice-only conversational experiences grounded in interactional principles from Conversation Analysis (CA) and Linguistics:

| Category | Prompts |
|----------|---------|
| **Turn Design** | [Turn Design & Progressivity](conversation-design/_voice-ux/turn-design-progressivity.md), [Sequential Information Gathering](conversation-design/_voice-ux/sequential-information-gathering.md), [Turn-Final Placement & End Focus](conversation-design/_voice-ux/turn-final-placement.md) |
| **Repair & Recovery** | [Voice Conversation Repair](conversation-design/_voice-ux/voice-conversation-repair.md), [No-Input & No-Match Escalation](conversation-design/_voice-ux/no-input-no-match-escalation.md), [Silence & Latency Management](conversation-design/_voice-ux/silence-latency-management.md) |
| **Prosody & Persona** | [Prosodic Design & Intonation](conversation-design/_voice-ux/prosodic-design-intonation.md), [Voice Persona & Phrasebook](conversation-design/_voice-ux/voice-persona-phrasebook.md) |
| **Interaction Management** | [Barge-In & Overlap Management](conversation-design/_voice-ux/barge-in-overlap-management.md), [Grounding & Confirmation](conversation-design/_voice-ux/grounding-confirmation-voice.md), [Opening & Closing Sequences](conversation-design/_voice-ux/opening-closing-sequences.md) |
| **Cross-Modal** | [Voice-to-Screen Handoff](conversation-design/_voice-ux/voice-to-screen-handoff.md) |

The Voice UX prompts draw on principles from Conversation Analysis and Interactional Linguistics (the empirical study of talk and social interaction) – including turn-taking organization (Sacks, Schegloff & Jefferson), repair preference hierarchy, end-focus and information structure (Halliday), and a situational personality & prosodic-lexical recipient design framework — to ground design recommendations in how spoken interaction actually works.

### Accessibility & Localization (20 prompts)

Build conversational experiences that work for everyone, everywhere:

| Category | Prompts |
|----------|---------|
| **Assistive Technology** | [Screen Reader Content](conversation-design/_accessibility-localization/screen-reader-content.md), [Alternative Text Generation](conversation-design/_accessibility-localization/alternative-text-generation.md), [ARIA Label Creation](conversation-design/_accessibility-localization/aria-label-creation.md), [Keyboard Navigation Copy](conversation-design/_accessibility-localization/keyboard-navigation-copy.md), [Captions & Subtitles](conversation-design/_accessibility-localization/captions-subtitles.md) |
| **Cognitive Accessibility** | [Plain Language Simplification](conversation-design/_accessibility-localization/plain-language-simplification.md), [Cognitive Accessibility](conversation-design/_accessibility-localization/cognitive-accessibility.md), [Color-Blind Safe Copy](conversation-design/_accessibility-localization/color-blind-safe-copy.md) |
| **Internationalization** | [Internationalization Prep](conversation-design/_accessibility-localization/internationalization-prep.md), [Cultural Adaptation](conversation-design/_accessibility-localization/cultural-adaptation.md), [RTL Content Design](conversation-design/_accessibility-localization/rtl-content-design.md), [Idiomatic Expression Handling](conversation-design/_accessibility-localization/idiomatic-expression-handling.md), [Character Expansion Planning](conversation-design/_accessibility-localization/character-expansion-planning.md), [Translation Quality Checks](conversation-design/_accessibility-localization/translation-quality-checks.md) |
| **Formatting & Conventions** | [Date/Time Localization](conversation-design/_accessibility-localization/date-time-localization.md), [Number/Currency Formatting](conversation-design/_accessibility-localization/number-currency-formatting.md), [Address Formatting](conversation-design/_accessibility-localization/address-formatting.md), [Name Formatting](conversation-design/_accessibility-localization/name-formatting.md) |
| **Inclusive Language** | [Gender-Neutral Language](conversation-design/_accessibility-localization/gender-neutral-language.md), [Emoji Usage Guidelines](conversation-design/_accessibility-localization/emoji-usage-guidelines.md) |

---

## How to Use

1. **Choose a prompt** — Browse the tables above or explore the folders directly
2. **Replace `{{variables}}`** with your project context (system name, audience, tone, etc.)
3. **Run with any LLM** — Works with Claude, ChatGPT, Gemini, Microsoft Copilot, or custom models
4. **Get structured output** — Each prompt returns JSON/YAML you can integrate directly

### Examples

```
# Open a prompt
conversation-design/_conversational-ai/handoff-to-human.md

# Fill in variables
ai_assistant_name: "MyAssist"
product_domain: "insurance claims"
handoff_trigger: "customer frustration or policy dispute"
tone: "empathetic, professional"

# Paste into your LLM → get structured handoff flows with
# transition copy, context summaries, and agent briefing templates
```

```
# Open a prompt
conversation-design/_voice-ux/voice-conversation-repair.md

# Fill in variables
assistant_name: "Nova"
platform: "Google Assistant"
domain: "flight booking"
repair_trigger: "no-match after destination input"

# Paste into your LLM → get a repair sequence with
# escalation hierarchy, re-prompt wording, and prosodic cues
```

---

## Who This Is For

- **Conversation designers** building chatbot and voice agent architectures and interaction flows
- **UX writers** crafting accessible, localized and CxD-optimized interface copy
- **Product teams** standardizing conversational patterns across features
- **AI engineers** designing graceful error handling and context management

---

## Credits

A conversation design-specific adaptation of the Content Design Prompt Library by Adedayo Agarau. Referred-to literature is mentioned in the respective prompt sections.

---

## License

This work is licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).

You are free to share, adapt, and build upon this material for any purpose, including commercially, as long as you provide appropriate credit to Philipp Haenggi, link to the license, and indicate if changes were made.
