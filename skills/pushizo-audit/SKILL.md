---
name: pushizo-audit
description: Audit a Pushizo app for delivery blockers and return ranked fixes - channel credentials, subscribers, failed sends, email reputation. Read-only. Use when asked why messages are not delivered, to review an app's setup, or to find what is hurting delivery.
---

# Pushizo audit (read-only)

Find what is blocking delivery and rank the fixes. This skill only reads. Do not change settings, send anything or delete anything.

## Rules

- Use the remote MCP server `https://api.pushizo.com/mcp` or the REST API. The API key comes from the environment; never ask the user to paste it into the chat and never write it to a file.
- Report counts and rates, not raw user lists.
- Separate what you measured from what you infer. If a check could not run, say so instead of assuming it passed.
- Answer in the user's language.

## Checks

1. **Channels:** `GET /v1/apps/{APP_ID}/channels`. Flag missing or `invalid` credentials and certificates close to expiry. `POST …/channels/{channel}/validate` re-checks one, and it changes nothing but the stored status, so ask before running it.
2. **Audience:** `get_subscription_report` for subscribed and unsubscribed counts per channel. Flag a falling trend and channels with no reachable subscribers.
3. **Recent sends:** `list_messages`, then `get_message_report` and `get_provider_report`. Group failures by reason and provider; a single dominant reason is usually the top fix.
4. **Email:** `GET /v1/apps/{APP_ID}/email-deliverability` and `…/email-deliverability/postmaster`. Check domain authentication, spam rate and reputation, and bounce rate. For a draft campaign use `GET …/campaigns/{id}/email-risk`.
5. **Automations:** `list_journeys`, `get_journey_stats`. Flag live automations (`journeys` in the API) with entries but no sends.

## Output

A table ordered by impact, then a one-line next step for the top item:

| # | Finding | Evidence | Fix | Effort |

Say plainly when nothing is wrong. Do not invent problems to fill the table.

Docs: https://docs.pushizo.com/docs/api · MCP: https://docs.pushizo.com/docs/integrations/mcp
