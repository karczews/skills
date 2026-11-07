# Repository conventions

This repo is a Claude Code plugin marketplace whose skills must also work in OpenCode.

- Each skill is its own plugin at `plugins/<name>/`, with `.claude-plugin/plugin.json` and `skills/<name>/SKILL.md`. Put several skills in one plugin only when the user asks for it.
- Every plugin has an entry in `.claude-plugin/marketplace.json`. `name` matches across the directory, `plugin.json`, `SKILL.md` frontmatter and the marketplace entry. Bump `version` in both manifests when a plugin changes.
- Write `SKILL.md` for both harnesses: no Claude Code-only tool names or `$ARGUMENTS` placeholders; describe tools by what they do. OpenCode honours `disable-model-invocation: true` and loads skills with `@<name>`.
- Run `claude plugin validate .` after changing a manifest.
- Write commit messages as [Conventional Commits](https://www.conventionalcommits.org/): `<type>(<scope>): <summary>`, for example `feat(explain): ask where to show the answer`. Use `feat`, `fix`, `docs`, `refactor` or `chore` as the type. The scope is the plugin name; leave it out for changes that span the repo.
