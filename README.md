# EazeMyAPI Skill for AI

AI skill that gives instant expert knowledge for building with [EazeMyAPI](https://eazemyapi.com) — the AI-powered no-code backend builder.

## What it does

Once installed, AI automatically knows:

- Correct URL structure for CRUD and custom query endpoints
- All 5 auto-generated actions: `list`, `show/:id`, `create`, `update/:id`, `delete/:id`
- Authentication via `X-API-SIGNATURE` header
- Ready-to-use fetch code for React, Next.js, and vanilla JS
- All supported field types and when to use them
- Custom SQL query patterns and parameterized queries
- Vercel serverless proxy pattern for securing API keys in frontend apps
- Schema design best practices for SaaS apps

## Install

Download `eazemyapi.skill` and upload it in AI Platform → Customize → Skills.

## Compatible Platforms

This skill follows the open `SKILL.md` standard — works across 16+ AI tools without any modification:

| Platform | How to install |
|----------|---------------|
| **Claude.ai** | Settings → Skills → Upload ZIP |
| **Claude Code** | Drop folder into `.claude/skills/` |
| **OpenAI Codex CLI** | Drop folder into `~/.codex/skills/` |
| **ChatGPT** | Adopted same format |
| **Cursor** | Reads from `.claude/skills/` (cross-compatible) |
| **Gemini CLI** | `.gemini/skills/` directory |
| **GitHub Copilot** | Supports same spec |
| **Windsurf** | Supports same spec |
| **Aider** | Supports same spec |
| **Augment** | Supports same spec |

## Publish / Discover

Find and share skills across the ecosystem:

- [skills.sh](https://skills.sh) — community directory for Claude skills
- [skillsmp.com](https://skillsmp.com) — aggregates skills for Claude Code, Codex, and more
- [agentskills.io](https://agentskills.io) — Anthropic's official open standard site

## Built by

[Yogi Technolabs](https://yogitechnolabs.com) — the team behind EazeMyAPI.
