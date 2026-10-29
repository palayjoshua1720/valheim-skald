# Valheim AI Companion: Project Brief and Scaffold Prompts

How to use this file:

1. Create a new repo folder, e.g. `E:\dev\ValheimAICompanion`, and open it in VS Code.
2. Save this file in the repo root.
3. In the GPT chat, paste **Section 2 (Shared Context)** first, once per chat session.
4. Then paste **one prompt from Section 4 at a time**, review the output, and save it to the file path shown in the prompt heading.
5. Do not let the AI invent Valheim facts. Everything it writes in `facts.json` must be checked against the Valheim wiki or the game before you trust it.

---

## 1. Project Description

**Name (working):** Valheim AI Companion

**What it is:** A free, open-source BepInEx mod for Valheim. It adds an AI companion that watches what the player does and reacts with short comments and hints. Reactions appear as an on-screen caption overlay, as text-to-speech voice, or both.

**What it is for:**

- Make solo and co-op play funnier and more alive, with an AI that roasts your deaths and comments on your progress.
- Give accurate hints when the player enters a new biome, a dungeon, or meets a new enemy for the first time.
- Let the player talk to it with push-to-talk and ask questions like "what's weak to fire here?"

**Key features:**

- **Output modes:** Off / Caption / TTS / Both, configurable separately for advice and for comments.
- **Style options:** Friendly, Casual, Professional, Unhinged.
- **Roast intensity slider** (1 to 5), which is separate from the style.
- **Core rule:** advice is always accurate and sincere. Comments carry the attitude, from friendly up to dark and unhinged. Dark humor targets the player's gameplay, never real-world traits.
- **Event reactions:**
  - death roasts based on cause and history, such as the first death by Deathsquito right after killing Moder, the first treefall death, or the first drowning
  - first time in a biome
  - first dungeon entry
  - boss defeated
  - first time meeting an enemy type
  - tactical irony, such as using fire on a fire-immune enemy
- **Ashlands-style examples:** comments can mention biome-appropriate gear (e.g. fire-based weapons) with the facts coming from curated data, not from the AI's memory.

**BYOK (bring your own key):**

- The mod is free. Users paste their own LLM API key into the BepInEx config.
- The mod calls the provider directly from the game with `UnityWebRequest`. There is no proxy server.
- Support any OpenAI-compatible endpoint through a configurable base URL (OpenAI, Groq, OpenRouter, Gemini compat, local Ollama or LM Studio), plus an optional native Anthropic adapter.

**Design goals:**

1. **Lowest possible token use.** About 90% of lines come from pre-generated cached pools, about 10% are live calls. Live calls use tiny compact prompts and `max_tokens` of about 50.
2. **Feels realtime.** Stream responses, show captions as tokens arrive, use fast small models, pre-generate during loading and death screens, drop stale responses.
3. **Accurate advice.** Advice comes from curated `facts.json`. The LLM only adds tone.
4. **Safe and private.** Never log or commit API keys. Only compact event text and voice transcripts are sent to the user's chosen provider. Local STT (Whisper) and local TTS (Piper) are free.

**Tech stack:**

- C#, .NET Framework 4.8 (Valheim runs on Unity Mono)
- BepInEx + HarmonyX, with Jotunn optionally for config and GUI helpers
- `UnityWebRequest` for HTTP and SSE streaming
- Local Whisper (whisper.cpp or faster-whisper) for STT, optional helper process
- Local Piper for TTS, optional helper process, WAV or OGG output (avoid MP3 in Unity Mono)

**State tracking:** first-time flags and death stats are saved **per character and per world** as JSON, so a new character gets first-time reactions again.

---

## 2. Shared Context (paste this first in every new GPT chat)

