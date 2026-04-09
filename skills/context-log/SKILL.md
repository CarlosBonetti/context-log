---
name: context-log
description: >
  A file-based convention for building context-aware memory over time. Teaches the agent how to
  organize events from any source (meetings, PRs, docs, messages) into a structured folder
  hierarchy: daily logs, weekly summaries, and a rolling "state of the world" running summary.
  Each tracked stream of activity (a project, a 1:1, a team, an initiative) gets its own
  isolated "bucket" with its own memory. The skill defines the folder structure, file formats,
  and the workflow for processing new events, updating summaries, and compressing older context
  so the running summary stays useful over weeks and months. Use this skill whenever the user
  mentions: processing events, updating a project log, reviewing what happened recently,
  running summaries, "catch me up", "what's the latest on", daily or weekly logs, context
  logs, or bucket management. Also trigger for: "process today's events for X", "summarize
  this week", "what happened on project X", "update my 1:1 with [person]".
---

# Context Log

A convention-based skill that defines how to organize events from any source into a
structured folder hierarchy -- daily logs, weekly summaries, and a rolling "state of the
world" document -- so that the agent can build and maintain context-aware memory over time
for any stream of activity you want to track.

## Core Concepts

### Buckets

A bucket is an isolated context log environment that tracks one stream of activity over time.
Buckets never share context with each other. What constitutes a bucket is up to you -- it's
any scope where you want a unified, evolving understanding of what's happening. Common examples:

- A **project** (with its ceremonies, code activity, docs, and conversations)
- A **1:1 relationship** with a specific person
- A **client relationship** spanning multiple projects
- A **team or workgroup** you want to track across its activities
- A **topic or initiative** that cuts across other boundaries (e.g., "platform migration")

The only rule is that a bucket should represent a single coherent context. If you find yourself
wanting to filter events by two unrelated concerns within one bucket, that's a sign it should
be two buckets.

Each bucket contains a `config.md` that declares which event sources to pull from and how to
identify relevant events.

### Events

An event is any discrete unit of activity relevant to a bucket. Examples:
- A Fathom meeting transcript (DSW, demo, daily, 1:1)
- A GitHub pull request (opened, reviewed, merged)
- A Google Drive document (created or significantly updated)
- A Slack thread or message in a project channel
- Any other activity surfaced by a connected MCP tool

Events are not stored as raw data. Instead, the daily log captures a summary of each event
plus enough metadata (IDs, URLs, links) to re-fetch the full details from the original source
via its connector.

### The Three-Layer Hierarchy

Context compresses as it ages, flowing through three layers:

1. **Daily Logs** (`daily/YYYY-MM-DD.md`) -- Every event from a single day, individually
   listed with metadata, plus a narrative summary of the day's happenings.

