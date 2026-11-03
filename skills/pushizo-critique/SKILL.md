---
name: pushizo-critique
description: Review push, email, SMS or in-app copy for a Pushizo app and suggest rewrites - clarity, truncation, call to action, personalization safety. Read-only unless the user asks to save a draft. Use when asked to review, improve or rewrite message copy.
---

# Pushizo copy critique

Review message copy and propose concrete rewrites. Nothing is saved or sent unless the user asks.

## Rules

- Get the copy from the user, or read it with `get_template`, `get_campaign` or `get_message`.
- Keep the user's meaning, offer and facts. Never add a discount, deadline or claim that is not in the source.
- Keep Liquid tags (`{{ user.tags.name }}`) exactly as written.
- Answer in the user's language; rewrite copy in the language it was written in.

## What to check

| Channel | Check |
|---|---|
| Push | The title and first 40 characters carry the message on their own; body under about 120 characters; one action |
| Email | Subject under about 50 characters; preheader adds something new; one primary link |
| SMS | Segment count (non-GSM characters such as Turkish letters lower the limit to 70 per segment); opt-out where required |
| In-app | Button says what happens next; can be dismissed |

For every channel: is it clear who it is from and why now; does it depend on a Liquid value with no fallback (`{{ user.tags.name | default: "" }}`); does the deep link match the message.

Use `get_template_stats` or `get_message_report` when past results exist, and say when the sample is too small to mean anything.

## Output

For each message: the two or three most important problems, then two rewrites with the trade-off in one line. If the copy is already good, say so.

To save a rewrite, use `create_template_draft` after the user confirms. Do not send it; `send_test_notification` reaches test devices only and still needs the user's request.

Docs: https://docs.pushizo.com/docs/api