```text
You are helping me scaffold a GitHub repo for a Valheim mod called "Valheim AI Companion".

Summary:
- Free, open-source BepInEx/HarmonyX mod written in C# for Valheim (Unity Mono, .NET Framework 4.8).
- An AI companion reacts to gameplay events (deaths, first biome visits, dungeons, bosses, new enemies, tactical mistakes) with short comments and accurate hints.
- Output via caption overlay, TTS, or both, with separate toggles for advice and comments.
- Styles: Friendly, Casual, Professional, Unhinged. Separate roast intensity 1-5.
- Core rule: ADVICE is always accurate and sincere at every style. COMMENTS carry the tone. Dark humor targets the player's gameplay only, never real-world traits.
- BYOK: users paste their own API key. The mod calls an OpenAI-compatible endpoint directly (configurable base URL). No Node/Python proxy.
- Token minimum: ~90% of lines come from pre-generated cached pools, ~10% live calls, max_tokens ~50, compact event payloads, advice served from local data.
- Local STT (Whisper) and local TTS (Piper) are optional helper components. Output WAV or OGG, not MP3.
- Flags and stats are stored per character and per world.

Rules for everything you write:
1. Never invent Valheim facts. If you are not sure about a game fact (damage weaknesses, item names, boss order, mechanic details), add "verified": false so I can check it against the wiki. Prefer fewer correct entries over many guessed ones.
2. Valheim has no RNG upgrade failures or item destruction on crafting. Do not reference them.
3. Never include real API keys. Use placeholders like "YOUR_API_KEY_HERE".
4. Keep files clean, commented where useful, and consistent with each other (same event IDs, same style names, same biome IDs everywhere).

Reply "ready" and wait for my first file request.
```

---

## 3. Target Repo Structure

```
ValheimAICompanion/
├── README.md
├── LICENSE
├── .gitignore
├── PROJECT_BRIEF_AND_PROMPTS.md
├── docs/
│   ├── SPEC.md
│   ├── ARCHITECTURE.md
│   ├── HOOK_NOTES.md
│   └── ROADMAP.md
├── data/
│   ├── facts.json
│   ├── events.json
│   └── fallback_lines.json
├── prompts/
│   ├── system_base.txt
│   ├── style_friendly.txt
│   ├── style_casual.txt
│   ├── style_professional.txt
│   ├── style_unhinged.txt
│   ├── intensity.txt
│   └── pool_generation.txt
├── config/
│   └── ValheimAICompanion.cfg.example
└── prototype/            (optional, web-dev-style script to test LLM logic before porting to C#)
    ├── package.json
    └── src/
```

Not in the repo for now: the C# plugin project. That comes once Valheim, r2modman, Visual Studio and the Jotunn stub are set up.

---

## 4. File Prompts

Paste them in this order. Each heading shows the file path to save to.

### 4.1 `.gitignore`

```text
Create a .gitignore for this repo. It must include the standard Visual Studio / .NET template (bin/, obj/, .vs/, *.user, *.suo, packages/, etc.), plus:
- Node (node_modules/, .env, dist/) for the optional prototype folder
- Valheim game DLLs and BepInEx output (assembly_*.dll, UnityEngine*.dll, BepInEx/, *.pdb where appropriate)
- Local config files that could contain API keys: *.cfg (but NOT *.cfg.example), .env, secrets.json
- Downloaded models: *.bin, *.onnx, *.gguf, models/
- OS junk (.DS_Store, Thumbs.db)
Output only the file content.
```

### 4.2 `LICENSE`

```text
Create an MIT LICENSE file for the year 2026 with the copyright holder "YOUR NAME HERE". Output only the file content.
```

### 4.3 `README.md`

```text
Write README.md for the repo. Audience: Valheim players who install mods with r2modman, and developers who may contribute. Include:

1. Title, one-line pitch, and a "Status: early development / not yet released" badge-style line.
2. What it does (feature list: output modes, styles, roast intensity, event reactions, voice input).
3. The core rule: advice is accurate, comments carry the tone, dark humor targets gameplay only.
4. Style table: Friendly, Casual, Professional, Unhinged, each with a one-line description and an example line. Examples must be Valheim-appropriate and must not state unverified game facts.
5. BYOK setup: where to paste the API key (BepInEx config), configurable base URL, list of supported providers (OpenAI-compatible endpoints, Groq, OpenRouter, Gemini compat, local Ollama/LM Studio), suggested fast small models.
6. Cost and token philosophy: cached line pools, ~50 token cap, per-hour call cap, usage meter. Say a typical session should cost pennies, but do not promise exact numbers.
7. Privacy and security: only compact event text and voice transcripts go to the chosen provider, local STT/TTS option, warning to never share config files or paste them in Discord or bug reports, key is never logged.
8. Installation (placeholder steps for r2modman / Thunderstore, mark as "coming soon").
9. Configuration options table (output mode for advice, output mode for comments, style, intensity, provider base URL, model, API key, push-to-talk key, TTS on/off, hourly call cap, caption position/size).
10. Roadmap summary linking to docs/ROADMAP.md.
11. Contributing, license (MIT), and a disclaimer: unofficial fan mod, not affiliated with Iron Gate or Coffee Stain.

Tone: clear, friendly, a little fun. Output only the file content in Markdown.
```

