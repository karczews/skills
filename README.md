# skills

Agent skills for [Claude Code](https://code.claude.com) and [OpenCode](https://opencode.ai). Each plugin lives under `plugins/<name>/` and is listed in `.claude-plugin/marketplace.json`. The `explain` plugin holds two skills, `explain` and `diagramize`; the `code-simplifier` plugin holds one.

## Installation

Claude Code, as a plugin (install either plugin on its own):

```text
/plugin marketplace add karczews/skills
/plugin install explain@karczews-skills
/plugin install code-simplifier@karczews-skills
```

The `explain` plugin installs both `explain` and `diagramize`.

Any agent the [`skills` CLI](https://github.com/vercel-labs/skills) supports, including OpenCode:

```bash
npx skills add karczews/skills --skill explain --skill diagramize -a opencode   # explain cannot draw without diagramize
npx skills add karczews/skills --skill code-simplifier -a opencode
```

Or copy a skill's directory from `plugins/<plugin>/skills/` into `~/.config/opencode/skills/` (OpenCode) or `~/.claude/skills/` (both read it). `explain` needs `diagramize` next to it.

## Available skills

### explain

Explains the current topic, or the topic you name, with a one- or two-sentence answer and one to three visuals sized to the question: a call tree, pseudocode, a text sequence diagram, a file or structure tree, or a diff of one of those. It reads the code before it draws, uses real names with `path:line`, and marks unverified steps with `(?)`. Before answering, it asks whether to show the answer in the terminal or on an HTML page, and recommends the page when it needs more than three parts, overflows terminal limits, or is an HTML-only kind (UI, dense graph). The prose names the mechanism and the actor in plain words, with no filler.

It runs only when you ask for it:

```text
/explain:explain how a request reaches the database   # Claude Code, plugin install
/explain how a request reaches the database           # Claude Code, standalone skill
@explain how a request reaches the database           # OpenCode
```

Ask a follow-up and it extends the previous diagram in the same place as before: as a diff of a text diagram, or as the whole Mermaid diagram again with the new parts marked `(new)`. End the request with `in html` or `in terminal` to skip the question, with `to docs` or `to <file>.md` to save the answer as markdown with Mermaid diagrams, or with `to <file>.html` to save the HTML page. `diagramize` draws the diagrams.

### diagramize

Draws a diagram in the form the place it is read can show: plain text with box-drawing glyphs for a terminal, and Mermaid or plain text for a Markdown file or an HTML page. It picks the kind of diagram from what it has to show, writes Mermaid labels so they parse, and checks every Mermaid block with `mermaid-cli` when it can. Mermaid colours live in `plugins/explain/skills/diagramize/references/`: `mermaid-light-frontmatter.yaml` is the light config diagramize copies into Markdown files, `mermaid-light.json` for light HTML pages and `mermaid-dark.json` for dark HTML pages; `page-template.html` is the base HTML page. [`docs/diagramize/preview.md`](docs/diagramize/preview.md) shows each kind as it appears in a Markdown file, and [`docs/diagramize/preview.html`](docs/diagramize/preview.html), opened in a browser, shows the exact look of HTML pages in light and dark mode.

After changing a config or a `docs/diagramize/*.mmd` source, run `scripts/render-diagramize-preview.py` to rebuild the frontmatter file and both previews, or enable the pre-commit hook that runs it: `git config core.hooksPath .githooks`.

`explain` calls it for every diagram. You can also call it yourself, and end the request with `in terminal`, `in html`, `to <file>.md` or `to <file>.html`:

```text
/explain:diagramize the login flow in html   # Claude Code, plugin install
/diagramize the login flow in html           # Claude Code, standalone skill
@diagramize the login flow in html           # OpenCode
```

### code-simplifier

Simplifies recently changed code as a pure refactor: outputs, side effects, errors and public signatures stay the same. It first reads the project's `AGENTS.md` or `CLAUDE.md`, linter configs and the code around each change, then flattens nesting, removes redundant code and comments that restate the code, renames for clarity and turns nested ternaries into if/else or switch. It runs the tests and checks that cover the change and reverts any edit that breaks one. When the host can launch subagents, it hands the pass to one so the code gets fresh eyes, then checks each edit the subagent reports.

It runs only when you ask for it, optionally naming files, a directory, a commit range or a branch:

```text
/code-simplifier:code-simplifier src/auth   # Claude Code, plugin install
/code-simplifier src/auth                   # Claude Code, standalone skill
@code-simplifier src/auth                   # OpenCode
```

With nothing named, it works on the code changed in the session, or else on uncommitted changes. Claude Code also has a built-in `/simplify` with a similar job; this skill is mainly for OpenCode and for one behavior across both hosts.
## Adding a skill

One skill per plugin by default; a plugin may bundle skills when one cannot run without the other, as `explain` bundles `diagramize`:

```text
plugins/<name>/
├── .claude-plugin/plugin.json      # name, description, version, author
└── skills/<name>/SKILL.md          # plus references/ if it needs them
# or, bundled:
plugins/explain/
├── .claude-plugin/plugin.json
└── skills/{explain,diagramize}/SKILL.md
```

Add a matching entry to `.claude-plugin/marketplace.json` with `"source": "./plugins/<name>"`. Keep `name` the same in the directory, `plugin.json`, the main skill's `SKILL.md` and the marketplace entry, and bump `version` in both manifests on every change. Check with:

```bash
claude plugin validate .
```
