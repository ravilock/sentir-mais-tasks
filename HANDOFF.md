# Backend Handoff

This document merges the earlier `plan.md` and `HANDOFF.md` context into a single backend handoff for `sentir-mais`.

## 1. Problem

`sentir-mais` currently has only a frontend with a ChatGPT-like chat experience and placeholder dashboard routes. The product goal is to let users vent freely, receive supportive AI responses, identify feelings from their text, relate those feelings to life events, and later visualize emotional patterns and key events over time.

## 2. Current repository state

- The repo currently contains the frontend under `./sentir-mais/`.
- A backend repo was created under `./sentir-mais-backend/`.
- Relevant root files already added:
  - `plan.md`
  - `swagger.yaml`
- Current top-level status when the earlier handoff was written:
  - untracked: `.tool-versions`, `plan.md`, `swagger.yaml`

## 3. Important correction to historical plan assumptions

`plan.md` still mentions **Go + authx + MongoDB** in several places.

That is now partially outdated:

- **Go** is still the backend language.
- **MongoDB** is still the intended database.
- **Do not use `authx`.**
- The latest user direction was: **implement authentication on our own**.

So the backend should assume:

- app-owned auth
- app-owned user model
- app-owned session / token strategy

## 4. Product summary

The app is an emotional support chat product with a ChatGPT-like interface.

The core user flow is:

1. User logs in.
2. User starts a chat and vents freely.
3. The AI responds supportively.
4. During the conversation, the AI helps establish:
   - what happened
   - what the user felt
   - what the user's reaction was
   - what the user expected should have happened, or how they think they should have reacted
5. Once enough context exists, the backend runs a separate extraction step.
6. Extracted event/context is sent to a feeling classifier service.
7. Backend stores messages, extracted events, and feeling predictions.
8. Dashboard endpoints later summarize feelings and main events over time.

## 5. Frontend behavior the backend must support first

The current frontend already proves a minimal chat contract.

### Existing routes

- `/`
  - initial chat entry
- `/chat/:id`
  - ongoing chat view
- `/dashboard/week`
  - placeholder
- `/services`
  - placeholder

### Current frontend integration points

- `src/http/chat/chat.interface.ts`
  - defines:
    - `createChat(initialMessage: string): Promise<{ chatId, response }>`
    - `sendMessage(chatId, message): Promise<Message>`
- `src/http/chat/chat.mock.ts`
  - current in-browser mock implementation
- `src/views/HomeView.vue`
  - calls `createChat`
  - stores initial user text + first assistant response in a store
  - redirects to `/chat/:id`
- `src/views/ChatView.vue`
  - appends the user message locally
  - calls `sendMessage`
  - renders the assistant reply with typing animation
  - temporarily persists chat messages in `sessionStorage`

### Current API contract already documented

`swagger.yaml` currently documents the backend inferred from the frontend:

- `POST /chats`
- `POST /chats/{chatId}/messages`

That file is a good starting point, but it is **not yet the full MVP API**.

### Frontend follow-up after backend exists

- Replace the mock chat transport with real HTTP requests once auth and chat endpoints exist.
- Keep the current landing page and chat UX for the MVP.
- Add dashboard screens after the first weekly dashboard contract is stable.

## 6. Confirmed architecture decisions

### Backend

- Build a **single Go API service** first.
- Keep the first version synchronous unless latency forces async work later.
- Separate responsibilities inside the Go service:
  - HTTP handlers
  - auth
  - chat orchestration
  - LLM client
  - classifier client
  - repositories
  - analysis/extraction pipeline

### Auth

- Implement authentication in-house.
- Accounts are required from the start.
- Likely MVP shape:
  - email + password
  - password hashing
  - JWT or session-token based auth

### Storage

- Use **MongoDB**.
- Likely collections:
  - `users`
  - `chats`
  - `messages`
  - `message_analyses`
  - `daily_summaries`
  - `weekly_summaries`

### LLM

- Consume the LLM as an **external API/service**.
- MVP provider assumption: **OpenRouter** with about **USD 100** budget.
- The LLM has two distinct jobs:
  1. supportive conversation
  2. structured extraction from conversation history

### Classifier

- Consume the feeling classifier as an **external API/service**.
- It may be:
  - another container on the same machine, or
  - a separately hosted service
- It should not be embedded directly into the Go process.

## 7. Conversation orchestration to implement

This is the latest agreed backend flow.

1. User sends initial message.
2. Backend authenticates user and stores the message.
3. Backend sends conversation history to the LLM using the **supportive system prompt**.
4. The LLM should:
   - respond supportively
   - try to help the user calm down
   - guide the conversation toward identifying:
     - what happened
     - what the user felt
     - what the user's reaction was
     - what the user expected should have happened or how they should have reacted
5. The conversation continues through normal chat turns until those four areas are sufficiently established.
6. Once enough context exists, the backend sends conversation history to the LLM again with a **different extraction prompt**.
7. The extraction prompt should return a structured representation of the event/context.
8. Backend sends the extracted context to the classifier service.
9. Classifier returns predicted feelings and confidence scores.
10. Backend persists:
    - raw messages
    - assistant replies
    - extracted event structure
    - predicted feelings
    - confidence scores
    - model metadata if useful

