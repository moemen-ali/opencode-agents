# opencode-agents

A collection of custom [opencode](https://opencode.ai) agents for real-world development workflows.

Agents are plain markdown files — no installs, no plugins. Drop one into your project and opencode picks it up automatically.

---

## Agents

| Agent | Description |
|-------|-------------|
| [`angular-review`](./angular-review.md) | Angular 19+ code review agent. Diffs your branch, optionally validates against Jira or a spec file, and writes a timestamped report to `code-review/` |
| [`bug-report`](./bug-report/) | Jira bug report orchestrator. Fetches bugs under an epic or story, writes a structured markdown report, and optionally invokes AI subagents for pattern analysis, health scoring, blocker prediction, release notes, and duplicate detection |

---

## `angular-review`

### Installation

**Project-specific** — scoped to one repo, committed and shared with your team:

```bash
mkdir -p .opencode/agents && curl -sL https://raw.githubusercontent.com/moemen-ali/opencode-agents/main/angular-review.md -o .opencode/agents/angular-review.md
```

**Global** — available across all your projects:

```bash
mkdir -p ~/.config/opencode/agents && curl -sL https://raw.githubusercontent.com/moemen-ali/opencode-agents/main/angular-review.md -o ~/.config/opencode/agents/angular-review.md
```

### Usage

Open opencode in your project, then press `Tab` to cycle to the `angular-review` agent.

The agent will guide you through the full flow automatically:

1. Shows your current branch and available branches
2. Asks which branch to diff against
3. Optionally fetches your Jira ticket or reads a spec file for business validation
4. Reviews all changed files against the Angular 19+ checklist
5. Writes the report to `code-review/<branch>-vs-<target>-<date>.md`

### Jira Integration (optional)

To enable automatic ticket fetching from Jira, add the following to your `opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "jira": {
      "type": "local",
      "command": ["npx", "-y", "@aashari/mcp-server-atlassian-jira"],
      "environment": {
        "ATLASSIAN_SITE_NAME": "your-company",
        "ATLASSIAN_USER_EMAIL": "you@yourcompany.com",
        "ATLASSIAN_API_TOKEN": "your_api_token"
      }
    }
  }
}
```

Get your API token at [https://id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens).

> The agent works fine without Jira — it will offer spec file or manual paste as alternatives.

### Business Validation

When the agent asks about business context, you have four options:

| Option | What it does |
|--------|-------------|
| **Jira** | Extracts ticket number from branch name (e.g. `feature/PROJ-123`) and fetches it via MCP |
| **Spec file** | Reads a local file you point to (e.g. `docs/specs/auth.md`) |
| **Paste** | You paste the user story or acceptance criteria directly |
| **Skip** | Technical review only |

### Report Output

Reports are written to `code-review/` with the naming convention:

```
code-review/<current-branch>-vs-<target-branch>-<YYYY-MM-DD>.md
```

Each report includes:

- List of changed files
- Per-file review with 🔴 Critical / 🟡 Warning / 🔵 Suggestion issues and line references
- Refactor previews for non-obvious fixes
- Business validation table (if context was provided)
- Overall summary with top actionable next steps

---

## `bug-report`

A Jira bug report orchestrator with optional AI subagents for deeper analysis. The main agent handles all Jira I/O — subagents receive a pre-built data payload and do pure AI reasoning with zero additional API calls.

### How it works

Run `bug-report` against any epic or story key. It:

1. Detects the issue key from your branch name (or confirms with you)
2. Resolves your identity via `jira_get_myself` — `currentUser()` doesn't work with API token auth
3. Confirms the issue key, assignee scope, and date window before touching Jira
4. Discovers the exact bug type name used in your project before querying (handles custom names like `"Bug - Subtask"`)
5. Fetches all bugs and writes a structured report to a timestamped run directory

Pass flags to enable AI subagents after the report is written:

| Flag | What it adds |
|------|-------------|
| `--analyze` | Clusters bugs by root cause and affected component — derived entirely from your actual bug data |
| `--score` | Gives the epic a health score (0–100) with a plain-English summary ready for standup |
| `--blockers` | Flags what will prevent shipping, with specific keys and assignees from your data |
| `--release-notes` | Writes an internal QA summary + external release notes draft, both derived from the payload |
| `--duplicates` | Clusters bugs likely filed twice, with confidence levels and specific evidence |
| `--all` | Runs all five subagents |

### Output structure

Each run creates a directory inside `bug-report/`:

```
bug-report/
└── {ISSUE_KEY}_{YYYY-MM-DD}/
    ├── bug-report.md           ← always generated
    ├── pattern-analysis.md     ← --analyze
    ├── health-score.md         ← --score
    ├── blocker-report.md       ← --blockers
    ├── release-notes.md        ← --release-notes
    └── duplicate-clusters.md   ← --duplicates
```

### Files

| File | Role |
|------|------|
| [`bug-report/bug-report.md`](./bug-report/bug-report.md) | Primary agent — orchestrates everything |
| [`bug-report/bug-analyzer.md`](./bug-report/bug-analyzer.md) | Subagent — pattern analysis |
| [`bug-report/epic-scorer.md`](./bug-report/epic-scorer.md) | Subagent — health scoring |
| [`bug-report/blocker-predictor.md`](./bug-report/blocker-predictor.md) | Subagent — blocker prediction |
| [`bug-report/release-writer.md`](./bug-report/release-writer.md) | Subagent — release notes |
| [`bug-report/duplicate-detector.md`](./bug-report/duplicate-detector.md) | Subagent — duplicate detection |

### Installation

The agent is 6 files. They all go flat inside `.opencode/agents/` — OpenCode does not support subdirectories inside that folder.

**Project-specific:**

```bash
mkdir -p .opencode/agents

BASE="https://raw.githubusercontent.com/moemen-ali/opencode-agents/main/bug-report"

curl -sL $BASE/bug-report.md         -o .opencode/agents/bug-report.md
curl -sL $BASE/bug-analyzer.md       -o .opencode/agents/bug-analyzer.md
curl -sL $BASE/epic-scorer.md        -o .opencode/agents/epic-scorer.md
curl -sL $BASE/blocker-predictor.md  -o .opencode/agents/blocker-predictor.md
curl -sL $BASE/release-writer.md     -o .opencode/agents/release-writer.md
curl -sL $BASE/duplicate-detector.md -o .opencode/agents/duplicate-detector.md
```

**Global:**

```bash
mkdir -p ~/.config/opencode/agents

BASE="https://raw.githubusercontent.com/moemen-ali/opencode-agents/main/bug-report"

curl -sL $BASE/bug-report.md         -o ~/.config/opencode/agents/bug-report.md
curl -sL $BASE/bug-analyzer.md       -o ~/.config/opencode/agents/bug-analyzer.md
curl -sL $BASE/epic-scorer.md        -o ~/.config/opencode/agents/epic-scorer.md
curl -sL $BASE/blocker-predictor.md  -o ~/.config/opencode/agents/blocker-predictor.md
curl -sL $BASE/release-writer.md     -o ~/.config/opencode/agents/release-writer.md
curl -sL $BASE/duplicate-detector.md -o ~/.config/opencode/agents/duplicate-detector.md
```

### Jira MCP setup (required)

Add this to your `opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "jira": {
      "type": "local",
      "command": ["npx", "-y", "@aashari/mcp-server-atlassian-jira"],
      "environment": {
        "ATLASSIAN_SITE_NAME": "your-company",
        "ATLASSIAN_USER_EMAIL": "you@yourcompany.com",
        "ATLASSIAN_API_TOKEN": "your_api_token"
      }
    }
  }
}
```

Get your API token at [https://id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens).

### Usage

Press `Tab` in opencode to switch to the `bug-report` agent. The confirmation prompt appears before any Jira call:

```
🔍 Ready to run bug report. Please confirm or override:

  Issue key   : {ISSUE_KEY}  ← from branch "{your-branch-name}"
  Assignee    : {your display name} (default)
  Date window : last 30 days (default)
  Subagents   : --analyze --score

Reply with "yes" to proceed, or override inline (e.g. "{PROJECT}-999 all 60")
```

---

## Customizing for your stack

All agents are plain markdown — the system prompt is just text. To adapt them for a different workflow or framework, open the `.md` file and edit the relevant sections directly.

---

## Contributing

PRs welcome. If you've built an agent for a different workflow (security audit, performance review, accessibility check, test generation), open a PR and add it to the collection.

---

## Author

Built by [Moemen Ali](https://moemen-elsayeh.netlify.app) · [Dev.to](https://dev.to/moemenali) · [LinkedIn](https://www.linkedin.com/in/moemenali)