---
name: pushizo-drafts
description: Prepare Pushizo content and configuration as drafts for human review - iOS/Android push channel setup, email templates and draft campaigns, segment drafts - without sending anything. Use when the user asks an agent to set up a Pushizo channel, write a campaign or define an audience.
---

# Pushizo drafts (review before sending)

These tasks change a live Pushizo app. The agent prepares and saves **drafts**; a human reviews and sends them from
the dashboard or by explicitly asking.

## Rules

- Use the Pushizo REST API (or the remote MCP server `https://api.pushizo.com/mcp`, which exposes read and draft tools
  and no send tools). The API key comes from the environment; never ask the user to paste it into the chat and never
  write it to a file.
- Never call send, schedule, campaign `send`/`schedule`/`resend-failed`, or delete endpoints unless the user says so
  explicitly in this conversation.
- Do not export or print raw user lists; summarize counts.
- Write user-facing copy in every language the app uses (at least English, plus the user's language).
- At the end, list what was created (ids and names) and what the human still has to do.

## iOS / Android push channel

1. `GET /v1/apps/{APP_ID}/channels` and look for `ios_push` / `android_push`.
2. If missing, tell the user which credential is needed and where to get it:
   - iOS: APNs auth key `.p8`, Key ID, Team ID, bundle id (Apple Developer → Keys).
   - Android: Firebase service account JSON (Firebase console → Project settings → Service accounts).
   The user uploads it in the dashboard, or the agent sends it with `PUT /v1/apps/{APP_ID}/channels/{channel}` reading
   the file from a path the user gives. Never echo the credential back into the chat.
3. `POST /v1/apps/{APP_ID}/channels/{channel}/validate` and report the result.
4. Suggest a test only to test devices (`target.test_devices_only: true`) and only send it after the user confirms.
   Example copy: title "Your order is on its way" / "Siparişin yola çıktı"; body under 120 characters.

## Email template and draft campaign

1. Write subject, preheader and body (HTML or plain text) for each language.
2. Save the template: `POST /v1/apps/{APP_ID}/templates`. Preview with `POST …/templates/{id}/render`.
3. Save the campaign as a draft: `POST /v1/apps/{APP_ID}/campaigns` with the template and audience. Do not set a send
   time and do not call `…/schedule` or `…/send`.
4. Check `GET …/campaigns/{id}/email-risk` and report the estimated invalid and bounce rates.

## Segment draft

1. Express the rule with one of: tag, last activity, subscription channel, external id list.
2. Validate with `POST /v1/apps/{APP_ID}/segments/preview` (count and sample, no side effects); summarize the count.
3. Save with `POST /v1/apps/{APP_ID}/segments`, giving the name in English and in the user's language.

Docs: https://docs.pushizo.com/docs/api · MCP: https://docs.pushizo.com/docs/integrations/mcp
