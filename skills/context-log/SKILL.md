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
for any stream of activity the user wants to track.

## Core Concepts

### Buckets

A bucket is an isolated context log environment that tracks one stream of activity over time.
Buckets never share context with each other. What constitutes a bucket is up to the user --
it's any scope where they want a unified, evolving understanding of what's happening. Common
examples:

- A **project** (with its ceremonies, code activity, docs, and conversations)
- A **1:1 relationship** with a specific person
- A **client relationship** spanning multiple projects
- A **team or workgroup** the user wants to track across its activities
- A **topic or initiative** that cuts across other boundaries (e.g., "platform migration")

The only rule is that a bucket should represent a single coherent context. If the user wants
to filter events by two unrelated concerns within one bucket, that's a sign it should be two
buckets.

Each bucket contains a `config.md` that declares which event sources to pull from, how to
identify relevant events, and any special instructions from the user that should be followed
when pulling events or writing logs and summaries for that bucket.

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
   files via relative links. Two kinds of content live here, and only these two: **fresh
   knowledge** (what is happening right now and what is actively in flight) and **foundational
   knowledge** (project goals, scope, team, durable decisions that still shape current work).
   Stale tactical detail does NOT live here -- it is reachable via weekly and daily files.

Each layer is more condensed than the one below it. Daily logs are detailed. Weekly summaries
distill patterns. The running summary is the executive briefing -- short enough to fit easily
in an agent's context and to be read end-to-end by a human in a few minutes (target ~200-300
lines, ~3500-5000 tokens). If it grows past that, the next update MUST compress before it
adds. The size constraint is not aesthetic; it is the point of this layer.

### Reading Context: Top-Down Drill-Down

Whenever a task involves a bucket -- reviewing state, answering questions, preparing for a
meeting, writing a document, processing new events -- always load context top-down:

1. **Always start with `running-summary.md`.** This is mandatory for every interaction with a
   bucket, no exceptions. It gives you the current state, recent trajectory, and key decisions.

2. **Pull weekly summaries when the task needs more depth.** If the user asks about a specific
   time period, wants to understand how something evolved, or the running summary alone doesn't
   give you enough to act on, read the relevant weekly files. Follow the relative links in the
   running summary.

3. **Pull daily logs when you need event-level detail.** If the user asks about a specific
   event, meeting, PR, or conversation -- or the weekly summary references something you need
   to understand fully -- drill into the daily log. Follow the relative links in the weekly
   summary.

4. **Re-fetch from the original source when full fidelity is needed.** Daily logs store
   re-fetch metadata (recording IDs, PR URLs, doc IDs, thread permalinks). When the user needs
   exact quotes, full transcripts, complete diffs, or raw data that the summary doesn't
   capture, use the appropriate connector to fetch the original event.

How far down you drill depends on what the user is asking. A quick status check may only need
the running summary. Preparing a detailed report may require reading through weekly and daily
files. Answering "what exactly did Jordan say about the timeline?" may require re-fetching the
meeting transcript. Match the depth of your reading to the depth of the task.

## Folder Structure

All context logs live under a root folder the user selects. Each bucket is a direct
subdirectory of that root — do not add intermediate folders like `context-log/` or `buckets/`.