### 4.4 `docs/SPEC.md`

```text
Write docs/SPEC.md: the product spec. Sections:

1. Vision.
2. Personality system: four styles (Friendly, Casual, Professional, Unhinged) plus a separate roast intensity 1-5. Explain that style = voice, intensity = how harsh the comments get, and that advice is always accurate and sincere regardless of both. Include a table of what each style sounds like with Valheim-flavored sample lines (no unverified facts).
3. Trigger categories:
   A. First-time milestones (one-time, flagged per character and world): first death overall, first death by each cause (Tree, Drowning, Fall, Burning, Freezing, Poisoned, specific creatures like Deathsquito), first qualifying death within a configured window after a boss kill (including the planned Deathsquito-after-Moder example), first time entering each biome, first dungeon/interior entry, first time meeting each enemy type, boss defeated. Track boss-defeat timestamps per character/world; verify the event signals against the current game version.
   B. Cooldown-gated tactical comments (repeatable, e.g. 5-10 minute cooldown): hitting an enemy with a damage type it is immune or heavily resistant to, boat wrecked, food burned on a cooking station, fishing line snapped, dying right after a long portal run or long trip.
   C. Ambient random comments: probability roll plus global cooldown.
   D. Voice questions via push-to-talk.
4. Rules for advice vs comment, and the required structured LLM output {"type":"advice"|"comment","text":"..."}.
5. Output modes and toggles.
6. Spam control: global cooldown, per-event cooldown, probability, priority queue, stale event dropping.
7. Non-goals for v1: multiplayer RPC roasting of other players, a proxy server, MP3 audio.
8. Explicitly note that Valheim has no RNG upgrade failures, so "failed gamble" triggers are replaced by the Valheim-native ones above.

Output only the file content in Markdown.
```

### 4.5 `docs/ARCHITECTURE.md`

```text
Write docs/ARCHITECTURE.md describing the technical design. Include:

1. Overview diagram in ASCII:
   Game event / mic input -> Event Director (flag, cooldown, probability, priority) -> Pool hit? yes -> show line (free) / no -> Prompt Builder -> direct LLM call (<=50 tokens, streamed) -> Output Router (caption overlay / TTS / both). Mic -> local Whisper -> text -> same path.
2. Components: Harmony patches (event emitters), Event Director, State Store (per character + world JSON), Facts Provider (reads data/facts.json), Pool Manager (pre-generate and refill cached lines), Prompt Builder, LLM Client (UnityWebRequest, OpenAI-compatible, optional Anthropic adapter, SSE streaming), Output Router, Caption Overlay (Unity canvas), TTS player (AudioSource, WAV/OGG), STT helper (optional local process).
3. Hook list with the Valheim methods to investigate (clearly marked "verify in dnSpy/ILSpy against the current game version"): Player.OnDeath, HitData hit types and GetAttacker, biome change tracking, interior/dungeon detection, boss death and global keys, Character.Awake or SetTarget for first-encounter.
4. Token-saving strategy: cached pools (about 90%), advice from local data, compact event payload format (example: "E=death cause=Deathsquito n=14 prev=Moder@2m FACTS=..."), only 1-3 relevant facts injected, max_tokens ~50, no chat history, anti-repetition via first words of last 3 lines, static prompt first / dynamic part last, per-session and per-hour call caps.
5. Realtime strategy: streaming captions, sentence-level TTS streaming, prefetch around loading/death screens, canned filler line for slow calls, drop stale responses.
6. Failure handling: 401, 429, timeouts, refusals (fall back to cached pool or hardcoded line), one-time in-game notice, never spam errors.
7. Security: plaintext BepInEx config warning, never log keys, optional DPAPI encryption on Windows.
8. Threading rule: never block the main thread, marshal async results back to it.
9. Notes: reasoning models can burn max_tokens on hidden thinking, so recommend non-reasoning small models; MP3 is problematic in Unity Mono.
10. Create docs/HOOK_NOTES.md as a versioned investigation log with a table for game version, candidate type/method, dnSpy/ILSpy confirmation, runtime test result, quirks, and verification status. Mark all candidate hooks unverified until inspected and tested.

Output only the file content in Markdown.
```

