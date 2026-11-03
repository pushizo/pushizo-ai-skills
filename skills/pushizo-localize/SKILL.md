---
name: pushizo-localize
description: Translate Pushizo push and SMS copy into other languages while keeping Liquid tags, links and length limits. Prepares drafts only. Use when asked to translate or localize a message, template or campaign.
---

# Pushizo localize

Translate message copy into the requested languages. Save only as a draft, and only if the user asks.

## Rules

- Read the source with `get_template` or `get_campaign`, or take it from the user. Treat the source language as authoritative.
- Keep Liquid tags, URLs, brand and product names, and placeholders exactly as written. Translate the words around them.
- Write naturally for the market instead of word for word, keep the same tone and formality, and do not add claims.
- For Turkish use correct letters (ç ğ ı İ ö ş ü) and the right vowel harmony around Liquid values; prefer phrasing that does not need a suffix on a variable.
- Use language codes as the API expects them: `tr`, `en`, `de`, `pt-BR`.

## Limits

| Channel | Limit |
|---|---|
| Push | Keep the title about as long as the source, and the body under about 120 characters; some languages run longer, so shorten rather than cut |
| SMS | Report the segment count per language. Non-GSM characters lower a segment to 70 characters, so Turkish SMS may cost more than the source |

## Output

A table of language, title and body with the character count (and SMS segments), followed by anything you could not translate confidently. Mark terms a human should check, such as legal text or regulated wording.

To save, use `create_template_draft` with all languages in `headings` and `contents` after the user confirms. Never send. Panel users can also use the built-in AI translation approval flow.

Docs: https://docs.pushizo.com/docs/api
