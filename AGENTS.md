# AGENTS.md

Agent skills distributed with the `skills` CLI and as Claude Code plugins.

## Adding a skill

1. Create `skills/<name>/SKILL.md`. The directory name must equal `name`, in kebab-case.
2. Use only these frontmatter keys: `name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools`.
3. Say what the skill does and when to use it in `description`, and include the words users would type.
4. Keep `SKILL.md` under 500 lines. Move detail to `references/`, and say when to read each file.
5. Add `evals/evals.json` with at least three cases.
6. Add the skill to `.claude-plugin/marketplace.json`, using `"source": "./"`, `"strict": false` and `"skills": ["./skills/<name>"]`, and to the list in `README.md`.

## Portability

- No private paths or references to skills outside this repo.
- Describe the capability first. Name agent-specific tools as examples, and give a fallback.
- Declare real requirements in `compatibility`.

## Releases

release-please opens a Release PR from Conventional Commit titles. PRs are squash-merged, so the PR title sets the bump: `feat:` is minor, and `fix:` or `perf:` is patch. Merging the Release PR tags the release. Use `fix:` for skill wording changes, because `docs:` creates no release.