## 8. Feeling model guidance

The research direction already established:

- A dedicated classifier is likely **more consistent than prompt-only LLM classification**.
- The current recommended MVP classifier candidate is:
  - `MoritzLaurer/mDeBERTa-v3-base-mnli-xnli`
- The target labels are:
  - happy
  - sad
  - stressed
  - angry
  - doubtful
  - anxious
  - relaxed
  - tense

Important caveat:

- Off-the-shelf models do not map perfectly to this label set.
- `stressed`, `tense`, and `relaxed` are likely the hardest labels.
- A hybrid architecture is intentional:
  - classifier for consistent scoring
  - LLM for support, extraction, and ambiguity handling

## 9. Safety behavior

The product direction is:

- If the user writes something suggesting self-harm, suicide risk, or immediate danger:
  - do **not** silently continue as normal
  - keep the tone supportive
  - provide crisis resources
  - advise seeking immediate human help

This still needs to be turned into explicit product rules and prompt rules.

## 10. Suggested backend package/module boundaries

This is a practical starting point for the Go implementation.

```text
backend/
  cmd/api/
  internal/
    app/
    config/
    http/
      handlers/
      middleware/
      dto/
    auth/
    chat/
    analysis/
    llm/
    classifier/
    storage/
      mongo/
    domain/
```

Suggested ownership:

- `auth/`
  - registration
  - login
  - password hashing
  - token issuing / validation
- `chat/`
  - chat creation
  - message append
  - chat history retrieval
- `analysis/`
  - "do we have enough context yet?"
  - extraction orchestration
  - classifier call orchestration
- `llm/`
  - supportive prompt
  - extraction prompt
  - provider integration
- `classifier/`
  - HTTP client for external classifier service
- `storage/mongo/`
  - repositories

## 11. Data model starting point

This is not finalized, but it is a strong initial shape.

### users

- `_id`
- `email`
- `password_hash`
- `created_at`
- `updated_at`

### chats

- `_id`
- `user_id`
- `created_at`
- `updated_at`
- optional current status / analysis status

### messages

- `_id`
- `chat_id`
- `user_id`
- `sender` (`user` or `assistant`)
- `content`
- `created_at`

### message_analyses

- `_id`
- `chat_id`
- `message_id` or conversation checkpoint id
- `has_event_context`
- `what_happened`
- `felt`
- `reaction`
- `expected_outcome`
- `extracted_event`
- `primary_feeling`
- `secondary_feelings`
- `confidence_scores`
- `classifier_provider`
- `classifier_model`
- `llm_provider`
- `llm_model`
- `created_at`

### weekly_summaries

- `_id`
- `user_id`
- `week_start`
- `dominant_feelings`
- `main_events`
- `timeline_points`
- `generated_at`

## 12. API work that should happen next

The first backend pass should define and implement:

### Auth

- `POST /auth/register`
- `POST /auth/login`
- `GET /auth/me`

### Chat

- `POST /chats`
- `POST /chats/{chatId}/messages`
- `GET /chats/{chatId}/messages`

### Dashboard

- `GET /dashboard/week`
- possibly later `GET /dashboard/timeline`

## 13. Open questions still to refine

- The exact auth/session strategy for the in-house implementation.
- Concrete safety rules for crisis detection and the exact response template/resource links.
- The structure of the extracted event object.
- The exact first dashboard contract beyond the weekly summary.
- Data retention and privacy rules for chats, extracted events, and derived analyses.

## 14. Recommended implementation order

1. **Update `plan.md` to remove `authx` references**
   - this is the first cleanup step
2. **Create the Go backend scaffold**
   - `go.mod`
   - server entrypoint
   - config
   - routing
3. **Implement in-house auth**
   - register
   - login
   - auth middleware
4. **Implement MongoDB repositories**
   - users
   - chats
   - messages
5. **Implement the current chat contract from `swagger.yaml`**
   - create chat
   - send message
6. **Add LLM integration**
   - supportive chat prompt
7. **Add extraction orchestration**
   - second prompt for structured event extraction
8. **Add classifier integration**
   - external service client
   - persistence of predictions
9. **Expand OpenAPI**
   - auth
   - chat history
   - dashboard
10. **Wire the frontend off the mock client**
11. **Design retention/privacy rules and dashboard follow-up**
   - data lifecycle
   - weekly dashboard contract
   - later dashboard screens

## 15. Known repo and validation notes

- Earlier frontend validation found:
  - `npm run build` passes
  - `npm run lint` passes
  - unit tests have an existing failure in `src/components/ChatMessage/ChatMessage.spec.ts`
    because `vue-mdr` imports a missing `ShikiProvider`
- This failure is unrelated to the backend plan, but it is useful context if frontend checks are rerun.

## 16. Immediate next action for whoever picks this up

Start by doing these three things:

1. Update `plan.md` to replace `authx` with app-owned auth.
2. Add the Go backend scaffold in a new backend directory or other agreed layout.
3. Expand `swagger.yaml` into the first real MVP contract covering auth + chat + dashboard.

If those three are done cleanly, the rest of the backend implementation can proceed without needing the original conversation.
