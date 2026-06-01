# OpenRouter MVP Research

Date: 2026-06-01

## Decision

OpenRouter is a good fit for the Sentir Mais MVP as the backing LLM provider behind `sentir-mais-prompter`.

<https://openrouter.ai/settings/credits>

It matches the current backlog assumptions, gives us a single OpenAI-compatible API surface, and keeps provider-specific logic out of `sentir-mais-backend`. For MVP, the main benefit is flexibility and speed of integration.

The main caveat is privacy. Since Sentir Mais handles mental-health-adjacent text, we should not rely on default routing behavior. Provider selection and privacy-related routing controls should be explicit.

## Why It Fits

- OpenAI-compatible API shape reduces implementation complexity.
- It supports routing across multiple model providers behind one interface.
- It makes it easier to switch models later without changing the backend contract.
- It fits the planned `sentir-mais-prompter` service boundary well.
- It appears cost-compatible with the stated MVP budget.

## Recommended MVP Integration Shape

Use OpenRouter through `sentir-mais-prompter`, not directly from `sentir-mais-backend`.

Recommended first approach:

- endpoint: `POST /api/v1/chat/completions`
- auth: `Authorization: Bearer <OPENROUTER_API_KEY>`
- one runtime key per environment
- set spend caps on the key
- pin a model instead of relying on unconstrained auto-routing

## Request / Response Shape

For supportive conversation:

- use `chat/completions`
- send:
  - `model`
  - `messages`
  - `temperature`
  - `max_completion_tokens` or equivalent token limit
  - optional trace metadata such as user/session identifiers

Expected response fields:

- `choices[0].message.content`
- `usage.prompt_tokens`
- `usage.completion_tokens`
- `usage.total_tokens`
- `id`
- `model`

For structured extraction:

- use the same `chat/completions` endpoint
- provide conversation history in `messages`
- prefer structured output with JSON schema response format
- require parameter support when routing requests

This is the cleanest MVP path for extracting structured event/context data.

## Routing And Privacy Guidance

Do not leave everything on unrestricted default routing.

Prefer:

- a pinned model
- limited provider fallback
- `data_collection: "deny"` where supported
- `zdr: true` where supported
- `require_parameters: true` for extraction requests

Useful routing controls mentioned in the research:

- `order`
- `allow_fallbacks`
- `require_parameters`
- `only`
- `ignore`

## Budget Notes

Examples observed during research:

- `google/gemini-2.5-flash-lite`: about `$0.10/M` input and `$0.40/M` output
- `openai/gpt-4.1-mini`: about `$0.40/M` input and `$1.60/M` output
- `anthropic/claude-3.5-haiku`: about `$0.80/M` input and `$4.00/M` output

Illustrative conversation estimate used in research:

- about `25k` input tokens total
- about `4k` output tokens total
- one extraction pass included

Approximate cost per conversation:

- Gemini 2.5 Flash Lite: about `$0.0041`
- GPT-4.1 Mini: about `$0.0164`
- Claude 3.5 Haiku: about `$0.036`

Conclusion: the earlier `$100` MVP budget looks workable for low-volume usage, as long as we control history size and model selection.

## Risks And Caveats

- Privacy depends on both OpenRouter and the selected upstream provider.
- Structured outputs are not supported equally across all model/provider combinations.
- Default routing can reduce reproducibility.
- Model availability and pricing can change over time.
- Safety handling is still our responsibility; OpenRouter does not replace explicit crisis behavior in the app.
- `chat/completions` is the simpler MVP surface; richer alternatives exist but are not necessary for the first version.

## Recommendation

Use OpenRouter for the MVP with this posture:

- use `chat/completions`
- put it behind `sentir-mais-prompter`
- start with a pinned low-cost model
- use JSON-structured extraction output
- enforce privacy-aware routing controls
- set spend caps per environment
- log provider request id, model, and token usage

Good first model candidates mentioned in the research:

- `google/gemini-2.5-flash-lite`
- `openai/gpt-4.1-mini`

## References

- Authentication: <https://openrouter.ai/docs/api/reference/authentication>
- Chat completions: <https://openrouter.ai/docs/api/api-reference/chat/send-chat-completion-request>
- API overview: <https://openrouter.ai/docs/api/reference/overview>
- Provider routing: <https://openrouter.ai/docs/guides/routing/provider-selection>
- Structured outputs: <https://openrouter.ai/docs/guides/features/structured-outputs>
- Pricing: <https://openrouter.ai/pricing>
- Zero Data Retention: <https://openrouter.ai/docs/guides/features/zdr>
- Provider logging/privacy: <https://openrouter.ai/docs/guides/privacy/provider-logging>
- Generation metadata: <https://openrouter.ai/docs/api/api-reference/generations/get-generation>