### 4.6 `docs/ROADMAP.md`

```text
Write docs/ROADMAP.md as a checklist-style roadmap using this MVP order:
1. BepInEx plugin that logs death, biome and boss events (hello world + first Harmony patch).
2. Caption overlay with hardcoded lines.
3. Persisted first-time and death stats tracker (per character + world), including boss-defeat timestamps for the configured post-boss death window.
4. Direct LLM call with styles and roast intensity.
5. Pool pre-generation, caching and refill.
6. Local TTS.
7. Local STT push-to-talk and voice questions.
8. Ambient comments, tactical irony triggers, config UI, usage meter.
9. Later ideas: multiplayer roasts via RPC, more languages, community fact packs, Thunderstore release.
Use Markdown checkboxes. Add a short "Current focus" section at the top that says: data files and prompts first, C# plugin once the dev environment is ready. Include a task to verify 5-10 high-value starter facts before relying on advice, and a release check to update docs/HOOK_NOTES.md. Output only the file content.
```

### 4.7 `data/facts.json`

```text
Create data/facts.json: curated Valheim facts that will be injected into prompts so advice stays accurate.

Structure:
{
  "schemaVersion": 2,
  "gameVersionChecked": "UNKNOWN - fill in",
  "causeMap": {
    "tree": "hazard:treefall",
    "fall": "hazard:falling",
    "burning": "hazard:fire",
    "poisoned": "hazard:poison",
    "freezing": "hazard:freezing",
    "drowning": "hazard:drowning",
    "deathsquito": "enemy:deathsquito"
  },
  "biomes": {
    "<biome_id>": {
      "displayName": "...",
      "order": null,
      "bossOrGate": null,
      "dangers": [{"id":"...","text":"...","verified":false,"source":"","gameVersion":""}],
      "tips": [{"id":"...","text":"...","verified":false,"source":"","gameVersion":""}],
      "recommendedGear": [{"id":"...","text":"...","verified":false,"source":"","gameVersion":""}],
      "verified": false
    }
  },
  "enemies": {
    "<enemy_id>": {
      "displayName": "...",
      "prefabNames": [],
      "biomes": ["..."],
      "tips": [{"id":"...","text":"...","verified":false,"source":"","gameVersion":""}],
      "recommendedGear": [{"id":"...","text":"...","verified":false,"source":"","gameVersion":""}],
      "verified": false
    }
  },
  "hazards": {
    "<hazard_id>": { "displayName": "...", "tips": [{"id":"...","text":"...","verified":false,"source":"","gameVersion":""}], "verified": false }
  }
}

Cover these biomes: Meadows, Black Forest, Swamp, Mountain, Plains, Mistlands, Ashlands, Deep North. Cover hazards: treefall, drowning, falling, cold, freezing, poison, fire, plus boats and cooking if you know something solid. Cover enemies including Deathsquito, Skeleton, Draugr, Greydwarf, Troll, Fuling, Lox, Wolf, Serpent, and the main enemies of each biome you are sure about.

Rules:
- Use stable snake_case display/concept IDs. Store actual internal prefab names separately in `prefabNames` only after checking them in the supported game version; do not assume display names are prefab names.
- Each `dangers`, `tips`, and `recommendedGear` fact is an object with a stable `id`, short `text`, `verified`, `source`, and `gameVersion`. Leave arrays empty rather than adding unchecked claims. Every entity starts as `verified: false`.
- Do not store weaknesses/resistances as hand-copied facts. Investigate whether the current enemy instance exposes damage modifiers; confirm any proposed member/enum names against the game assemblies first.
- Add no game-fact content unless checked against a trusted current source. For a first playable version, verify a small starter set of high-value facts and record their source/version.
- Keep each tip to one short sentence.
- Do not guess item stats or numbers.
Output only valid JSON.
```

### 4.8 `data/events.json`

