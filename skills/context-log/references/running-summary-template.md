# Running Summary: [Bucket Name]

> Description: [what this bucket tracks]
> Last updated: [YYYY-MM-DD]
> Event sources: [list from config, e.g., Fathom, GitHub, Slack]

---

## Project Foundation

_Durable facts about this bucket. Edit only when foundations actually shift -- mission/goal,
scope, team and ownership, contract or timeline anchors, primary success criteria. Aim for
~10-20 lines. If something here changes weekly, it probably belongs in Now instead._

[Foundation goes here]

---

## Now

\_Rewrite entirely each update. What is actively in flight this week and the next one or two,
the immediate blockers, the focus for the current cycle, the next 1-2 milestones. This
section is only about the present and the near future. Aim for ~25-50 lines.

Use whatever structure fits the bucket -- short subsections (e.g., "This week", "In-flight",
"Blockers", "Next milestones") usually work better than one long prose blob. Use markdown
formatting for fast scanning. Be ruthless about what is "now" vs. what already happened.\_

[Now goes here]

---

## Recent Weeks

_One entry per week, newest first. Link to the weekly summary file. Hard cap: 3-5 single-line
bullets per week, max 3 weeks. Each bullet is one line; no compound bullets stuffed with
parentheticals or PR/ticket numbers (those live in the weekly file). When you add a new week,
re-trim the older weeks; older entries should get shorter as they age, not stay at original
length. When this section exceeds 3 weeks, move the oldest entry to Archived Context._

### [Week of YYYY-MM-DD](weekly/YYYY-MM-DD.md)

- [Highlight 1]
- [Highlight 2]
- [Highlight 3]

---

## Key Decisions Log

_Curated, not append-only. Captures decisions that still shape current work. Each entry is
1-2 lines, dated. Rules:_

- _When a decision is superseded, mark it superseded inline and link to the replacement._
- _When a decision is older than ~8 weeks AND no longer cited in current work, move it to
  Archived Context (or to a `decisions-archive.md` file in the bucket folder)._
- _Tactical implementation notes (PRs, version bumps) are NOT decisions. A decision changes
  direction, scope, ownership, timeline, or methodology._

- **YYYY-MM-DD**: [Decision] -- [Brief rationale / context]

---

## Archived Context

_Condensed older material. Merged from weeks that rolled out of Recent Weeks and from
retired decisions. Keep only what's still relevant to understanding the current state. If
this exceeds ~40 lines, condense further._

[Condensed older context]
