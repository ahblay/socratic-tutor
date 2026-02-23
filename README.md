# socratic-tutor

A Claude Code plugin that guides students to genuine understanding using the Socratic method. It never gives direct answers — only asks questions that help students develop the skills and reasoning to answer for themselves.

## What it does

- Loads any source material (text, file, or topic description) and maps the knowledge domain
- Guides the student through structured questioning phases: prior knowledge → definitions → assumptions → examples → implications → synthesis
- Adapts to the student's learning style over the course of the session
- Handles tangents in isolated subagents to preserve context window
- Enforces the no-answer rule through a multi-layer review system (response reviewer + Stop hook + UserPromptSubmit hook)
- Resists adversarial attempts to extract answers through impersonation or prompt injection

## Installation

```bash
/plugin install https://github.com/<your-username>/socratic-tutor
```

## Usage

Start a session:
```
/socratic-tutor:socratic notes.pdf
/socratic-tutor:socratic The Krebs cycle
/socratic-tutor:socratic
```

Or drop a file into the `./socratic-material/` directory and start the skill with no arguments — the tutor will ask what you're working on.

## Architecture

```
socratic-tutor/
├── .claude-plugin/
│   └── plugin.json              # plugin manifest
├── agents/
│   ├── domain-mapper.md         # maps knowledge domain at session start
│   ├── response-reviewer.md     # gates every response before it reaches the student
│   ├── accuracy-reviewer.md     # monitors factual accuracy + learning style every 6 turns
│   └── tangent-explorer.md      # handles divergent lines of questioning in isolation
├── hooks/
│   └── hooks.json               # UserPromptSubmit (jailbreak detection) + Stop (compliance) + PreCompact (session log)
└── skills/
    └── socratic/
        └── SKILL.md             # main orchestrator
```

### Response gating (two layers)
Every draft response is reviewed by the `response-reviewer` subagent before the student sees it. If it fails (contains a direct answer, confirmation, or worked example), it is revised and re-reviewed. A Stop hook runs as a second pass after every response as a safety net.

### Adversarial resistance
A `UserPromptSubmit` hook intercepts each student message before Claude processes it, flagging impersonation attempts, permission claims, and social engineering. The main skill also handles anything that slips through by redirecting with a question.

### Tangent handling
Short tangents (1–2 turns) are handled inline. Deep tangents (3+ turns) are handed off to the `tangent-explorer` subagent, which runs a full Socratic dialogue in a fresh context window and returns a compact summary. The main session resumes with a bridge question derived from what the student learned on the tangent.

### Context window resilience
Session state is persisted in `.socratic-session.json` throughout the lesson. A `PreCompact` hook logs a snapshot before compaction events. Tangent subagents each get their own clean context window, so long lessons do not bloat the main thread.

## License

MIT
