# Product Specification

## 1. Vision

Valheim AI Companion is an event-driven commentator and coach for Valheim. It notices selected gameplay events and responds with short comments or useful hints through captions, speech, or both. It is an off-screen companion voice, not a spawned follower NPC or autonomous combat companion.

The mod is free and open source. It runs in Valheim through BepInEx and HarmonyX, using C# on Unity Mono with a .NET Framework 4.8 target. Live language-model calls are optional and use the player's own provider credentials directly from the game; there is no LLM proxy service.

The product priorities are:

1. **Truthful hints:** Advice must be accurate and sincere. Game-specific advice is sourced from curated local data and is omitted when the relevant fact is unavailable or unverified.
2. **Expressive comments:** Style and roast intensity affect comments, not the truthfulness or sincerity of advice.
3. **Useful offline behavior:** Captions and local fallback lines work without an API key or an available provider.
4. **Player control:** Advice and comments have separate output settings, with spam controls and a live-call budget.
5. **Low data and token use:** Send only compact event information when a live call is needed; do not send conversation history by default.

## 2. Personality system

Style and roast intensity are separate settings:

- **Style** selects the companion's voice and vocabulary.
- **Roast intensity** controls how sharp comments are, from 1 to 5.
- Neither setting may make advice inaccurate, insincere, or insulting toward real-world traits.

| Style | Voice | Sample comment |
| --- | --- | --- |
| Friendly | Warm, reassuring, and encouraging. | “That was a rough moment. Take a breath and try again when you’re ready.” |
| Casual | Relaxed gaming buddy with mild humor. | “That plan had confidence. The results are asking for a rematch.” |
| Professional | Neutral, concise, and free of jokes or roleplay. | “The attempt ended in defeat. Review the situation before continuing.” |
| Unhinged | Sharp, sarcastic, darkly comic; targets gameplay only. | “That strategy has been reported missing. Last seen near your last decision.” |

These samples avoid specific claims about Valheim mechanics, items, or creature properties.

### Roast intensity

| Level | Intended tone |
| --- | --- |
| 1 | Gentle teasing, with reassurance. |
| 2 | Light ribbing. |
| 3 | Playful roasting. |
| 4 | Strong, pointed gameplay roast. |
| 5 | Merciless dark roast of in-game behavior only. |

Intensity never changes advice. Comments must not target real-world identity, appearance, ability, or personal circumstances.

## 3. Trigger categories

Triggers are proposals until their game signals and behavior are verified against the supported Valheim version. All one-time and statistical state is scoped to the combination of character and world.

### A. First-time milestones

Fire once for the relevant character/world state:

- First death overall and first death by a recognized cause.
- First qualifying death within a configured time window after a tracked boss defeat. The initial showcase is a Deathsquito death shortly after defeating Moder; record the boss-defeat timestamp and compare it with the death timestamp. Treat the window as a configurable value, not a game fact.
- First entry into a biome.
- First entry into a dungeon or interior type.
- First encounter with an enemy type.
- A boss defeated.

Event IDs include `death_first`, cause-specific `death_first_by_<cause_id>` entries, generic `death_first_by_enemy` (keyed by `enemyId`), generic `death_after_boss` (keyed by `bossId` and `causeId`), and the higher-priority `death_after_boss_moder_by_deathsquito` special case. Biome events use `biome_first_<biome_id>`; `dungeon_first`, `enemy_first_encounter`, and `boss_defeated` are reusable events refined by `oncePerKey`. IDs and identifiers must be reused exactly in `data/events.json`, `data/fallback_lines.json`, and related code. Do not add an identifier for a game concept until it has been checked.

`enemy_first_encounter` is a reusable event and must be limited to the curated `notableEnemyIds` allowlist. Do not fire it for every creature. Runtime prefab names are stored separately from display IDs in `prefabNames`; verify them against the supported game version and strip only a trailing `(Clone)` suffix before matching.

### Death cause-to-fact mapping

`data/facts.json` owns the mapping from event cause IDs to fact namespaces. Keep the mapping explicit rather than deriving it from labels:

| Event cause ID | Facts reference |
| --- | --- |
| `tree` | `hazard:treefall` |
| `fall` | `hazard:falling` |
| `burning` | `hazard:fire` |
| `poisoned` | `hazard:poison` |
| `freezing` | `hazard:freezing` |
| `drowning` | `hazard:drowning` |
| `deathsquito` | `enemy:deathsquito` |

### B. Cooldown-gated tactical comments

Repeatable events may fire after a per-event cooldown, for example:

- A verified damage interaction that supports a tactical observation.
- A boat wreck, burned food, snapped fishing line, or death after a long trip, if reliable detection is available.

