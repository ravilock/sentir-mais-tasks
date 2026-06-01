# LLM Prompting Spec

Date: 2026-06-01

## Purpose

This document closes the MVP prompting decisions for Sentir Mais so the next implementation tasks can proceed without reopening provider and prompt basics.

This spec is intended for use by `sentir-mais-prompter` and `sentir-mais-backend`.

## Provider And Runtime Decision

- provider gateway: OpenRouter
- integration boundary: `sentir-mais-prompter`
- primary API surface: `chat/completions`
- first default model candidate: `google/gemini-2.5-flash-lite`
- fallback candidate: `openai/gpt-4.1-mini`

Why:

- lower integration complexity
- low MVP cost
- easy future model replacement
- structured-output support is available on the OpenRouter path

## Prompt Families

We need two prompt families:

- supportive conversation
- structured extraction

The classifier remains a separate service and should not be replaced by LLM label prediction for the MVP.

Important boundary:

- the LLM extraction step may capture emotional language explicitly described by the user
- the LLM extraction step must not predict or normalize the final feeling labels
- final feeling labels and confidence scores are the classifier's responsibility

## Supportive Conversation Prompt

### Goal

Generate a supportive assistant reply that:

- acknowledges the user’s emotional state
- avoids diagnosis
- keeps the tone calm and grounded
- helps gather the four required reflection points over time
- escalates appropriately if there is crisis-risk language

### Required Reflection Points

The assistant should gradually help the user clarify:

1. what happened
2. what the user felt
3. how the user reacted
4. what the user expected should have happened, or how they believe they should have reacted

### Supportive System Prompt Draft

```text
You are Sentir Mais, a calm and supportive emotional support assistant.

Your role is to help the user slow down, feel heard, and organize what happened without judging or diagnosing them.

You are not a doctor, therapist, or crisis professional. Do not present yourself as one. Do not give clinical diagnoses. Do not claim certainty about the user's mental health state.

Your main conversational goals are:
1. respond with empathy and emotional steadiness
2. help the user describe what happened
3. help the user identify what they felt
4. help the user describe how they reacted
5. help the user identify what they expected should have happened, or how they think they should have reacted

Behavior rules:
- Keep responses concise, clear, and supportive.
- Prefer one grounded reflection plus one useful follow-up question.
- Do not overwhelm the user with too many questions at once.
- Do not sound robotic, preachy, or overly formal.
- Do not give legal, medical, or financial advice.
- Do not invent facts that the user did not say.
- If details are unclear, ask for clarification instead of guessing.
- If the user already supplied one of the four reflection points, build on it instead of repeating the same question.

If the user expresses self-harm, suicide risk, or immediate danger:
- respond with warmth and urgency
- encourage immediate human help
- recommend contacting local emergency services if there is immediate danger
- in Brazil, mention CVV 188 as a crisis resource
- encourage contacting a trusted person right now
- do not continue the conversation as if it were a normal reflective turn

Your output must be plain assistant text only.
```

### Runtime Guidance For Supportive Replies

- temperature: `0.7`
- token limit: moderate, enough for short conversational replies
- response format: normal text
- classifier is not called from this step directly

## Extraction Prompt

### Goal

Turn conversation history into a structured event/context representation suitable for persistence and downstream classification.

### Extraction Rules

- Use the full conversation history, not only the last message.
- Extract only what is reasonably supported by the conversation.
- If something is unclear or not yet established, leave it null instead of inventing content.
- The output should be valid JSON and follow the agreed schema strictly.
- If the user explicitly describes emotions in their own words, capture that as evidence text.
- Do not infer or predict final classifier labels such as `happy`, `sad`, `stressed`, `angry`, `doubtful`, `anxious`, `relaxed`, or `tense` unless the user explicitly used those terms.
- Do not convert free-form emotional language into normalized classifier output.

### Extraction System Prompt Draft

```text
You are an information extraction component for Sentir Mais.

Your task is to read a conversation and extract a structured representation of the user's situation.

You must extract only information grounded in the conversation. Do not invent missing facts. If a field is unclear, uncertain, or not established, return null for that field.

You are not a classifier. Do not predict final emotion labels. Only capture emotional words or descriptions when they were explicitly stated by the user or are directly quoted/paraphrased from the conversation as evidence.

The extraction should focus on:
- what happened
- what the user said they felt
- how the user reacted
- what the user expected should have happened, or how they think they should have reacted

You must also assess whether these four reflection points are sufficiently established in the conversation.

Return valid JSON only, matching the required schema exactly.
Do not include markdown.
Do not include explanatory prose.
```

### Extraction User Prompt Wrapper

```text
Extract the structured event/context from the conversation below.

Conversation:
{{conversation_history}}
```

### Runtime Guidance For Extraction

- temperature: `0.1` to `0.2`
- response format: structured JSON
- require structured-output-compatible routing where available
- stricter model behavior than the supportive prompt path