```text
Create data/events.json: the list of trigger events. Structure:

{
  "schemaVersion": 2,
  "events": [
    {
      "id": "death_first_by_deathsquito",
      "category": "first_time" | "tactical" | "ambient" | "voice",
      "group": "death" | "biome" | "boss" | "enemy" | "tactical" | "ambient" | "voice",
      "trigger": "plain-English description of when it fires",
      "oncePer": "character_world" | "never" ,
      "oncePerKey": ["payloadFieldName"],
      "cooldownSeconds": 0,
      "probability": 1.0,
      "priority": 1-10,
      "maxLatencyMs": how long the line stays relevant before being dropped,
      "usePool": true,
      "allowLiveCall": true,
      "factsOptional": ["enemy:deathsquito"],
      "payloadFields": ["cause", "totalDeaths", "previousBossKilled", "secondsSinceBoss"],
      "notes": "..."
    }
  ]
}

Include at least:
- first death overall, first death by Tree, Drowning, Fall, Burning, Freezing, Poisoned, first death by Deathsquito (with a field for "died within N minutes of killing Moder"), repeated deaths (3rd, 5th, 10th death milestone)
- generic first death by enemy (`death_first_by_enemy`, keyed by `enemyId`) and generic death after boss (`death_after_boss`, keyed by `bossId` and `causeId`); keep the Moder/Deathsquito event as the higher-priority special case
- first entry into each of the 8 biomes
- first dungeon/interior entry (e.g. crypts, mines, caves, if applicable)
- each boss defeated
- first time meeting an enemy type (generic event reused per enemy)
- tactical: hit enemy with a damage type it resists heavily, boat wrecked, food burned on cooking station, fishing line snapped, died soon after long trip/portal
- ambient random comment
- voice question

Every event must have a group. Within each group and source occurrence, display only the highest-priority matching event, but consume once-only flags for all matching events. Use `oncePerKey` to scope flags per boss, enemy, biome, or interior type. Limit `enemy_first_encounter` to a curated notable-enemy allowlist rather than every creature. `factsOptional` is enrichment only: missing/unverified facts never block comments; add advice only when its required fact objects are verified. Death causes map through `facts.json` `causeMap`. Deaths and boss events should be high priority with longer relevance. Warnings/hints should have short maxLatencyMs so late ones get dropped. Output only valid JSON.
```

### 4.9 `data/fallback_lines.json`

```text
Create data/fallback_lines.json: hardcoded offline fallback lines, used when there is no API key, the call fails, or the provider refuses.

Structure:
{
  "schemaVersion": 1,
  "lines": {
    "<event_id>": {
      "friendly": ["...", "...", "..."],
      "casual": ["...", "...", "..."],
      "professional": ["...", "...", "..."],
      "unhinged": ["...", "...", "..."]
    }
  }
}

Use the event ids from data/events.json (death_first_by_deathsquito, death_first_by_tree, death_first_by_drowning, death_first_by_fall, biome_first_<id>, boss_defeated_<id>, and so on). 3 lines per style per event, each one sentence, 20 words max.

Rules:
- Comments carry the tone. If a line includes advice, it must be generic and safe (e.g. "bring a shield", "watch your stamina in water") and not a specific enemy weakness claim unless it is obviously correct.
- Unhinged lines can be savage and dark but roast the player's gameplay only, never real-world traits.
- Professional lines are neutral and wiki-like, with no jokes.
- No two lines should share the same opening words.
Output only valid JSON.
```

### 4.10 `prompts/system_base.txt`

```text
Write prompts/system_base.txt: the static base system prompt sent to the LLM on every live call. It must stay under about 100 tokens because it is paid for on every call. Requirements:
- You are a Valheim companion reacting to a game event.
- Output ONLY valid JSON: {"type":"advice"|"comment","text":"..."}.
- One sentence, max 20 words.
- Use ONLY the facts in FACTS for any game claim. Never invent items, weaknesses, or numbers. If FACTS is empty, give a comment, not advice.
- Advice is always accurate and sincere. Comments take the tone.
- Style and intensity placeholders: {STYLE_BLOCK} and {INTENSITY_LINE}.
- Never roast real-world traits (only in-game behavior).
Also include one example of the compact input format, using placeholders rather than an unchecked game claim:
E=death cause=<cause_id> count=<death_count> FACTS=<verified_fact_ids>
Output only the file content.
```

### 4.11 `prompts/style_*.txt` (do this prompt four times, once per style)

```text
Write prompts/style_<STYLE>.txt for the style "<STYLE>" (one of: friendly, casual, professional, unhinged). It is a short block (40-70 words) inserted into the system prompt at {STYLE_BLOCK}. Describe the voice, vocabulary, and what is allowed or forbidden.

- friendly: warm, encouraging, comforting after deaths.
- casual: relaxed gaming buddy, light slang, mild humor.
- professional: neutral wiki-style, zero jokes or roleplay, short factual statements.
- unhinged: savage, sarcastic, dark humor, laughs at failures, roasts gameplay only, but any advice must remain accurate and sincere.

Output only the file content.
```

