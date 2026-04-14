# Context Log

A [Claude Code skill](https://docs.anthropic.com/en/docs/claude-code/skills) that gives your agent structured, long-term memory over any stream of activity you want to track.

It defines a file-based convention for organizing events from any source — meetings, pull requests, documents, messages — into a three-layer hierarchy that compresses context as it ages:

1. **Daily logs** — every event from a single day, individually listed with metadata and a narrative summary.
2. **Weekly summaries** — synthesize a week's daily logs into themes, progress, and outlook.
3. **Running summary** — a living "state of the world" briefing that compresses older context as history accumulates.

Each tracked stream of activity gets its own isolated **bucket** (a project, a 1:1, a team, an initiative, etc.), with its own config, logs, and summaries that never bleed into each other.

## Why

LLM conversations are ephemeral. Context Log gives your agent a persistent, structured memory it can read at the start of any conversation to pick up where things left off — without replaying raw transcripts or stuffing everything into a single document that grows without bound.

## How it works

### Buckets

A bucket tracks one coherent stream of activity. Examples:

- `project-atlas` — a project and all its ceremonies, PRs, docs, and conversations
- `1on1-jordan-lee` — a recurring 1:1 with a specific person
- `team-platform` — a team's cross-cutting activity
- `initiative-auth-migration` — a topic that spans multiple projects

Each bucket has a `config.md` declaring its event sources (Fathom meetings, GitHub repos, Slack channels, Google Drive folders, etc.) and how to identify relevant events.

### Processing events

When you ask the agent to process events (e.g., "process today's events for project Atlas"), it:

1. Reads the bucket's running summary for historical context
2. Fetches events from each declared source using available MCP connectors
3. Writes or updates the daily log with an event inventory and narrative summary
4. Generates or updates the weekly summary when a week is complete
5. Rewrites the running summary to reflect the latest state

### Reviewing state

Ask things like "what's the latest on project Atlas?" or "catch me up on my 1:1 with Jordan" and the agent reads the running summary to give you a concise briefing. Drill into weekly or daily files for specifics.

## Folder structure

```
<root>/
  project-atlas/
    config.md
    running-summary.md
    weekly/
      2026-06-02.md
      2026-06-09.md
    daily/
      2026-06-02.md
      2026-06-03.md
      2026-06-04.md
```

## Installation

Copy the `skills/context-log` folder into your project's `.claude/skills/` directory, or add it to any location referenced by your Claude Code skills configuration.

## Usage

Once installed, the skill activates when you mention things like:

- "Process today's events for project Atlas"
- "Summarize this week"
- "What happened on project X?"
- "Catch me up on my 1:1 with Jordan"
- "Create a new bucket for the platform migration"

The agent handles bucket creation, event processing, summary generation, and context compression automatically.