## Extraction Schema Decision

This is the MVP extraction object to persist and send forward into analysis:

```json
{
  "enough_context": true,
  "context_gaps": [],
  "event_summary": "The user had an argument with their manager after receiving criticism in a meeting.",
  "what_happened": "The user received criticism from their manager during a meeting and later argued with them.",
  "felt_emotions_described_by_user": [
    "anxious",
    "angry"
  ],
  "user_reaction": "The user became defensive, argued, and later withdrew.",
  "expected_outcome_or_self_expectation": "The user expected more respectful feedback and thinks they should have stayed calmer.",
  "people_involved": [
    "manager"
  ],
  "setting": "workplace meeting",
  "time_reference": "today",
  "risk_flags": {
    "self_harm": false,
    "suicidal_ideation": false,
    "immediate_danger": false
  },
  "confidence_notes": "Most fields are directly supported by the conversation, but the time reference is somewhat vague."
}
```

`felt_emotions_described_by_user` is evidence from the conversation, not classifier output. It should contain only feelings the user explicitly stated or very directly described. It must not be treated as the final normalized feeling decision.

### JSON Schema Shape

```json
{
  "type": "object",
  "additionalProperties": false,
  "required": [
    "enough_context",
    "context_gaps",
    "event_summary",
    "what_happened",
    "felt_emotions_described_by_user",
    "user_reaction",
    "expected_outcome_or_self_expectation",
    "people_involved",
    "setting",
    "time_reference",
    "risk_flags",
    "confidence_notes"
  ],
  "properties": {
    "enough_context": {
      "type": "boolean"
    },
    "context_gaps": {
      "type": "array",
      "items": {
        "type": "string",
        "enum": [
          "what_happened",
          "felt_emotions_described_by_user",
          "user_reaction",
          "expected_outcome_or_self_expectation"
        ]
      }
    },
    "event_summary": {
      "type": ["string", "null"]
    },
    "what_happened": {
      "type": ["string", "null"]
    },
    "felt_emotions_described_by_user": {
      "type": "array",
      "items": {
        "type": "string"
      }
    },
    "user_reaction": {
      "type": ["string", "null"]
    },
    "expected_outcome_or_self_expectation": {
      "type": ["string", "null"]
    },
    "people_involved": {
      "type": "array",
      "items": {
        "type": "string"
      }
    },
    "setting": {
      "type": ["string", "null"]
    },
    "time_reference": {
      "type": ["string", "null"]
    },
    "risk_flags": {
      "type": "object",
      "additionalProperties": false,
      "required": [
        "self_harm",
        "suicidal_ideation",
        "immediate_danger"
      ],
      "properties": {
        "self_harm": {
          "type": "boolean"
        },
        "suicidal_ideation": {
          "type": "boolean"
        },
        "immediate_danger": {
          "type": "boolean"
        }
      }
    },
    "confidence_notes": {
      "type": ["string", "null"]
    }
  }
}
```

## Definition Of Enough Context

For MVP, `enough_context` should be `true` only when the conversation provides enough support for all four reflection areas:

- what happened
- what the user felt
- how the user reacted
- what the user expected should have happened or how they should have reacted

If any of those are missing or too vague, `enough_context` should be `false`, and the missing field names should appear in `context_gaps`.

## Proposed Config / Environment Variables

For `sentir-mais-prompter`:

- `API_KEY`
- `LLM_PROVIDER`
- `LLM_BASE_URL`
- `LLM_API_KEY`
- `DEFAULT_MODEL`
- `REQUEST_TIMEOUT_SECONDS`
- `APP_URL`
- `APP_TITLE`
- `ALLOW_MODEL_OVERRIDE`

For later routing/privacy controls, we should be ready to add:

- `OPENROUTER_ENABLE_ZDR`
- `OPENROUTER_DATA_COLLECTION`
- `OPENROUTER_REQUIRE_PARAMETERS`
- `OPENROUTER_ALLOWED_PROVIDERS`

For `sentir-mais-backend`:

- `PROMPTER_BASE_URL`
- `PROMPTER_API_KEY`
- `PROMPTER_TIMEOUT_SECONDS`

## Error Handling Notes

Recommended MVP behavior:

- provider timeout: treat as transient failure and return a backend integration error
- non-2xx provider response: capture status and short body snippet in logs
- invalid structured output: treat as extraction failure and do not persist partial extraction as valid
- missing API key or provider config: fail fast on service startup where possible

Retry posture:

- supportive generation: at most one retry on transient transport failure
- extraction: at most one retry on transient transport failure
- do not blindly retry validation failures or provider 4xx errors

Logging:

- log provider request id
- log model name
- log token usage
- avoid logging full user conversation content in plaintext unless explicitly needed for secure debugging

## Implementation Impact

This spec unblocks:

- `013-task-prompter-service`
- backend-to-prompter integration work
- context sufficiency detection
- structured extraction
- classifier-from-extracted-context flow
