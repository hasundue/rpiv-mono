## [Unreleased]

### Changed
- `saveJsonConfig(path, data)` now returns `boolean` (`true` on success, `false` on filesystem failure). All four current rpiv-* save callers are user-initiated UI actions that show a "Saved …" notification, so the boolean is what every honest caller needs. Callers that genuinely don't care about persistence outcome can discard the return value (TypeScript does not warn on ignored `boolean`).
- `loadJsonConfig` emits `console.warn` on malformed-JSON catch (carried over from `rpiv-i18n`'s prior diagnostic). Ensures silent save + silent load cannot compound into undiagnosable state loss when a config file is corrupted between sessions.
- `loadJsonConfig` now returns `{}` for JSON files whose root value is an array (e.g. `[1, 2, 3]`). Previous behavior cast through the array as `T`, violating the documented "plain object only" contract. No known config file in the rpiv-* family is array-valued; this closes a latent inconsistency.
