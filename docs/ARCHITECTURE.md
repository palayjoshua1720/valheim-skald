# Architecture

## 1. Overview

```text
Game event
   │
   ▼
Harmony event emitters ─────────────── Mic input (optional)
   │                                        │
   ▼                                        ▼
Event Director                         Local Whisper STT
(flags, cooldown,                           │ transcript
 probability, priority)                     │
   │                                        │
   ├── Verified local advice / cached line pool
   │                 │
   │                 └── output line
   │
   └── No suitable cached comment?
                     │
                     ▼
              Compact Prompt Builder
                     │
                     ▼
       Direct provider call (optional)
       UnityWebRequest, ≤50 output tokens
                     │
                     ▼
             Validate / fallback
                     │
                     ▼
                  Output Router
               ┌─────┴─────┐
               ▼           ▼
         Caption overlay   Local TTS
                           (WAV / OGG)
```

Voice transcripts enter the same question/event decision path. No proxy service is involved in LLM requests. Whisper and Piper are optional local helper components; they do not relay LLM traffic.

## 2. Components

- **Harmony patches / event emitters:** Observe selected game lifecycle and combat signals, then emit small domain events. Each hook must be verified in the current game's assemblies before relying on it.
- **Event Director:** Validates events, checks once-only flags and cooldowns, applies ambient probability, ranks queued events, coalesces matches by group and occurrence, and drops stale work. For each occurrence, display only the highest-priority eligible event in each group. Consume the once-only keys of every matching eligible event before selecting the display winner, so suppressed milestones do not surface on a later occurrence. Break priority ties deterministically by event ID.
- **State Store:** Persists flags and statistics separately for each character/world pair. Writes must not block the Unity main thread.
- **Facts Provider:** Reads curated local facts. `factsOptional` entries identify candidate collections using singular namespaces (`biome:<id>`, `enemy:<id>`, `hazard:<id>`) that resolve to the `biomes`, `enemies`, and `hazards` objects in `data/facts.json`. The provider selects verified individual fact objects/IDs from those collections; absent or unverified facts never block a comment.
- **Pool Manager:** Loads and selects cached lines, tracks repetition, and can identify when a pool needs replenishing. Cached comments must avoid unsupported game claims; biome-, item-, and damage-themed lines require verification of the facts they imply. Factual advice comes from verified local data.
- **Prompt Builder:** Combines a compact event payload with style, intensity, and only the relevant verified facts. `factsOptional` is enrichment, never a condition for a comment: missing or unverified facts suppress advice that depends on them, not the event comment. Avoid chat history by default.
- **LLM Client:** Calls the player's configured OpenAI-compatible endpoint directly using `UnityWebRequest`. Streaming is a later optimization. The initial implementation should support one compatible endpoint and be extensible to others.
- **Output Router:** Applies the independent advice/comment output modes and dispatches to captions, TTS, or both.
- **Caption Overlay:** Displays short text in the Unity UI with configurable location, size, opacity, and duration.
- **TTS Player:** Plays optional local Piper output using WAV or OGG. Do not target MP3.
- **STT Helper:** Optional local Whisper process for push-to-talk voice transcription. The transcript can be sent to the configured provider when needed; raw microphone audio is not sent by this design.
- **Usage/Budget tracker:** Enforces session and hourly live-call caps and reports usage without exposing keys or sensitive prompt content.

## 3. Game hooks to investigate

The following are investigation leads from the project brief, **not confirmed hooks**. Verify signatures, call timing, and behavior in dnSpy/ILSpy against the current Valheim version before implementing them:

- `Player.OnDeath` for death observation and cause attribution.
- `HitData` hit information and attacker lookup such as `GetAttacker` for tactical event context.
- Biome transition tracking.
- Dungeon/interior entry detection.
- Boss death and relevant global-key updates.
- `Character.Awake` or `SetTarget` as possible first-encounter signals.
- `Character.m_damageModifiers` and its modifier enum/value representation as a hypothesis for reading damage responses from the runtime enemy instance. Confirm the member, access path, enum names, and values in the current game assemblies before using it.

Prefer a reliable, minimal event signal over a fragile hook that produces richer but untrustworthy context. Record the game version and verified hook behavior in [HOOK_NOTES.md](HOOK_NOTES.md) as testing proceeds. Candidate hooks in this section remain unverified until those notes contain inspection and runtime-test results.

