---
name: pushizo-server-api
description: Call the Pushizo REST API from backend code with the official server SDKs (Node.js, Python, PHP, Go, Java, .NET, Ruby, Rust) or plain HTTP - send push/email/SMS notifications, upsert users and tags, manage segments, templates and campaigns, handle errors, retries, idempotency and pagination, and generate identity verification hashes or JWTs. Use when the user asks to send notifications or sync users from a server, script or job.
---

# Pushizo server API

Base URL `https://api.pushizo.com` (OpenAPI 3.1: https://github.com/pushizo/pushizo-openapi). All public endpoints
are under `/v1`. Authentication: `Authorization: Bearer <API key>`.

## Rules

- Read the API key from the environment or the project's secret manager (e.g. `PUSHIZO_API_KEY`). Never hard-code
  it, log it, print it in errors, commit it, or send it to the browser or a mobile app. If the project has no secret
  pattern, add the variable to `.env.example` without a value and tell the user to set it.
- App keys are scoped to one app; organization keys can call `/v1/apps` and work across apps. Least privilege: ask
  the user which scopes the key needs when creating one.
- Sending is a side effect visible to real users. Do not run send, schedule or campaign `send` calls (or tests that
  hit the live API) unless the user explicitly asks. Prefer `target.test_devices_only: true` or
  `POST /v1/notifications/estimate` while developing.
- Pass an `Idempotency-Key` (or the body `idempotency_key`) for every send that might be retried, derived from the
  business event (e.g. `order-1001-shipped`), so retries never send twice. The SDKs add a random key automatically,
  but only a deterministic key protects against duplicate jobs.
- Use the official SDK for the project's language when one exists; fall back to plain HTTP only for languages without
  an SDK.

## SDKs

| Language | Install | Client |
|---|---|---|
| Node.js / TypeScript | `npm i @pushizo/node` | `new Pushizo({ apiKey, appId })` |
| Python | `pip install pushizo` | `Pushizo(api_key=..., app_id=...)` |
| PHP | `composer require pushizo/pushizo-php` | `new \Pushizo\Client(['api_key' => ..., 'app_id' => ...])` |
| Go | `go get github.com/pushizo/pushizo-go` | `pushizo.NewClient(key, pushizo.WithAppID(...))` |
| Java | `com.pushizo:pushizo-java` | `Pushizo.builder().apiKey(...).appId(...).build()` |
| .NET | `dotnet add package Pushizo` | `new PushizoClient(new PushizoOptions { ApiKey = ..., AppId = ... })` |
| Ruby | `gem install pushizo` | `Pushizo::Client.new(api_key: ..., app_id: ...)` |
| Rust | `cargo add pushizo` | `pushizo::Client::builder(key).app_id(...).build()?` |

All server SDKs are generated from the same OpenAPI schema and share behavior: default app id, automatic retries
(429 always; 5xx and network errors for idempotent requests), `Retry-After` support, cursor pagination helpers,
typed errors with `request_id`, and webhook signature verification.

## Send a notification

```ts
import { Pushizo } from "@pushizo/node";

const hs = new Pushizo({ apiKey: process.env.PUSHIZO_API_KEY!, appId: process.env.PUSHIZO_APP_ID! });
const result = await hs.notifications.create({
  idempotency_key: "order-1001-shipped",
  target: { include_aliases: { external_id: ["user_42"] } },
  headings: { en: "Your order shipped", tr: "Siparişiniz kargoya verildi" },
  contents: { en: "Track it in the app.", tr: "Uygulamadan takip edin." },
});
console.log(result.id, result.status);
```

Plain HTTP:

```bash
curl -sS https://api.pushizo.com/v1/notifications \
  -H "Authorization: Bearer $PUSHIZO_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: order-1001-shipped" \
  -d '{"app_id":"'"$PUSHIZO_APP_ID"'","target":{"include_aliases":{"external_id":["user_42"]}},"contents":{"en":"Hello"}}'
```

`POST /v1/notifications` returns `202` with `id`, `status` and `recipients_estimate`; delivery is asynchronous.
Track it with `GET /v1/notifications/{id}?app_id=…` and `GET /v1/notifications/{id}/deliveries`.

### Targeting (`target`)

- `include_aliases`: `{ "external_id": ["u1", "u2"] }` — preferred; uses your own user ids.
- `include_external_ids`, `include_subscription_ids`: explicit recipients.
- `included_segments` / `excluded_segments`: segment names (e.g. `"Subscribed Users"`).
- `filters`: ad-hoc segment definition. `channels`: limit to `ios_push`, `android_push`, `web_push`, `email`, `sms`.
- `email_to`, `include_phone_numbers`: direct email/SMS recipients.
- `test_devices_only: true`: only devices marked as test devices.

Content is localized per language code (`headings`, `contents`, `subtitle`); include at least `en`. Use
`template_id` to send a saved template, `send_after` (ISO 8601) to schedule, and `email_subject` / `email_body`
for email.

## Users, tags and events

- Upsert: `POST /v1/apps/{appId}/users` with `external_id`, `tags`, `properties`, optional `subscriptions`.
- Read / update / delete: `/v1/apps/{appId}/users/by/{label}/{value}` (`label` = `external_id`, `hs_id` or a custom
  alias). `PATCH` merges tags and properties.
- Custom events: `POST /v1/apps/{appId}/events` (up to 500 per call).

## Segments, templates and campaigns

- Segments: `GET|POST /v1/apps/{appId}/segments`, `POST …/segments/preview` (count and sample, no side effects).
- Templates: `GET|POST /v1/apps/{appId}/templates`, `POST …/templates/{id}/render` to preview with sample data.
- Campaigns: `POST /v1/apps/{appId}/campaigns` creates a draft. `…/schedule` and `…/send` are real sends and may
  need approval; only call them when the user explicitly asks. Email campaigns are blocked with `409` when the
  list risk is too high; report the `email-risk` result instead of forcing.

## Errors, retries and pagination

Error body: `{"error": {"code": "HS_…", "message": "…", "request_id": "…", "details": {…}}}`.
Common codes: `HS_VALIDATION_ERROR` (400/422), `HS_UNAUTHENTICATED` (401), `HS_FORBIDDEN` (403),
`HS_NOT_FOUND` (404), `HS_IDEMPOTENCY_CONFLICT` (409), `HS_RATE_LIMITED` (429, honor `Retry-After`),
`HS_QUOTA_EXCEEDED`, `HS_CHANNEL_NOT_CONFIGURED`. Log `request_id`, never the API key.

List endpoints use cursor pagination: `?limit=…&cursor=…` → `{ "data": [], "next_cursor": "…", "has_more": true }`.
Use the SDK pagination helpers (for example `paginate` in Node, `ListAll` in Go, `list_all` in Rust) instead of
hand-written loops.

## Identity verification

When the app requires identity verification, the client SDK's `login` needs proof generated on your server:

- HMAC: `identity_hash = HMAC-SHA256(sdk_secret, external_id)` as hex (SDK helper `Pushizo.identityHash` in Node,
  `identity_hash` in Python and Rust, `IdentityHash` in Go).
- JWT: ES256 token with `sub = external_id`, `aud = app_id`, `kid` header, max 24 h (SDK helper `identityToken` /
  `identity_token` / `IdentityToken`).

Keep the SDK secret and private key server-side; expose an authenticated endpoint that returns the hash or token for
the signed-in user only.

Docs: https://docs.pushizo.com/docs/api · SDK guides: https://docs.pushizo.com/docs/sdk
