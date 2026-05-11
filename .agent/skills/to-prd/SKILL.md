---
name: to-prd
description: Turn the current conversation context into a PRD and save it to wiki/llm_conversations/. Use when user wants to create a PRD from the current context.
---

This skill takes the current conversation context and produces a PRD. Do NOT interview the user — just synthesize what you already know.

## Process

1. Sketch out the major modules or schema changes needed. Check with the user that these match expectations.

2. Write the PRD using the template below, then save it to `wiki/llm_conversations/YYYY-MM-DD-<feature-slug>.md`.

<prd-template>

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A numbered list of user stories in the format:
1. As a <actor>, I want <feature>, so that <benefit>

## Implementation Decisions

- Schema changes or new page types
- Operational changes to AGENT.md
- Skill or command changes

## Out of Scope

What is explicitly not covered by this PRD.

## Further Notes

Any additional context.

</prd-template>
