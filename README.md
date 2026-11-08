# skills

Agent skills for [Claude Code](https://code.claude.com) and [OpenCode](https://opencode.ai). Each plugin lives under `plugins/<name>/` and is listed in `.claude-plugin/marketplace.json`. The `explain` plugin holds two skills, `explain` and `diagramize`.

## Installation

Claude Code, as a plugin:

```text
/plugin marketplace add karczews/skills
/plugin install explain@karczews-skills
```

The plugin installs both `explain` and `diagramize`.

Any agent the [`skills` CLI](https://github.com/vercel-labs/skills) supports, including OpenCode:

```bash
npx skills add karczews/skills --skill explain -a opencode
npx skills add karczews/skills --skill diagramize -a opencode
```

Or copy each directory under `plugins/explain/skills/` into `~/.config/opencode/skills/` (OpenCode) or `~/.claude/skills/` (both read it). `explain` needs `diagramize` next to it.

## Available skills

### explain

Explains the current topic, or the topic you name, with a one- or two-sentence answer and a plain-text visual sized to the question: a call tree, pseudocode, a text sequence diagram, a file or structure tree, or a diff of one of those. It reads the code before it draws, uses real names with `path:line`, and marks unverified steps with `(?)`. Before answering, it asks whether to show the answer in the terminal, as plain-text diagrams, or on an HTML page, and recommends the page when the explanation is big: several visuals, a visual too long or wide for a terminal, a UI, or a dense graph. The prose names the mechanism and the actor in plain words, with no filler.

It runs only when you ask for it:

```text
/explain:explain how a request reaches the database   # Claude Code, plugin install
/explain how a request reaches the database           # Claude Code, standalone skill
@explain how a request reaches the database           # OpenCode
```

Ask a follow-up and it extends the previous diagram in the same place as before: as a diff of a text diagram, or as the whole Mermaid diagram again with the new parts marked `(new)`. End the request with `in html` or `in terminal` to skip the question, or with `to docs` or `to <file>.md` to save the answer as markdown, with Mermaid diagrams. `diagramize` draws the diagrams.

### diagramize

Draws a diagram in the form the place it is read can show: plain text with box-drawing glyphs for a terminal, and Mermaid for a Markdown file or an HTML page. It picks the kind of diagram from what it has to show, writes Mermaid labels so they parse, and checks every Mermaid block with `mermaid-cli` when it can. Mermaid colours live in `plugins/explain/skills/diagramize/references/`: `mermaid-light.json` for Markdown files and light HTML pages, `mermaid-dark.json` for dark HTML pages, and `mermaid-light-frontmatter.yaml`, the light config as the frontmatter diagramize copies into Markdown files. [`docs/diagramize/preview.md`](docs/diagramize/preview.md) shows each kind as it appears in a Markdown file, and [`docs/diagramize/preview.html`](docs/diagramize/preview.html), opened in a browser, shows the exact look of HTML pages in light and dark mode.

`scripts/render-diagramize-preview.py` writes the frontmatter file and both previews from the two configs and `docs/diagramize/*.mmd`, after `mermaid-cli` draws every diagram. Run it after changing a config or a source, or enable the pre-commit hook that runs it: `git config core.hooksPath .githooks`.

`explain` calls it for every diagram. You can also call it yourself, and end the request with `in terminal`, `in html` or `to <file>.md`:

```text
/explain:diagramize the login flow in html   # Claude Code, plugin install
/diagramize the login flow in html           # Claude Code, standalone skill
@diagramize the login flow in html           # OpenCode
```

## Adding a skill

One skill per plugin, unless a plugin is meant to bundle several skills, as `explain` bundles `diagramize`:

```text
plugins/<name>/
├── .claude-plugin/plugin.json      # name, description, version, author
└── skills/<name>/SKILL.md          # plus references/ if it needs them
```

Add a matching entry to `.claude-plugin/marketplace.json` with `"source": "./plugins/<name>"`. Keep `name` the same in the directory, `plugin.json`, the main skill's `SKILL.md` and the marketplace entry, and bump `version` in both manifests on every change. Check with:

```bash
claude plugin validate .
```
