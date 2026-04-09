# Bucket Config

## Identity

- **Name**: [Human-readable name, e.g., "Project Atlas", "1:1 with Jordan Lee", "Platform Migration"]
- **Slug**: [folder name, e.g., "project-atlas", "1on1-jordan-lee", "initiative-platform-migration"]
- **Description**: [One line on what this bucket tracks and why it exists]

## Participants

- [Name] -- [Role/relationship, e.g., "Tech Lead", "Client PM", "my direct report"]

## Event Sources

_Each source declares: which connector tool to use, how to find relevant events, and any
filtering criteria. Only include sources that are actually connected and relevant._

### Fathom Meetings
- **Enabled**: [yes | no]
- **Search patterns**: [meeting title keywords to match, e.g., "Atlas DSW", "Atlas Demo"]
- **Filters**: [optional: recorded_by, teams, domains]

### GitHub
- **Enabled**: [yes | no]
- **Repositories**: [org/repo names to track]
- **Event types**: [PRs, issues, reviews, merges -- which ones matter for this bucket]

### Google Drive
- **Enabled**: [yes | no]
- **Search queries**: [keywords, folder names, or doc title patterns]
- **File types**: [docs, sheets, slides -- which ones to look for]

### Slack
- **Enabled**: [yes | no]
- **Channels**: [channel names to monitor, e.g., "#project-atlas", "#atlas-eng"]
- **Search queries**: [additional keyword filters if needed]

### Other: [Source Name]
- **Enabled**: [yes | no]
- **Tool**: [which MCP tool to use]
- **How to find events**: [describe the search/list approach]

_Repeat the "Other" block for any additional sources._

## Notes

_Additional context that should inform how summaries are written for this bucket.
Things like: "Client is sensitive about timeline slips", "Jordan is ramping up on the
auth service -- track progress specifically", "This 1:1 focuses on career growth, not
project work."_

- [Optional notes]