2. **Weekly Summaries** (`weekly/YYYY-MM-DD.md`, named by Monday's date) -- References daily
   logs via relative links. Synthesizes the week's events into themes, progress, and outlook.

3. **Running Summary** (`running-summary.md`) -- The living memory. References recent weekly
   files via relative links. Provides a standalone briefing of current state, plus a decisions
   log that persists across time.

Each layer is more condensed than the one below it. Daily logs are detailed. Weekly summaries
distill patterns. The running summary is the executive briefing.

## Folder Structure

All context logs live under a root folder the user selects. The structure:

```
context-log/
  buckets/
    <slug>/
      config.md
      running-summary.md
      weekly/
        2026-06-02.md          # Monday of that week
        2026-06-09.md
      daily/
        2026-06-02.md
        2026-06-03.md
        2026-06-04.md
```

Slugs are lowercase, hyphen-separated. Use a short prefix that hints at the bucket type
when it helps clarity (e.g., `project-atlas`, `1on1-jordan-lee`, `team-platform`,
`initiative-auth-migration`), but this is a convention, not a requirement.

Weekly files are always named after the Monday that starts that week, regardless of which
day you generate them on.

## Workflow: Processing Events

When the user asks to process events (e.g., "process today's events for project Atlas",
"update my 1:1 with Jordan", "process this week"), follow these steps.

### Step 1: Identify the Bucket

Determine which bucket the user is referring to. Look at existing bucket folders. If the
bucket doesn't exist yet, follow the "Creating a New Bucket" workflow below before continuing.

Read the bucket's `config.md` to understand its event sources and identification patterns.

### Step 2: Load Context

Read the bucket's `running-summary.md`. This is the historical context that makes everything
"context-aware." If the bucket is brand new, skip this step -- the first processing run
establishes the baseline.

### Step 3: Fetch Events from Each Source

For each event source declared in the bucket's `config.md`, use the appropriate connector to
find events in the requested time range. The config file specifies which tools to use and how
to identify relevant events (search queries, channel names, repo filters, etc.).

General approach per source type:

- **Fathom meetings**: Use `search_meetings` or `list_meetings` with date filters. For each
  matching meeting, use `get_summary` for the overview and `get_transcript` if more detail
  is needed.
- **GitHub**: Look for PRs opened, reviewed, merged, or commented on. Issues created or
  closed. Use the relevant GitHub connector tools.
- **Google Drive**: Search for documents created or modified in the time range that match
  the bucket's patterns. Use the Drive connector to fetch content.
- **Slack**: Search for messages in the declared channels or threads. Use the Slack connector
  with relevant queries.
- **Other MCP tools**: Follow the same pattern -- use whatever search/list tools the connector
  provides, filtered by date and relevance.

If a source connector is not available (the user hasn't connected it), skip that source
and note it in the daily log. Don't error out -- partial data is still useful.

Present the list of events found and confirm with the user before writing anything.

### Step 4: Write or Update the Daily Log

For each day in the requested range, create or update a daily log file.
Read `references/daily-log-template.md` for the format.

The daily log has two parts:

**Event Inventory** -- Every event listed individually with:
- Event type and source (e.g., "Fathom: DSW Meeting", "GitHub: PR #142")
- Timestamp
- Participants (where applicable)
- A 2-4 line summary of what happened
- A metadata block with IDs/URLs sufficient to re-fetch the full event via its connector
  (e.g., Fathom recording_id, PR URL, Drive doc ID, Slack thread permalink)

**Daily Summary** -- A short narrative (aim for 5-15 lines) of the day's happenings in the
context of this bucket. This is where you synthesize across events: "The DSW focused on auth
migration planning. Jordan opened PR #142 with the initial schema changes, which Sarah started
reviewing. The client asked about timeline in Slack and was told Sprint 14." The summary
should reference the running summary context to highlight what moved forward, what's new,
and what changed.

If the daily log already exists (e.g., the user processed morning events and is now adding
afternoon ones), append the new events to the inventory and rewrite the daily summary to
cover the full day.

### Step 5: Write or Update the Weekly Summary

If the user processed events that complete (or nearly complete) a week, or if they explicitly
ask for a weekly rollup, generate or update the weekly summary file.

Read `references/weekly-summary-template.md` for the format.

The weekly file:
- Is named after the Monday of that week (e.g., `weekly/2026-06-02.md`)
- Links to each daily log in that week using relative paths (e.g., `[Monday](../daily/2026-06-02.md)`)
- Contains a weekly narrative summary that synthesizes themes, progress, blockers, and outlook
- Highlights the most significant events/decisions of the week
- Is more condensed than the sum of daily logs -- it should focus on what matters at the
  week-level rather than repeating every event

If the week is still in progress, the weekly file should note which days have been processed
and which haven't. It can be regenerated later when more days are available.

When generating a weekly summary, read all daily logs for that week. Don't rely on memory
of what you processed earlier in the conversation -- always read the files.

### Step 6: Update the Running Summary

Read `references/running-summary-template.md` for the format. The running summary has four
sections:

1. **Current State** -- Rewrite entirely. A standalone briefing someone could read with zero
   prior context: active workstreams, ownership, blockers, upcoming milestones, open questions.
   Should make sense on its own without reading any other file.

2. **Recent Weeks** (last ~3-4 weeks) -- One entry per week, linking to the weekly file
   (e.g., `[Week of Jun 2](weekly/2026-06-02.md)`). Each entry is 3-5 bullets summarizing
   that week. When this section exceeds ~4 weeks, move the oldest entries to Archived Context.

3. **Key Decisions Log** -- Append-only, never deleted. When a meaningful decision is made
   (technical direction, scope change, timeline shift, personnel change), add a dated entry.
   Keep each to 1-2 lines.

4. **Archived Context** -- Condensed older material. When weeks roll out of Recent Weeks,
   summarize and merge them here. Keep only what's still relevant to current work. If this
   section exceeds ~40 lines, condense further.

The running summary should stay under ~200 lines total.

### Processing Multiple Days

When processing a range (e.g., "process this week"), work through each day chronologically.
Write each daily log, then generate the weekly summary once all days are done, then update
the running summary once at the end. Don't update the running summary after each day --
just once when everything is processed.

## Workflow: Reviewing Current State

When the user asks things like "what's the latest on project Atlas?" or "catch me up on
my 1:1 with Jordan":

1. Read the bucket's `running-summary.md`
2. Present a concise verbal summary of the most important current items
3. If the user asks for more detail on a specific topic, time period, or event:
   - Read the relevant weekly or daily file (follow the relative links)
   - For a specific event, use the metadata in the daily log to re-fetch details from
     the original connector if needed

## Workflow: Creating a New Bucket

When the user wants a new bucket, or when you can't match events to an existing one:

1. Ask for: bucket name, a short description of what it tracks
2. Ask which event sources to track. Walk through what connectors are available and let the
   user pick which ones are relevant. For each source, gather the identification patterns
   (meeting title keywords, Slack channels, GitHub repos, Drive folder names, etc.)
3. Create the folder structure: `config.md`, empty `running-summary.md`, `weekly/`, `daily/`
4. Read `references/bucket-config-template.md` and fill it in based on answers from step 2
5. Initialize `running-summary.md` from `references/running-summary-template.md`
6. Ask if the user wants to backfill historical events. If yes, determine the date range
   and process chronologically.

## Workflow: Stale Weekly Summary Handling

When reading a bucket's running summary and you notice the most recent weekly file is older
than 1-2 weeks, the weekly layer may be stale. In this case:

1. Check if there are daily logs that aren't covered by a weekly summary yet
2. If yes, generate the missing weekly summaries from those daily logs
3. Update the running summary with the new weekly entries

This can happen if the user processes events daily but doesn't always ask for a weekly rollup.
Handle it gracefully -- don't scold the user, just fill in the gaps.

## Important Notes

- **No raw data storage.** Daily logs contain summaries and re-fetch metadata, not full
  transcripts, diffs, or message bodies. The connector is always the source of truth.
- **Always process chronologically.** Context builds forward in time.
- **One bucket per context stream.** Don't mix unrelated activity streams in one bucket.
- **The running summary is the entry point.** When you need to understand a bucket, start
  there. Drill into weekly and daily files only when you need specifics.
- **Ask before creating buckets.** Confirm the name, type, and sources with the user.
- **Graceful degradation.** If a connector isn't available or a source returns no results,
  continue with what you have. Note the gap but don't block the whole process.
- **Relative links.** Weekly files link to daily files. The running summary links to weekly
  files. Always use relative paths so the folder can be moved without breaking links.
- **Daily log updates are additive.** If a daily log already exists and you're processing
  more events for the same day, append events and rewrite the summary -- don't discard
  what was already there.