These are comments by default. For tactical damage reactions, prefer inspecting the current enemy's runtime damage-modifier data rather than duplicating weaknesses/resistances in `facts.json`. The candidate member and values are unverified until checked in dnSpy/ILSpy and recorded in `docs/HOOK_NOTES.md`. Do not infer a weakness, resistance, cause, or mechanic from incomplete event data. Advice is allowed only when backed by a relevant verified local fact or confirmed runtime data.

### C. Ambient comments

Optional low-probability comments use a global cooldown and should not interrupt higher-priority reactions. Ambient comments are not part of the initial playable milestone.

### D. Voice questions

Optional push-to-talk input is transcribed locally with Whisper. The transcript follows the same advice/comment rules as typed or event-driven input. Send a transcript to the chosen provider only when a provider response is needed and the user has enabled that behavior.

## 4. Advice and comment rules

Every generated or selected line has a type: `advice` or `comment`.

- **Advice** is accurate, actionable, and sincere. It must use curated facts that are marked verified for the supported game version. If no applicable verified fact is available, do not generate a game-specific hint.
- **Comments** may carry the selected style and intensity. They may tease in-game choices, but never real-world traits.
- Comments must also avoid unsupported game claims. A biome-, item-, or damage-themed line (for example, an Ashlands/fire-weapon joke) can only be used after its underlying game facts have been verified; otherwise use a non-specific flavor line.
- Style and intensity are context for comments only; neither overrides fact constraints.
- Event `factsOptional` entries are enrichment references, not event prerequisites. Missing or unverified facts never block a comment; advice that depends on them is omitted.
- Cached or live text must not introduce unverified item names, damage properties, numbers, progression claims, or mechanics.
- Prefer a short, safe comment or no line over a guessed hint.

The structured response shape for any live generation is:

```json
{"type":"advice|comment","text":"One short sentence."}
```

The implementation must validate the response type, non-empty text, and length before displaying it. For the first live-provider milestone, live calls should generate comments; advice remains local. A later live advice path may only paraphrase explicitly supplied verified facts and must be discarded when those facts are absent.

## 5. Output modes and toggles

Advice and comments have independent output settings. Each supports:

- `Off`
- `Caption`
- `TTS`
- `Both`

Captions display on the in-game overlay. TTS is optional and local when using the planned Piper helper. Spoken audio output uses WAV or OGG; MP3 is out of scope for the Unity Mono player.

## 6. Spam and latency control

The Event Director applies, in order:

1. Event validity and required-data checks.
2. Once-per-character/world flags and per-event cooldowns.
3. Ambient probability and a global cooldown, where applicable.
4. Priority selection and queueing.
5. Event expiration: drop stale warnings or responses that no longer fit the current situation.

Each event has a `group` (`death`, `biome`, `boss`, `enemy`, `tactical`, `ambient`, or `voice`). For one occurrence, only the highest-priority matching eligible event in each group is displayed. Once-only flags for all matching eligible events are consumed even when their messages are suppressed by a higher-priority match. `dungeon_first` uses the `biome` group as a location-discovery event. Optional `oncePerKey` payload fields refine the character/world flag, such as `bossId`, `enemyId`, `interiorType`, or `biomeId`.

Deaths and major milestones may receive higher priority and longer relevance than short-lived warnings. Exact cooldowns, probabilities, and latency budgets are configuration/data values to tune during playtesting, not assumed game facts.

## 7. Initial release scope and non-goals

The first playable milestone should prove a small vertical slice: plugin loads, selected events are detected, state persists for a character/world, and local captions appear without a provider key. Verify an initial set of 5–10 high-value facts before relying on advice in the first playable version. Expand event coverage only after the corresponding signals are verified.

Not planned for the initial release:

- A spawned follower NPC, companion combat, or autonomous resource gathering.
- Multiplayer RPCs that roast other players.
- A Node.js or Python LLM proxy server.
- MP3 audio output.
- Ambient chatter, every tactical trigger, and every voice feature before the core caption/event path is stable.

Valheim does not have RNG upgrade failures or item destruction during crafting. Do not add triggers or comments about those mechanics.

## 8. State and configuration

First-time flags and relevant statistics are persisted per character and per world, so different characters/worlds do not share milestone history. Persistence format and identifiers must be stable and safe to load when fields are added in later versions.

User-facing settings include enabled state, style, roast intensity, separate advice/comment output modes, provider base URL, model, API key, live-call limits, push-to-talk key, TTS/STT toggles, and caption position/size. The API key belongs in the local BepInEx configuration and must never be logged or committed.

## 9. Compatibility and validation

The target is Valheim on Unity Mono with .NET Framework 4.8, using BepInEx and HarmonyX. Game hook names and event signals are provisional until inspected and tested against the supported game version. No advice should be marked verified merely because it appears in a prompt, fallback line, or generated cache.
