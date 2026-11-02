---
name: pushizo-webhooks
description: Receive Pushizo outgoing webhooks (event streams) safely - verify the Pushizo-Signature HMAC header on the raw body, handle single and batch payloads, deduplicate by event id and rotate secrets. Use when the user builds an endpoint that consumes Pushizo events such as message delivery, clicks or user changes.
---

# Pushizo webhooks

Pushizo signs every webhook request:

```
Pushizo-Signature: t=<unix seconds>,v1=<hex HMAC-SHA256(secret, t + "." + raw body)>
Pushizo-Event-Id: <event id>
```

During secret rotation the header carries two `v1` values for 24 hours (old and new secret).

## Rules

- Verify against the **raw request body bytes**. Configure the framework to give you the raw body for this route
  (Express `express.raw({ type: "application/json" })`, Next.js route handler `await req.text()`, Flask
  `request.get_data()`, Laravel `$request->getContent()`, Go `io.ReadAll(r.Body)`). Never re-serialize parsed JSON
  before verifying.
- Read the secret (`whsec_…`) from the environment; never log it. The secret is shown only once, when the endpoint is
  created (`POST /v1/apps/{appId}/webhooks`) or rotated (`…/rotate-secret`).
- Reject with `400` when verification fails; do not leak which check failed in the response body.
- Default timestamp tolerance is 300 seconds (replay protection); keep it.
- Respond `2xx` quickly and process asynchronously (queue). Deliveries are retried on failure and can be replayed,
  so deduplicate by `Pushizo-Event-Id` / `event.id`.
- Batch endpoints receive `{"events": [ … ]}`; single deliveries receive one event object. The SDK helpers return a
  list in both cases.

## SDK helpers

| Language | Verify and parse |
|---|---|
| Node.js | `constructWebhookEvents({ payload, header, secret })` from `@pushizo/node` (throws `WebhookSignatureError`) |
| Python | `construct_webhook_events(payload, header, secret)` from `pushizo` |
| PHP | `\Pushizo\Webhook::constructEvents($payload, $header, $secret)` |
| Go | `pushizo.ConstructWebhookEvents(payload, header, []string{secret}, nil)` |
| Rust | `pushizo::construct_webhook_events(raw, header, &[secret], &Default::default())` |
| Java, .NET, Ruby | `Webhooks` helpers in the respective SDK (see the SDK README) |

Node.js (Express):

```ts
import express from "express";
import { constructWebhookEvents, WebhookSignatureError } from "@pushizo/node";

const app = express();
app.post("/hooks/pushizo", express.raw({ type: "application/json" }), (req, res) => {
  try {
    const events = constructWebhookEvents({
      payload: req.body,
      header: req.headers["pushizo-signature"],
      secret: process.env.PUSHIZO_WEBHOOK_SECRET!,
    });
    for (const event of events) enqueue(event); // deduplicate by event.id
    res.sendStatus(204);
  } catch (err) {
    if (err instanceof WebhookSignatureError) return res.sendStatus(400);
    throw err;
  }
});
```

## Without an SDK

1. Split the header on `,`, read `t` (digits) and every `v1` (64 lowercase hex chars).
2. Reject if `|now - t| > 300`.
3. Compute `HMAC-SHA256(secret, t + "." + rawBody)` and compare with each `v1` using a constant-time comparison.
4. Parse the JSON only after the signature matches.

## Testing

- `POST /v1/apps/{appId}/webhooks/{webhookId}/test` sends a signed test event and returns the result.
- `GET …/webhooks/{webhookId}/deliveries` lists attempts; `POST …/webhooks/deliveries/{deliveryId}/replay` resends
  with the same event id.

Docs: https://docs.pushizo.com/docs/webhooks