### 4.12 `prompts/intensity.txt`

```text
Write prompts/intensity.txt: five one-line modifiers for roast intensity 1 to 5, formatted as "1: ...", "2: ...", etc. They are inserted at {INTENSITY_LINE}. 1 = very gentle teasing, 3 = playful roasting, 5 = merciless dark roast of in-game behavior only. Each line must be under 20 words and must not override the rule that advice stays accurate. Output only the file content.
```

### 4.13 `prompts/pool_generation.txt`

```text
Write prompts/pool_generation.txt: the prompt used to batch-generate cached lines for one event, one style, and one intensity. It must:
- Accept placeholders: {EVENT_ID}, {EVENT_DESCRIPTION}, {STYLE_BLOCK}, {INTENSITY_LINE}, {FACTS}, {COUNT}.
- Ask for {COUNT} distinct lines as a JSON array of {"type":"advice"|"comment","text":"..."}.
- Require variety (different openings and angles), max 20 words each, use only FACTS for game claims.
- Forbid real-world-trait insults.
- Output ONLY JSON.
Output only the file content.
```

### 4.14 `config/ValheimAICompanion.cfg.example`

```text
Create config/ValheimAICompanion.cfg.example in BepInEx .cfg format (sections in [brackets], "## description" comment lines above each setting, "Key = value" lines). Include sections and settings for:
- [General]: Enabled, Style (Friendly/Casual/Professional/Unhinged), RoastIntensity (1-5)
- [Output]: AdviceOutput (Off/Caption/TTS/Both), CommentOutput (Off/Caption/TTS/Both)
- [LLM]: ApiBaseUrl (default https://api.openai.com/v1), ApiKey = YOUR_API_KEY_HERE, Model, MaxTokens = 50, TimeoutSeconds
- [Budget]: MaxLiveCallsPerHour = 20, MaxLiveCallsPerSession, AmbientCommentChance, GlobalCooldownSeconds, PoolSizePerEvent
- [Voice]: PushToTalkKey, EnableSTT, EnableTTS, WhisperModelPath, PiperVoicePath
- [Caption]: Position, FontSize, DurationSeconds, Opacity
Add a comment at the very top: "NEVER share this file publicly. It contains your API key." Output only the file content.
```

### 4.15 `prototype/` (optional, do this after the files above)

```text
Create a small Node.js + TypeScript prototype in prototype/ that tests the LLM logic outside the game, so I can port it to C# later. Files:
- package.json (scripts: dev, start), tsconfig.json
- src/config.ts: reads provider base URL, model and API key from environment variables (.env), never hardcoded
- src/state.ts: per-character-and-world first-time flags and death stats stored in a JSON file
- src/director.ts: decides whether an event fires, using flags, cooldowns, probability and priority from data/events.json
- src/pool.ts: loads cached pools from disk, picks a line, reports when a pool is low, refills it using prompts/pool_generation.txt
- src/prompt.ts: builds the compact prompt from prompts/*.txt, the style, the intensity, and 1-3 relevant facts from data/facts.json
- src/llm.ts: calls any OpenAI-compatible endpoint with fetch, max_tokens 50, parses the JSON reply, handles 401/429/timeout/refusal by falling back to data/fallback_lines.json
- src/cli.ts: a command-line simulator where I can type fake events like "death cause=Deathsquito prev=Moder@2m" and see the result, with a token usage printout
Keep each file small and well commented, since I am a web dev learning to port this to C#.
```

---

## 5. After the Files Exist

1. Read through every file once. Fix anything that sounds wrong.
2. Check every `facts.json` entry against the Valheim wiki and flip `"verified"` to `true` as you confirm it.
3. Commit with a clear first message, e.g. `Initial project docs, data schemas and prompts`.
4. When you're home with the game, r2modman and Visual Studio set up, add the C# plugin project and start with the hello world and the first death hook.

## 6. Quick Pitfalls

- If GPT writes a fact you don't recognize, leave it unverified or delete it. A wrong tip in an "accurate advice" mod hurts more than a missing one.
- Keep event ids identical across `events.json`, `fallback_lines.json` and the prototype. A mismatch there is the most common bug.
- Before every commit, check `git status` for `*.cfg` files and `.env`. They must never be committed.
- Check your employer's policy before pushing personal projects from a work machine.
