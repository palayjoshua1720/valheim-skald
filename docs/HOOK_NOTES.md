# Valheim Hook Notes

Use this log to record game-version-specific inspection and runtime results for candidate hooks. Names listed here are investigation leads, **not confirmed APIs**. Do not rely on a hook until its actual type, method signature, call timing, and behavior have been checked in the supported game version.

## Inspection log

| Game version | Candidate type / method | dnSpy / ILSpy confirmation | Runtime test result | Quirks / notes | Status |
| --- | --- | --- | --- | --- | --- |
| TBD | `Player.OnDeath` | Not inspected | Not tested | Candidate for death observation; cause attribution needs separate confirmation. | Unverified |
| TBD | `HitData` / attacker lookup (`GetAttacker`) | Not inspected | Not tested | Candidate context only; confirm actual signatures and when attacker data is populated. | Unverified |
| TBD | Biome transition signal (method TBD) | Not inspected | Not tested | Find and confirm a reliable entry/change signal. | Unverified |
| TBD | Dungeon/interior entry signal (method TBD) | Not inspected | Not tested | Confirm which interior types can be distinguished, if needed. | Unverified |
| TBD | Boss death / global-key signal (method TBD) | Not inspected | Not tested | Confirm event timing and how to associate the relevant boss identifier. | Unverified |
| TBD | `Character.Awake` or `Character.SetTarget` | Not inspected | Not tested | Candidate first-encounter signal; confirm it distinguishes a new encounter from ordinary lifecycle/target changes. | Unverified |
| TBD | `Character.m_damageModifiers` (candidate member) | Not inspected | Not tested | Hypothesis only: confirm the member, access path, modifier enum/value names, and runtime values before using it for resistance reactions. | Unverified |
| TBD | Enemy prefab `name` normalization | Not inspected | Not tested | Confirm clone-name shape in the supported version. Matching removes only a trailing `(Clone)`; prefab aliases remain explicit in `prefabNames`. | Unverified |

## What to record when verifying a hook

For each supported game version, update the table with:

- The exact game version and assembly inspected.
- The declaring type, exact method name, and full method signature.
- What the method does that is relevant to the event, based on inspection.
- The Harmony patch point used and what was confirmed during an in-game test.
- Any timing, null-data, repeated-call, multiplayer, or lifecycle quirks observed.
- Whether the hook is verified for production use, still under investigation, or no longer applicable.

Keep observed facts separate from assumptions. If a game update changes an assembly or the test result, add a new version-specific row or note rather than silently reusing an older confirmation.
