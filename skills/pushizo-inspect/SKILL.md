---
name: pushizo-inspect
description: Inspect Pushizo messages and campaigns for mismatched links, settings and missing languages before or after sending. Read-only. Use when asked to check a campaign before launch, find broken links, or compare messages for inconsistencies.
---

# Pushizo inspect (read-only)

Compare what a message says with how it is configured. Report mismatches; do not fix them unless the user asks.

## Rules

- Read with `get_campaign`, `get_template`, `render_template`, `get_message` and `get_in_app_message`, or the matching REST endpoints.
- Do not open links with a logged-in session and do not call send, schedule or delete endpoints.
- Report what you found with the id and field, so the user can go straight to it.
- Answer in the user's language.

## Checks

1. **Links:** the URL and deep link match what the copy promises (a "see your order" message that opens the home page). Flag `http://`, staging or localhost hosts, and missing or inconsistent UTM values across languages.
2. **Languages:** every language in `headings` and `contents` has both a title and a body, and the app's default language exists.
3. **Personalization:** Liquid tags that use a tag or property with no default. Check with `render_template` using a sample user that lacks the value.
4. **Audience:** the segment exists and is not empty (`estimate_segment`, `get_segment`). Flag a campaign aimed at all users when its copy looks targeted.
5. **Timing:** a send time inside quiet hours, or a stated date in the copy that has already passed.
6. **Channel settings:** a sender or template that is not approved for the channel; an email with no unsubscribe path.

Use `GET …/campaigns/{id}/email-risk` for email campaigns.

## Output

| Severity | Item (id) | Field | Problem | Suggested fix |

Group blockers first. Say so if everything checks out, and list which checks you could not run.

Docs: https://docs.pushizo.com/docs/api
