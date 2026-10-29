# Valheim AI Companion

An event-driven Valheim commentator and coach for short gameplay reactions and accurate, locally curated hints—with captions, voice, or both.

![Status: early development / not yet released](https://img.shields.io/badge/status-early%20development%20%2F%20not%20yet%20released-orange)

## What it does

Valheim AI Companion is planned as a free, open-source BepInEx mod for Valheim, built in C# for Unity Mono and .NET Framework 4.8. It reacts to your gameplay as an off-screen companion; its focus is commentary and advice, not a follower NPC that fights alongside you.

- Show reactions as captions, text-to-speech (TTS), both, or turn either output off. Advice and comments have separate output settings.
- Choose a companion style: **Friendly**, **Casual**, **Professional**, or **Unhinged**.
- Set roast intensity independently from style, from **1 (gentle)** to **5 (merciless gameplay roast)**.
- React to events such as deaths, first visits to biomes and dungeons, boss victories, new enemy encounters, and tactical mishaps.
- Ask questions using optional push-to-talk voice input. Local Whisper speech-to-text (STT) is planned as an optional helper.
- Use optional local Piper TTS for spoken output; audio output is intended to use WAV or OGG.

### The companion's rule

**Advice stays accurate and sincere. Comments carry the tone.** Advice will be based on curated local game data, not improvised game facts. Dark humor can roast the player's in-game decisions, never real-world traits.

## Personality styles

Examples are flavor lines and avoid claims about specific game mechanics or item properties.

| Style | Voice | Example |
| --- | --- | --- |
| Friendly | Warm, encouraging, and supportive. | “That was a rough moment. Take a breath and give it another go.” |
| Casual | Relaxed gaming buddy with light humor. | “Well, that plan had confidence. Let’s workshop the next one.” |
| Professional | Neutral, concise, and matter-of-fact. | “You have been defeated. Review the situation before continuing.” |
| Unhinged | Sharp, darkly comic roasts aimed only at gameplay. | “That strategy has been reported missing. Last seen near your last decision.” |

## Bring your own key (BYOK)

The mod is intended to call your selected provider directly; there is no Node or Python proxy server. To enable live AI responses, paste your own key into the plugin's local BepInEx configuration, expected at `BepInEx/config/ValheimAICompanion.cfg`, under `[LLM]` → `ApiKey`. The exact setting name and location may change during development. Never commit the real config file.

The provider base URL is configurable. Planned compatibility includes:

- OpenAI-compatible API endpoints
- Groq
- OpenRouter
- Gemini through an OpenAI-compatible endpoint
- Local Ollama or LM Studio endpoints

Fast, small, non-reasoning models are preferred to keep responses quick and inexpensive. Possible starting points include **GPT-4o mini**, **Gemini Flash** variants, **Llama 3.1 8B Instruct** through a compatible provider, or a small local instruct model in Ollama/LM Studio. Model names and availability change; check the provider's current documentation and use a model that supports the configured API format.

## Cost and token use

The design aims for roughly **90% cached lines** from pre-generated pools and **10% live calls**. Live prompts should contain only compact event details and a few relevant facts, with a response cap of about **50 tokens**. Advice comes from local data. A configurable per-hour call cap, per-session limits, and an in-game usage meter are planned so you can monitor and limit live usage.

A typical session should cost pennies, but actual cost depends on your provider, model, usage, and current pricing. Local models can avoid provider API charges, though they still use your own hardware and electricity.

## Privacy and security

- Only compact event text and, when you ask a voice question, its transcript are intended to be sent to the provider you configure. Raw microphone audio is intended to be transcribed locally with optional Whisper.
- Optional Piper speech output is local. Per-character and per-world flags and stats are stored locally.
- The API key is never logged by design. Treat the BepInEx configuration as sensitive because it may contain the key in plain text.
- **Never share your config file or paste it into Discord, GitHub issues, or bug reports.** Remove secrets before sharing logs or other files.

## Installation

**Coming soon — the mod is not released yet.** r2modman and Thunderstore installation instructions will be added when a tested release is available.

Planned install flow:

1. Find **Valheim AI Companion** on Thunderstore and install it through r2modman.
2. Launch Valheim once through your r2modman profile so BepInEx can create its configuration.
3. Configure outputs and style. Add an API key only if you want to use a live provider.

## Configuration

Names below describe the planned settings and may change before release.

| Option | Purpose |
| --- | --- |
| Advice output | `Off`, `Caption`, `TTS`, or `Both` for hints. |
| Comment output | `Off`, `Caption`, `TTS`, or `Both` for reactions. |
| Style | `Friendly`, `Casual`, `Professional`, or `Unhinged`. |
| Roast intensity | Integer from `1` (gentle) to `5` (strong gameplay-only roast). |
| Provider base URL | OpenAI-compatible API endpoint to call. |
| Model | Model identifier accepted by the selected endpoint. |
| API key | Your provider key, stored in the local BepInEx config. |
| Push-to-talk key | Key used to record a voice question. |
| TTS | Enable or disable spoken output. |
| Hourly call cap | Maximum live provider calls allowed per hour. |
| Caption position / size | On-screen placement and text size for captions. |

## Roadmap

Development is planned to start with project data and prompts, then move to the C# plugin once the game development environment is ready. The roadmap covers event hooks, captions, persisted tracking, direct provider calls, cached pools, and optional local voice features.

See [docs/ROADMAP.md](docs/ROADMAP.md) for the full checklist.

## Contributing

Contributions are welcome as the project takes shape. Please keep game claims grounded in verified sources, preserve the rule that advice is accurate and sincere, and never include API keys or local config files in commits. See the [roadmap](docs/ROADMAP.md) for the current focus and the [hook research notes](docs/HOOK_NOTES.md) for game-version-specific investigation; contribution and development guides will be expanded as the codebase grows.

## License and disclaimer

This project is licensed under the [MIT License](LICENSE).

Valheim AI Companion is an unofficial fan-made mod. It is not affiliated with, endorsed by, or sponsored by Iron Gate Studio or Coffee Stain Publishing. Valheim is a trademark of its respective owners.
