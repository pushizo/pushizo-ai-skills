---
name: pushizo-program
description: Turn a marketing goal into a multi-step Pushizo campaign plan - audience, channels, messages, timing and measurement - and save the parts as drafts on request. Use when asked to plan a campaign, onboarding, win-back, cart recovery or any multi-message program.
---

# Pushizo program

Turn a goal into a plan the user can review, then build it as drafts. Nothing goes live from this skill.

## Rules

- Ask only what changes the plan: the goal metric, the audience, and any channels or dates that are fixed. Use sensible defaults for the rest and state them.
- Base the plan on what the app actually has. Read it with `get_app`, `list_segments`, `list_event_definitions`, `list_templates`, `get_subscription_report` and `get_retention_report`.
- Do not plan on a channel that is not configured or an event that is not tracked. Name it as a prerequisite instead.
- Do not invent benchmarks or expected results. Use the app's own numbers and say when there are none.
- Respect quiet hours, frequency caps and consent. Marketing messages need opt-in.
- Answer in the user's language.

## Plan

1. **Goal and measure:** one metric, the conversion event, and a holdout group if the audience is large enough to read the result.
2. **Audience:** the segment rule and its size (`estimate_segment`), plus who is excluded.
3. **Steps:** a table of step, trigger or delay, channel, message idea, and the exit rule.
4. **Test:** one thing to A/B test first.
5. **Prerequisites and risks:** missing channels, events or consent; what could annoy users.

## Build, after the user approves

In this order, and report the id and name of each:

1. `create_segment_draft`
2. `create_template_draft` for each message, in every language the app uses
3. `create_journey_draft` for an automation (`journeys` in the API), or `create_campaign_draft` for a single send

Drafts stay drafts. The user reviews and publishes them in the dashboard. Do not publish, schedule or send.

Docs: https://docs.pushizo.com/docs/journeys · MCP: https://docs.pushizo.com/docs/integrations/mcp