```
<root>/
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

**Event Inventory** -- Every event listed individually with a heading and a summary of what
happened, plus the metadata the connector needs to find this specific event again (e.g.,
recording ID, PR URL, doc ID, thread permalink).

**Daily Summary** -- A narrative of the day's happenings in the context of this bucket. This
is where you synthesize across events: "The DSW focused on auth migration planning. Jordan
opened PR #142 with the initial schema changes, which Sarah started reviewing. The client
asked about timeline in Slack and was told Sprint 14." Be as long or short as needed to
consolidate all important events from the day. The summary should reference the running
summary context to highlight what moved forward, what's new, and what changed. Use markdown
formatting (headings, bold, bullet points, etc.) to keep the summary easily scannable.

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
  -- formatted for easy scanning using markdown (headings, bold, bullet points, etc.)
- Highlights the most significant events/decisions of the week
- Is more condensed than the sum of daily logs -- it should focus on what matters at the
  week-level rather than repeating every event

If the week is still in progress, the weekly file should note which days have been processed
and which haven't. It can be regenerated later when more days are available.

When generating a weekly summary, read all daily logs for that week. Don't rely on memory
of what you processed earlier in the conversation -- always read the files.

### Step 6: Update the Running Summary

Read `references/running-summary-template.md` for the format. The running summary has five
sections, each with a specific purpose. Treat the size budget (~200-300 lines, ~3500-5000
tokens) as a hard constraint: if a section grows past its share, it must be compressed
before anything new is added.

Reframing the goal: the running summary answers two questions only -- "what is this project
for" (foundational) and "what is happening right now" (fresh). Anything that doesn't answer
one of those does not belong here. Detailed history lives in weekly and daily files, which
are one click away.

1. **Project Foundation** -- Edit only when foundations actually shift. The durable facts
   about this bucket: mission/goal, scope, team and ownership, contract or timeline anchors,
   primary success criteria. Should rarely change update-to-update. Aim for ~10-20 lines.

2. **Now** -- Rewrite entirely each update. What is actively in flight this week and the
   next one or two; the immediate blockers; the focus for the current cycle; the next 1-2
   milestones. This section is _only_ about the present and the near future. Aim for ~25-50
   lines.

3. **Recent Weeks** (last 3 weeks max) -- One entry per week, linking to the weekly file.
   **Hard cap: 3-5 single-line bullets per week.** Each bullet is one line; no compound
   bullets stuffed with parentheticals or PR/ticket numbers (those live in the weekly file).
   When you add a new week, re-trim the older weeks; older entries should get shorter as
   they age, not stay at original length. When this section exceeds 3 weeks, move the
   oldest entry to Archived Context.

4. **Key Decisions Log** -- Curated, not append-only. Captures decisions that still shape
   current work. Each entry is 1-2 lines, dated. Rules:

   - When a decision is superseded, mark it superseded inline and link to the replacement
     decision; do not silently delete it.
   - When a decision is older than ~8 weeks AND is no longer being cited in current work,
     move it to Archived Context (or, if the bucket is large, to a `decisions-archive.md`
     file in the bucket folder). The decision remains discoverable but does not bloat the
     active log.
   - Tactical implementation notes (which PR shipped what, which version was bumped) are
     NOT decisions; do not log them here. A decision changes direction, scope, ownership,
     timeline, or methodology.

5. **Archived Context** -- Condensed older material. When weeks roll out of Recent Weeks,
   collapse them into a single line or short paragraph and merge here. Same for retired
   decisions. Keep only what's still relevant to understanding the current state. If this
   section exceeds ~40 lines, condense further.

### Step 7: Pruning Pass (mandatory)

After writing the new content in Step 6, re-read the entire running summary end-to-end and
apply the following checks. **This step is not optional.** It is what prevents drift from
"executive briefing" into "project encyclopedia."

For each item currently in the file, ask:

1. **Is it fresh or foundational?** If it is neither (e.g., a workstream that has shipped, a
   bug that has been resolved, a milestone that has passed, a decision no longer cited),
   remove it. The detail still exists in the weekly and daily logs, which are linked.
2. **Has it moved in the last 2 weeks?** If a Now-section item hasn't moved in 2+ weeks and
   is not foundational, drop it or fold it into a short "still pending" line.
3. **Are resolved items being kept with strikethrough?** If yes, delete them. The resolution
   belongs in the Decisions Log if it was a decision, otherwise in the weekly summary that
   covers the week of resolution.
4. **Is "Upcoming Milestones" actually upcoming?** If the section contains "Done [date]"
   entries, remove them. Only future-dated items belong there.
5. **Are Recent Weeks bullets one line each, capped at 5 per week?** If not, trim.
6. **Total size check.** Roughly count lines. If the file is over budget (~300 lines), more
   aggressive pruning is required before this update is considered complete. Compress
   first, then add.

If pruning produces a meaningfully shorter file, that is a successful update. The right
mental model is gardening, not archiving: the file should feel lighter after each update,
not heavier.

### Processing Multiple Days

When processing a range (e.g., "process this week"), work through each day chronologically.
Write each daily log, then generate the weekly summary once all days are done, then update
the running summary once at the end. Don't update the running summary after each day --
just once when everything is processed. Run the Step 7 pruning pass exactly once, at the
very end.

## Workflow: Reviewing Current State

When the user asks things like "what's the latest on project Atlas?" or "catch me up on
my 1:1 with Jordan", follow the top-down drill-down pattern:

1. Read the bucket's `running-summary.md` (always -- this is the entry point)
2. Present a concise verbal summary of the most important current items
3. Proactively drill deeper when the running summary isn't enough to fully address
   what the user is asking -- don't wait for them to ask for more detail:
   - Read relevant weekly files to cover a time period or understand how something evolved
   - Read daily logs to get event-level specifics
   - Re-fetch from the original source via connectors when exact details are needed

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

## Workflow: Bucket Maintenance / Aggressive Re-prune

If the running summary has clearly drifted past budget (e.g., it's 5x the target size, or
the user explicitly asks to "clean up" / "prune" / "compress" the bucket's running summary),
do a maintenance pass independent of any event processing:

1. Read the running summary end-to-end.
2. Identify content that should be removed entirely vs. moved to Archived Context vs. kept.
3. Apply Step 7's checklist aggressively.
4. Show the user the proposed shape (sections + approximate sizes) before rewriting; this
   is a destructive operation in spirit even though older content is still recoverable
   from weekly/daily files.
5. Rewrite the file under budget.

## Important Notes

- **No raw data storage.** Daily logs contain summaries and re-fetch metadata, not full
  transcripts, diffs, or message bodies. The connector is always the source of truth.
- **Always process chronologically.** Context builds forward in time.
- **One bucket per context stream.** Don't mix unrelated activity streams in one bucket.
- **Always read top-down.** The running summary is the mandatory starting point for every
  bucket interaction. Drill into weekly, daily, and original sources as the task demands
  (see "Reading Context: Top-Down Drill-Down" above).
- **Ask before creating buckets.** Confirm the name, type, and sources with the user.
- **Graceful degradation.** If a connector isn't available or a source returns no results,
  continue with what you have. Note the gap but don't block the whole process.
- **Relative links.** Weekly files link to daily files. The running summary links to weekly
  files. Always use relative paths so the folder can be moved without breaking links.
- **Daily log updates are additive.** If a daily log already exists and you're processing
  more events for the same day, append events and rewrite the summary -- don't discard
  what was already there.
- **Running summary updates are subtractive first, then additive.** When updating, prune
  before you write. The file should feel lighter after each update, not heavier.
