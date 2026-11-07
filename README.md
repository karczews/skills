# skills

Agent skills for [Claude Code](https://code.claude.com) and [OpenCode](https://opencode.ai). Each skill is its own plugin under `plugins/<name>/`, listed in `.claude-plugin/marketplace.json`.

## Installation

Claude Code, as a plugin:

```text
/plugin marketplace add karczews/skills
/plugin install explain@karczews-skills
```

Any agent the [`skills` CLI](https://github.com/vercel-labs/skills) supports, including OpenCode:

```bash
npx skills add karczews/skills --skill explain -a opencode
```

Or copy `plugins/<name>/skills/<name>/` into `~/.config/opencode/skills/` (OpenCode) or `~/.claude/skills/` (both read it).

## Available skills

### explain

Explains the current topic, or the topic you name, with a one- or two-sentence answer and a plain-text visual sized to the question: a call tree, pseudocode, a text sequence diagram, a file or structure tree, or a diff of one of those. It reads the code before it draws, uses real names with `path:line`, and marks unverified steps with `(?)`. Before answering, it asks whether to show the answer in the terminal, as plain-text diagrams, or on an HTML page, and recommends the page when the explanation is big: several visuals, a visual too long or wide for a terminal, a UI, or a dense graph. The prose names the mechanism and the actor in plain words, with no filler.

It runs only when you ask for it:

```text
/explain:explain how a request reaches the database   # Claude Code, plugin install
/explain how a request reaches the database           # Claude Code, standalone skill
@explain how a request reaches the database           # OpenCode
```

Ask a follow-up and it extends the previous diagram as a diff, in the same place as before. End the request with `in html` or `in terminal` to skip the question, or with `to docs` or `to <file>.md` to save the answer as markdown, with Mermaid diagrams.

## Adding a skill

One skill per plugin, unless a plugin is meant to bundle several skills:

```text
plugins/<name>/
├── .claude-plugin/plugin.json      # name, description, version, author
└── skills/<name>/SKILL.md          # plus references/ if it needs them
```

Add a matching entry to `.claude-plugin/marketplace.json` with `"source": "./plugins/<name>"`. Keep `name` the same in the directory, `plugin.json`, `SKILL.md` and the marketplace entry, and bump `version` in both manifests on every change. Check with:

```bash
claude plugin validate .
```
