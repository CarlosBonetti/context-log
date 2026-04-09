# Bucket Config

## Identity

- **Name**: [Human-readable name, e.g., "Project Atlas", "1:1 with Jordan Lee", "Platform Migration"]
- **Slug**: [folder name, e.g., "project-atlas", "1on1-jordan-lee", "initiative-platform-migration"]
- **Description**: [One line on what this bucket tracks and why it exists]

## Participants

- [Name] -- [Role/relationship, e.g., "Tech Lead", "Client PM", "my direct report"]

## Event Sources

_Only include sources the user asked for. Each source gets its own subsection with
whatever details are needed for the respective connector to find only the relevant
events (search queries, channel names, repo filters, title keywords, etc.). The format
and level of detail will vary by source -- include exactly what's needed, no more.
The examples below are illustrative only -- do not include them in the actual config._

**Examples:**

> ### Fathom Meetings
>
> - **Search patterns**: "Atlas DSW", "Atlas Demo"
> - **Filters**: recorded_by: jordan@acme.com
>
> ### GitHub
>
> - **Repositories**: acme/atlas-backend, acme/atlas-frontend
> - **Event types**: PRs, issues, reviews
>
> ### Google Drive
>
> - **Search queries**: "Project Atlas" in folder "Engineering/Atlas"
> - **File types**: docs, sheets
>
> ### Slack
>
> - **Channels**: #project-atlas, #atlas-eng
> - **Search queries**: "atlas" in #engineering-general

## Notes

_Additional context that should inform how summaries are written for this bucket.
Things like: "Client is sensitive about timeline slips", "Jordan is ramping up on the
auth service -- track progress specifically", "This 1:1 focuses on career growth, not
project work."_

- [Optional notes]
