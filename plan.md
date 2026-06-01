## Problem

`sentir-mais` currently has only a frontend with a ChatGPT-like chat experience and placeholder dashboard routes. The product goal is to let users vent freely, receive supportive AI responses, identify feelings from their text, relate those feelings to life events, and later visualize emotional patterns and key events over time.

## Current State

- Frontend only, built with Vue 3 + Vite.
- Existing user journey:
  - `/` starts a conversation with the first message.
  - `/chat/:id` continues the conversation.
  - `/dashboard/week` and `/services` are placeholders.
- Current chat backend is mocked in the frontend and only proves two operations:
  - create chat
  - send message in an existing chat
- A root `swagger.yaml` already captures the currently inferred backend contract for that chat flow.

## Confirmed MVP Decisions

- Users will have **accounts from the start**.
- Crisis-risk messages should still receive a **supportive response plus crisis resources and advice to seek immediate human help**.
- The backend stack will be **Go + authx + MongoDB**.
- The LLM will be consumed as an **external API/service**.
- The classifier will also be consumed as an **external API/service**, whether remote-hosted or as a separate local container.
- MVP generative usage can be budgeted through **OpenRouter** with roughly **USD 100**.
- The feeling-analysis architecture will be **hybrid**:
  - classifier-first for feeling prediction
  - LLM used for supportive replies
  - LLM also used for extraction and ambiguity handling

## Proposed Architecture

### Frontend

- Keep the current landing page and chat UX.
- Replace the mock chat transport with real HTTP requests.
- Add dashboard screens for recent feelings and notable events.

### Backend

- A single **Go API service** for the MVP.
- Main responsibilities:
  - authenticate users with `authx`
  - receive and persist chat messages
  - call the external LLM service
  - call the external classifier service
  - store structured event and feeling analysis
  - serve dashboard aggregates
- The API should isolate integrations behind internal interfaces:
  - auth middleware / auth service
  - `LLMClient`
  - `ClassifierClient`
  - MongoDB repositories
- If latency or throughput becomes a problem later, analysis can move to async workers, but MVP can start synchronously.

### Storage

- MongoDB is a good fit because chats, extracted events, and analysis results are document-shaped and likely to evolve.
- Likely collections:
  - users
  - chats
  - messages
  - message_analyses
  - daily_summaries
  - weekly_summaries

### External AI Services

- **LLM service**
  - Generates supportive responses.
  - Uses a system prompt focused on emotional support, not clinical diagnosis.
  - Uses a second prompt later for structured event extraction.

- **Classifier service**
  - Predicts feelings from extracted context.
  - Runs outside the Go API process.
  - Can be deployed as a separate container or hosted elsewhere.

## MVP Conversation Flow

1. User sends an initial message.
2. Go API authenticates the request with `authx` and stores the message.
3. API calls the external LLM with the system prompt and conversation history.
4. The LLM responds supportively and, when needed, asks the user about four key items:
   - what happened
   - what the user felt
   - what the user's reaction was
   - what the user expected should have happened, or how they think they should have reacted
5. The chat continues normally until those four items are sufficiently established in the conversation history.
6. Once enough context exists, the backend calls the LLM again with a separate extraction prompt over the conversation history.
7. The extraction step returns a structured representation of the event/context.
8. The backend sends that extracted context to the external classifier service.
9. The classifier returns predicted feelings and confidence values for the target labels.
10. The backend stores:
   - the raw messages
   - the assistant responses
   - the extracted event structure
   - the predicted feelings and confidences
11. Dashboard endpoints later aggregate this stored data into recent feelings and main events.

## AI / ML Responsibilities

### Supportive generation

- Respond calmly and supportively.
- Encourage the user to clarify the four key reflection points.
- Include safety-aware behavior for crisis language.

### Feeling identification

- Target labels for now:
  - happy
  - sad
  - stressed
  - angry
  - doubtful
  - anxious
  - relaxed
  - tense
- The classifier should receive normalized context after extraction, not only the last raw message.
- Current research recommendation for MVP classifier:
  - `MoritzLaurer/mDeBERTa-v3-base-mnli-xnli`

### Event extraction

- Use a dedicated prompt separate from the supportive chat prompt.
- Extract the event or situation in a structured format once enough context exists.
- Feed extracted event/context into the classifier.

## Suggested MVP API Surface

- auth endpoints required by `authx`
- `POST /chats`
- `POST /chats/{chatId}/messages`
- `GET /chats/{chatId}/messages`
- `GET /dashboard/week`
- `GET /dashboard/timeline`
- possibly later:
  - `GET /events`
  - `GET /feelings/history`

## Data We Likely Need Per Message / Analysis

- user id
- chat id
- raw user text
- assistant reply
- whether the four key reflection fields are already established
- extracted event object
- detected primary feeling
- detected secondary feelings
- confidence scores
- timestamps
- model metadata

## Still To Refine

- The exact `authx` integration shape and auth endpoints.
- Concrete safety rules for crisis detection and the exact response template/resource links.
- The structure of the extracted event object.
- The exact first dashboard contract: likely a simple weekly timeline before broader analytics.

## Working Todo Outline

1. Design the Go backend architecture and package boundaries.
2. Design MongoDB document schemas and retention/privacy rules.
3. Expand the OpenAPI contract from chat-only to the Go/authx/MongoDB MVP backend.
4. Implement the Go backend and replace the frontend mock API.
5. Implement the external classifier service integration.
6. Implement extraction + analysis persistence.
7. Implement dashboard aggregates and visualizations.
