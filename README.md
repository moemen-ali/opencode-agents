# opencode-agents

A collection of custom [opencode](https://opencode.ai) agents for real-world development workflows.

Agents are plain markdown files — no installs, no plugins. Drop one into your project and opencode picks it up automatically.

---

## Agents

| Agent | Description |
|-------|-------------|
| [`angular-review`](./angular-review.md) | Angular 19+ code review agent. Diffs your branch, optionally validates against Jira or a spec file, and writes a timestamped report to `code-review/` |

---

## Installation

### One-liner (recommended)

**Project-specific** — scoped to one repo, can be committed and shared with your team:

```bash
mkdir -p .opencode/agents && curl -sL https://raw.githubusercontent.com/moemenali/opencode-agents/main/angular-review.md -o .opencode/agents/angular-review.md
```

**Global** — available across all your projects:

```bash
mkdir -p ~/.config/opencode/agents && curl -sL https://raw.githubusercontent.com/moemenali/opencode-agents/main/angular-review.md -o ~/.config/opencode/agents/angular-review.md
```

### Manual

Download the `.md` file and place it in either:

- `.opencode/agents/` — project-specific
- `~/.config/opencode/agents/` — global

---

## Usage

Open opencode in your project, then press `Tab` to cycle to the `angular-review` agent.

The agent will guide you through the full flow automatically:

1. Shows your current branch and available branches
2. Asks which branch to diff against
3. Optionally fetches your Jira ticket or reads a spec file for business validation
4. Reviews all changed files against the Angular 19+ checklist
5. Writes the report to `code-review/<branch>-vs-<target>-<date>.md`

---

## Jira Integration (optional)

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
        "ATLASSIAN_USER_EMAIL": "your@email.com",
        "ATLASSIAN_API_TOKEN": "your_api_token"
      }
    }
  }
}
```

Get your API token at [https://id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens).

> The agent works fine without Jira — it will offer spec file or manual paste as alternatives.

---

## Business Validation

When the agent asks about business context, you have four options:

| Option | What it does |
|--------|-------------|
| **Jira** | Extracts ticket number from branch name (e.g. `feature/PROJ-123`) and fetches it via MCP |
| **Spec file** | Reads a local file you point to (e.g. `docs/specs/auth.md`) |
| **Paste** | You paste the user story or acceptance criteria directly |
| **Skip** | Technical review only |

The report will include a **Business Validation** section with a table of acceptance criteria and their status (✅ Met / ❌ Missing / ⚠️ Partial).

---

## Report Output

Reports are written to `code-review/` in your project root with the naming convention:

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

## Customizing for Your Stack

The agent is written in plain markdown — the system prompt is just text. To adapt it for a different framework (React, Vue, etc.), open the `.md` file and update the **Review Scope** and **Quick Reference Checklist** sections to match your stack's best practices.

---

## Contributing

PRs welcome. If you've built an agent for a different workflow (security audit, performance review, accessibility check), feel free to open a PR and add it to the collection.

---

## Author

Built by [Moemen Ali](https://moemen-elsayeh.netlify.app) · [Dev.to](https://dev.to/moemenali) · [LinkedIn](https://www.linkedin.com/in/moemenali)
