# Experience: the GIVEL bot loop, 2026-10-25 (keys, free tiers, laws, deploy)

One day of turning the GIVEL PHUKET lead bot (@givel_phuket_bot) from a
hardcoded host into law-driven infrastructure. Everything below was MEASURED
on the live coding endpoint and the VPS, not inferred. Written so the next
agent does not re-derive it. Refs #41, PRs #45, #48, #49, #50, #51, #52.

## The discovery that changed the design: FREE tiers

z.ai prices `glm-4.7-flash` and `glm-4.5-flash` as **Free** (all four columns).
Measured consequence on the coding endpoint: a **zero-balance key gets 1113 on
paid models but 200 on the free ones**. A key without balance is not dead --
it is a free-tier worker with its own rate limit (1302 = this key throttled,
1305 = the service is busy). Twelve such keys were found on the owner's Mac;
each one multiplies free capacity.

**Standing owner rule (2026-10-25): never recommend a top-up.** The design is
free tiers + one funded coding key (which serves the paid ladder). The host
enforces this with key tiers: a key that fails 1113/1210 on a paid model is
marked free-only (seeded from the nightly bench, demoted live, upgraded back
automatically on a paid-model 200 -- a future top-up self-applies with zero
deploy).

## Outcome classes (coding endpoint)

| code | meaning | host action |
|---|---|---|
| 200 | answer | accept; empty content = failure |
| 1113 | key valid, no resource package | rotate; key serves free tiers only |
| 1210 | seen on paid models, zero-balance keys | rotate like 1113 |
| 1302 | free tier, THIS key throttled | rotate; key stays in pool |
| 1305 | free tier, service busy | rotate; global, not per-key |
| 1311 | concurrency | rotate |

**Reasoning-drain trap**: glm-5.3-flash with thinking on sometimes burns ALL
max_tokens into `reasoning_content` and returns empty `delta.content`. An
empty answer is a failure -- rotate. Fix shipped as law: `THINKING_DISABLED`
(398 reasoning chunks / 0 content at 400 tokens -> with thinking off, first
token 3.3 s, full answer 4.3 s).

## The gen-js boundary (bit us twice)

`t27c gen-js` emits **consts only, never function bodies**. Every function of
a law is mirrored in the host under FALLBACK-LAW (arrow fns named after the
spec) and drift-checked at boot. Traps we paid for:

- `AIM.ai_thinking_disabled()` in the bench threw TypeError on every probe ->
  all 30 results NET. Use `AIM.THINKING_DISABLED`.
- `AIM.HINT_LIMIT` read from the WRONG module (it lives in
  givel_catalog_media, not givel_ai_models) -> `slice(0, undefined)` returned
  everything. Cap constants come from their own law's artifact.

## Deploy order is a law of its own

Drift-checks refuse to boot on mismatch, so: **artifact BEFORE host, config
(models/key) BEFORE restart**. Both refusals happened for real:

- stale `givel_ai_models.mjs` (no THINKING_DISABLED) -> `ai drift: thinking rule`
- missing `givel_catalog_media.mjs` -> ERR_MODULE_NOT_FOUND crash loop ->
  systemd start-rate-limit -> recover with `systemctl reset-failed`.

After a crash loop the unit stays failed even with the cause fixed:
`reset-failed` first, then `start`.

## Host is the REAL Telegram Bot API

`api.telegram.org` standard methods only: `sendPhoto`/`sendMessage`/`inline_keyboard`.
`sendRichMessage` and `sendMessageDraft` are NOT standard -- anything built on
them silently falls back. The streaming draft mirror must be treated as
best-effort.

## Google Drive photos

Photo URLs are `drive.google.com/thumbnail?id=...`; Telegram cannot follow
them into a sign-in page ("failed to get HTTP URL content"). Public files
redirect to `lh3.googleusercontent.com/d/<id>=w800` and sendPhoto accepts the
URL directly (verified ok:True, Russian caption, booking button). Private
files must be shared by the owner (or the host uploads multipart). 144/158
were public; the public-id list ships as `photo_public.json` sidecar so the
host never pays a Telegram round trip for a known-private photo.

## OpenRouter as the emergency tier

`GET /api/v1/key` returning 200 does NOT mean generation works: the previous
key authenticated fine but the account returned 401 "User not found" on every
model. The replacement key works but has $0 balance -> paid z-ai models give
402. **Free models are the answer**: `cohere/north-mini-code:free` answered
in-role in Russian on a $0 account. The law orders the emergency ladder
working-model-first (`cohere/north-mini-code:free`, then the paid z-ai ones
for a funded future). The tier activates only when `cfg.ai.fallback.key`
exists in config -- a tier without a working key is absent, not failing.

## What stands

- 5-key pool (1 funded + 4 zero-balance), ladder of 6 models, free fallback last
- nightly bench (systemd `givel-bench.timer` 04:10 UTC) probes 5x6 = 30 pairs
  with 1 token and re-seeds `ai.state.json` to a proven-alive pair
- metrics.jsonl (selections, photos_sent) + weekly owner digest in watchdog;
  first viewing booking came through the booking button within an hour of deploy
- boot drift-checks cover: ladder, free tiers, outcome classes, rotation math,
  pacing, config-vs-law equality, media rules, quiz budget, emergency tier