For enemy matching, normalize the runtime prefab name by removing only a trailing `(Clone)` suffix before matching it against `prefabNames`. Populate those names only after checking the actual game prefabs; display names and IDs are not assumed to equal internal prefab names. Limit first-encounter reactions to the curated `notableEnemyIds` list in `events.json`, not every creature.

## 4. Token and data strategy

- Target roughly 90% cached/local output and 10% live calls after the event system and pools are established. Treat this as an optimization goal, not an MVP acceptance criterion.
- Serve advice from verified local data. Initially, live calls generate comments only.
- Use compact event payloads and inject only one to three relevant verified facts.
- Limit live output to about 50 tokens, with no conversation history by default.
- Use a static system/style prompt and small dynamic event context. Keep prompts concise and avoid reasoning models that spend output budgets on hidden reasoning.
- Use cached comments and deterministic fallback lines when there is no key, no suitable pool entry, or a provider failure.
- Avoid repetition by tracking the first words of a few recent lines or another small local signature.

Fact records in `facts.json` use this shape:

```json
{"id":"stable_fact_id","text":"Fact text","verified":false,"source":"","gameVersion":""}
```

Each fact is independently verified and traceable. Keep weakness/resistance claims out of this file when they can be read reliably from the current enemy instance at runtime. Use `causeMap` in `facts.json` to map event cause IDs to `hazard:<id>` or `enemy:<id>` fact namespaces.

Illustrative compact payload, with placeholders rather than unverified Valheim claims:

```text
E=death cause=<verified_cause_id> count=<death_count> FACTS=<verified_fact_ids>
```

Only include fields the event actually provides. Do not infer an attacker, cause, timing, damage type, or fact when it is unknown.

## Event grouping and once-only keys

Every event has a `group`. For one source occurrence, the Event Director displays only the highest-priority matching event in each group. It still consumes the once-only flags for all matching eligible events, including suppressed matches. This prevents a lower-priority reaction from unexpectedly appearing on a later occurrence. A group can produce a display independently of another group; for example, death and tactical events are distinct groups.

`oncePer` selects the character/world scope, and optional `oncePerKey` fields refine that scope. The stored flag identity is the character/world pair, event ID, and the values of the listed payload fields. The allowed group values are `death`, `biome`, `boss`, `enemy`, `tactical`, `ambient`, and `voice`. `dungeon_first` uses the `biome` group as a location-discovery event because the event schema has no separate dungeon group.

## 5. Realtime and thread handling

- Never block the Unity main thread on disk I/O, local helper processes, or network requests.
- Perform provider requests asynchronously; marshal Unity UI and audio operations back to the main thread as required by Unity.
- Apply an event-specific relevance deadline. Drop late responses rather than displaying a warning after it is useful.
- Add streaming captions, sentence-level TTS, and prefetching only after non-streaming calls and fallback behavior are stable.

## 6. Failure handling

For provider authentication failures (401), rate limits (429), timeouts, malformed responses, or refusals:

1. Discard invalid or overlong output.
2. Select a suitable cached/fallback comment, or show nothing if no safe line exists.
3. Display at most one concise configuration/status notice; do not repeat errors for every event.
4. Keep the game running and the local event system usable.

Do not retry indefinitely. Any retries must be bounded and must respect the hourly/session budget.

## 7. Security and privacy

- The API key is stored in the local BepInEx config and may be plaintext. Document that users must not share the config file.
- Never log the key, authorization headers, or full secret-bearing configuration.
- Only compact event text and voice transcripts intended for a provider request are sent to the selected endpoint. Do not transmit raw microphone audio in this design.
- STT and TTS are optional local helpers. Clearly indicate when the player enables provider-backed voice questions.
- Do not commit real configuration, `.env` files, downloaded models, or secrets.

## 8. Provider and runtime notes

Start with one OpenAI-compatible endpoint configured by base URL, model, and user key. Add provider-specific behavior only where wire-format differences require it. Potential endpoints from the product brief include OpenAI-compatible services, Groq, OpenRouter, Gemini-compatible endpoints, Ollama, and LM Studio; compatibility must be tested rather than assumed from a provider name.

The mod targets C# on Unity Mono and .NET Framework 4.8. Keep helper components optional and isolate their process/audio integration from the core plugin. Local helper processes are not an LLM proxy.
