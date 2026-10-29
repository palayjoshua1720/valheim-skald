# Roadmap

## Current focus

Create and review the project data files, prompts, and documentation first. Keep unchecked Valheim facts marked unverified. Start the C# plugin when the Valheim development environment is ready at home.

## MVP

- [ ] **Plugin foundation:** Create a BepInEx plugin that loads and logs startup successfully; verify the target game/BepInEx setup.
- [ ] **First event signals:** Log death, biome-entry, and boss events. Investigate each hook against the current game version before depending on it.
- [ ] **Caption overlay:** Display short hardcoded local lines for selected events, with no API key required.
- [ ] **State tracking:** Persist first-time flags and death statistics per character and world; verify reload and isolation behavior.
- [ ] **Boss-timed death milestone:** Record boss-defeat timestamps per character/world and add a first qualifying death event within a configurable window. Start with the planned Deathsquito-after-Moder showcase; confirm cause attribution and event timing in-game.
- [ ] **Starter fact set:** Select and verify 5–10 high-value facts, prioritizing common biome dangers and frequently encountered enemies. Record the source and checked game version; leave all other entries unverified.
- [ ] **Verified advice:** Add only confirmed facts to local data and serve advice locally. Keep comments separate from advice. Game-themed comments (including biome, item, or damage references) also require verification of their underlying claims.
- [ ] **Configuration and personality:** Add independent advice/comment output modes, the four styles, and roast intensity 1–5. Confirm style/intensity never alter advice accuracy.
- [ ] **Direct LLM comments:** Add one OpenAI-compatible provider path using the player's key and configurable base URL. Use compact event payloads, asynchronous requests, a roughly 50-token response cap, safe validation, and local fallbacks.
- [ ] **Budget and failure behavior:** Enforce session/hourly call caps, track usage, and handle authentication errors, rate limits, timeouts, malformed output, and refusals without disrupting the game.
- [ ] **Cached pools:** Pre-generate and cache varied comments, add anti-repetition, and refill pools in a controlled way. Keep generated facts out of advice unless verified.
- [ ] **Event coalescing:** Dispatch only the highest-priority matching event per group and source occurrence, while consuming all matching eligible once-only flags. Verify `oncePerKey` state is isolated per boss, enemy, biome, and interior type.

## Optional voice features

- [ ] **Local TTS:** Add optional Piper output and play WAV or OGG audio.
- [ ] **Local STT and questions:** Add push-to-talk with optional local Whisper transcription. Send transcripts to the configured provider only when needed and enabled.

## Later improvements

- [ ] Add ambient comments with probability and global cooldown controls.
- [ ] Add tactical reactions only where event signals are verified. Investigate runtime enemy damage-modifier data before implementing resistance comments; record the confirmed member and values in `docs/HOOK_NOTES.md` rather than hand-copying weaknesses.
- [ ] Add configuration UI and refine the usage meter.
- [ ] Expand supported provider adapters and test compatibility across endpoints.
- [ ] Consider multiplayer reactions through RPC, additional languages, community fact packs, and a Thunderstore release process.

## Release checks

- [ ] Test from a clean r2modman profile on the supported game version.
- [ ] Verify captions and offline fallback behavior without an API key.
- [ ] Verify the config example is safe to share and that keys never appear in logs.
- [ ] Check that each data event ID matches across event definitions, fallback pools, and code.
- [ ] Review every game-specific advice fact against a trusted current source before setting `verified` to `true`.
- [ ] Update [docs/HOOK_NOTES.md](HOOK_NOTES.md) with the supported game version, inspected method signatures, runtime observations, and quirks.
- [ ] Document known limitations and installation/configuration steps before publishing.
