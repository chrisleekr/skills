<h1 align="center">Skills</h1>

<p align="center">
  chrisleekr's agent skills
</p>

<p align="center">
  <a href="https://github.com/chrisleekr/skills/releases"><img alt="Latest release" src="https://img.shields.io/github/v/release/chrisleekr/skills"></a>
  <a href="https://agentskills.io"><img alt="Agent Skills format" src="https://img.shields.io/badge/Agent%20Skills-compatible-2c3e50"></a>
  <a href="LICENSE"><img alt="MIT license" src="https://img.shields.io/github/license/chrisleekr/skills"></a>
</p>

```bash
npx skills add chrisleekr/skills
```

Works with any agent the [skills CLI](https://github.com/vercel-labs/skills) supports, including Claude Code, Codex, Cursor, and OpenCode.

## Available skills

### Code review

| Skill | Use it to | Needs |
| --- | --- | --- |
| [clean-comments](skills/clean-comments/SKILL.md) | Tidy comments and docstrings on your branch before a PR | git |
| [pr-review-panel](skills/pr-review-panel/SKILL.md) | Review a diff through parallel correctness, security, and tests-and-docs lenses | git |

### Maintenance

| Skill | Use it to | Needs |
| --- | --- | --- |
| [improve-codebase](skills/improve-codebase/SKILL.md) | Audit code, docs, or a running web UI and write an RFC of improvements | git. `ui` mode needs a Playwright MCP server. Issue output needs `gh` or `glab` |
| [update-packages-and-tooling](skills/update-packages-and-tooling/SKILL.md) | Bump packages, runtimes, and CI pins in risk order, with one PR | git, the repo's package tools. Opening a PR or MR needs `gh`, `glab` or similar |

### Writing

| Skill | Use it to | Needs |
| --- | --- | --- |
| [humanizer](skills/humanizer/SKILL.md) | Rewrite AI-sounding text without changing what it says | Nothing. Optional local model |

## Install

Requires Node.js 22.20 or later. The command above asks which skills and agents to use, and installs to the current project by default.

To install one skill for all your projects:

```bash
npx skills add chrisleekr/skills --skill clean-comments -g -a claude-code
```

To pin a release:

```bash
npx skills add chrisleekr/skills#v1.0.0
```

To try a skill without installing it:

```bash
npx skills use chrisleekr/skills@clean-comments --agent claude-code
```

### Claude Code plugin

```text
/plugin marketplace add chrisleekr/skills
/plugin install <skill>@chrisleekr-skills
```

Use one install method per skill. Claude Code loads a plugin skill alongside a plain copy of the same name, so both would appear.

## Use

Ask in your own words and the agent loads the matching skill. You can also name a skill directly. In Claude Code, run `/clean-comments`, or `/clean-comments:clean-comments` for the plugin.

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

Skills are instructions your agent follows with its own permissions. Read a skill before installing it, and see each skill's audits on [skills.sh](https://skills.sh/chrisleekr/skills).

These skills contain no scripts. Two can publish, each only with your approval: `improve-codebase` creates an issue if you choose that output, and `update-packages-and-tooling` pushes a branch, opens a PR or MR, and can merge it. `update-packages-and-tooling` also runs your repository's own checks without asking first, so read them before using it on a repository you do not trust. `humanizer`'s optional model setup downloads third-party code.

## Repository layout

```text
skills/<name>/SKILL.md      Instructions loaded when the skill activates
skills/<name>/references/   Detail loaded only when needed
skills/<name>/evals/        Test prompts and expected results
skills/<name>/NOTICE.md     Third-party attribution, for adapted skills
```

## Contributing

See [AGENTS.md](AGENTS.md) for conventions and how to add a skill.

## License

[MIT](LICENSE). See [third-party notices](THIRD_PARTY_NOTICES.md).
