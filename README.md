# Skills

Agent skills for reviewing and improving code. They follow the [Agent Skills](https://agentskills.io) format and work with any agent the [skills CLI](https://github.com/vercel-labs/skills) supports, including Claude Code, Codex, Cursor, and OpenCode.

## Available skills

### clean-comments

Reviews comments and docstrings in the current branch's changes. It deletes noise, keeps comments that explain why, and rewrites vague ones. It proposes every change and waits for your approval before editing.

**Use when:**

- "Tidy the comments on this branch before I open the PR"
- "Clean up these AI-generated docstrings"
- "Are any of these comments stale?"

**Requires:** git.

### improve-codebase

Explores a target through the lenses you pick, presents candidates, designs alternatives for the one you choose, and writes an RFC. It changes no source files before the RFC step.

| Mode | Target | Lenses |
| --- | --- | --- |
| `code` | Source repository | Architecture, testability, performance, security, readability, duplication, dead code, maintainability |
| `docs` | Documentation folder | Accuracy, structure, clarity, consistency |
| `ui` | Running web frontend | Responsive layout, usability, visual consistency |

**Use when:**

- "Where should we refactor? This repo is getting hard to test"
- "Our docs are stale and hard to navigate"
- "Review the UI at http://localhost:3000, the mobile view looks broken"

**Requires:** git. `ui` mode needs a Playwright MCP server. Publishing an RFC as an issue needs `gh` or `glab`. Runs lenses in parallel where the agent supports subagents, and one at a time otherwise.

## Install

Requires Node.js 22.20 or later.

```bash
npx skills add chrisleekr/skills
```

Pick the skills and agents when prompted. Skills install to the current project by default. To install one skill for all your projects:

```bash
npx skills add chrisleekr/skills --skill clean-comments -g -a claude-code
```

To pin a release:

```bash
npx skills add chrisleekr/skills@v1.0.0
```

To try a skill without installing it:

```bash
npx skills use chrisleekr/skills@clean-comments --agent claude-code
```

### Claude Code plugin

```text
/plugin marketplace add chrisleekr/skills
/plugin install clean-comments@chrisleekr-skills
/plugin install improve-codebase@chrisleekr-skills
```

Use one install method per skill. Claude Code loads a plugin skill alongside a plain copy of the same name, so both would appear.

## Use

Ask in your own words, as in the examples above, and the agent loads the matching skill. You can also name a skill directly. In Claude Code, run `/clean-comments`, or `/clean-comments:clean-comments` for the plugin.

## Update and remove

```bash
npx skills check
```

```bash
npx skills update
```

```bash
npx skills remove clean-comments
```

For plugins, background auto-update is off by default. To turn it on, open **Marketplaces** in `/plugin`, select `chrisleekr-skills`, and select **Enable auto-update**.

## Security

Skills are instructions your agent follows with its own permissions. Read a skill before installing it. These skills contain no scripts, and only `improve-codebase` publishes anything: it creates an issue through `gh` or `glab` if you choose that output.

## Repository layout

```text
skills/<name>/SKILL.md      Instructions loaded when the skill activates
skills/<name>/references/   Detail loaded only when needed
skills/<name>/evals/        Test prompts and expected results
```

## Contributing

See [AGENTS.md](AGENTS.md) for conventions and how to add a skill.

## License

[MIT](LICENSE). See [third-party notices](THIRD_PARTY_NOTICES.md).
