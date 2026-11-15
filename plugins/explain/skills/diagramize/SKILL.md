---
name: diagramize
description: Draw a diagram in the form the place it is read can show, plain text for a terminal and Mermaid or plain text for a Markdown file or an HTML page. Use when a skill or the user needs a diagram drawn for one of those places.
---

# diagramize

The caller (a skill or the user) decides what the diagram holds and where it goes. This skill picks the kind and draws it. Configs live beside this file in `references/`: always read them, never reconstruct from memory.

## 1. Inputs

- Content: parts, steps or states and their labels. Use given names verbatim; keep `path:line` and `(?)` where the caller put them. Add nothing except UI details step 5 reads from code.
- Destination: terminal, Markdown file, or HTML page. Direct call with none given uses the terminal; a caller-given destination always wins.
- Kind: the row of step 2 fitting the content, or the kind the caller names.

Direct call: text after the invocation names the subject; a trailing `in terminal`, `in html`, `to <file>.md` or `to <file>.html` names the destination (`to docs` is `explain`-only). Suffixes match case-insensitively at the very end only. Read the subject's code so the diagram carries real names; do not invent parts the code does not show.

## 2. Pick the kind

| The diagram shows…                                   | Terminal                    | Markdown file or HTML page          |
|------------------------------------------------------|-----------------------------|-------------------------------------|
| what calls what at runtime                           | call tree                   | call tree, as text                  |
| what the logic decides, step by step                 | pseudocode                  | `flowchart TD` when branches matter, else pseudocode |
| who talks to whom, in what order                     | sequence diagram in text    | `sequenceDiagram`                   |
| what states something can be in, and what moves it   | state diagram in text       | `stateDiagram-v2`                   |
| where something lives, which module owns what        | shallow file tree           | file tree, as text                  |
| how pieces compose (UI components, classes, modules) | structure tree              | structure tree, or `classDiagram` for types and inheritance |
| records and how they relate                          | one real example, annotated | `erDiagram`                         |
| what the data looks like                             | one real example, annotated | the same example                    |
| which option to pick                                 | comparison table            | comparison table                    |
| what changes, or what would change                   | before and after (step 3)   | before and after, in the same kinds |
| how a UI looks, or a graph too dense for text        | does not fit, say so        | HTML page only (step 5)             |

A chain with no branch, fan-out, loop and a single actor stays text in every destination; this overrides the Mermaid column.

## 3. Terminal

The terminal shows Mermaid as source: use `text`, `diff`, `jsonc` blocks or Markdown tables, ≤20 lines × 80 cols. Cut branches to `…` and name omissions. When it cannot fit, say so and suggest HTML.

Call tree:

```text
handleUpload                  api/upload.go:31
  validateFile                api/validate.go:12
  storeBlob
    pickBackend (?)           # backend chosen from config at runtime
  enqueueThumbnail
```

Sequence diagram in text:

```text
User          CLI            API
 │  run job    │              │
 │────────────▶│  POST /jobs  │
 │             │─────────────▶│
 │             │◀── 202, id ──│
 │◀─── id ─────│              │
```

Other shapes follow the same density; see `docs/diagramize/preview.md` for glyphs. Pseudocode nests with two-space indent; state moves use `──label──▶`; file/structure trees use `├──`/`└──`; examples come from a fixture, log line or literal with a comment per unclear field; tables have one option per column, one separating criterion per row.

Before and after (combined ≤20 lines). Pick the form that stays readable:

- A `diff` of the shape when the change adds/removes parts and leaves the rest.
- `Before`/`After` visuals of the same kind when it reorders steps or swaps actors. In a text pair, `After` prefixes added lines `+` and changed lines `~`; removed lines appear only in `Before`.
- A `Before`/`After`-column table when it is about behaviour, not location. For a fix mark broken `✗` and replacement `✓`.

```diff
 internal/
 ├── auth/
+│   └── refresh.go    # retries expired tokens
 └── storage/
-    └── s3.go
+    ├── backend.go    # common interface
+    └── gcs.go
```

## 4. Markdown file

- Draw the right-column kinds as Mermaid blocks; trees, pseudocode, tables, diffs and examples keep terminal shapes.
- Start each Mermaid block with `references/mermaid-light-frontmatter.yaml` (in `references/` next to `SKILL.md`'s directory), copied unchanged:

  ```mermaid
  ---
  config:
    …                  # the rest of mermaid-light-frontmatter.yaml
  ---
  flowchart TD
    …
  ```

- One caption line under each Mermaid block (fallback for source-only viewers).
- Follow step 6 and check per step 7.

## 5. HTML page

- Build from `references/page-template.html` (next to this file in `references/`). Paste the answer/notes around the diagrams unchanged, and the two `references/mermaid-*.json` configs where marked. Inline configs; do not fetch.
- HTML-escape every diagram source inside `<pre>` (as the generator does with `html.escape()`). Read sources back with `textContent`, never `innerHTML`, so `<`, `<<interface>>` and `-->` survive.
- Mermaid kinds go in `<pre class="mermaid">`; trees, pseudocode, diffs and examples in `<pre class="text">`; comparison and before/after tables as HTML `<table>`. Mermaid blocks keep the `references/` configs. No 20-line limit: draw the whole path instead of cutting to `…`, but still split when it answers more than one question.
- For a UI use real labels, values, states, colours, fonts and component names from the project's theme; read that code when the caller did not hand it over. For a UI change show before/after side by side, stacked on narrow windows. Fit a narrow window.
- Write to the caller's path; only for a direct call default to `${TMPDIR:-/tmp}/diagramize/<topic-slug>.html`, outside the repo. Follow step 6 and check per step 7. Open with `open` (macOS) / `xdg-open` (Linux), or the harness HTML viewer when it has one. Hand the path back to the caller.

## 6. Mermaid rules

One broken label turns a block into an error; for full syntax see https://mermaid.js.org. Project rules:

- Quote `flowchart` and `erDiagram` relationship labels (`A["parse(file)"]`, `-->|"on timeout"|`, `JOB ||--o{ ATTEMPT : "has many"`); leave sequence/state/class labels bare (quotes show as text). Bare labels hold brackets and parentheses; a class-relation colon is `#58;`. Declare spaced state names as `state "Waiting for token" as waiting`.
- Escape in every label: `#59;` for `;`, `#35;` for `#`, `#lt;` for `<`, `#quot;` for `"` inside quoted labels. Break lines with `<br/>`.
- Write `End`, not `end`; start comments with `%%` at line start. Node shapes: `["step"]`, `{"decision"}`, `(["start/end"])`, `[("store")]`; boundaries as `subgraph`. Sequence: `->>` call, `-->>` return, `-x` failed; ordering reasons in `Note over A,B:`.
- Keep ~dozen nodes; split when it answers more than one question. Mark new parts `(new)`, never colour alone. Only step-2 types; no `architecture-beta`, no `click` links (strict viewers disable them); put links beside the diagram.

## 7. Check that it renders

When `npx` exists, check every Mermaid block before handing back. One command, absolute path for `FILE.md`:

```bash
out="$(mktemp -d)" && (cd "$out" && npx -y --prefer-offline -p puppeteer@25 -p @mermaid-js/mermaid-cli@12.2.0 mmdc -q -i "FILE.md" -o "$out/check.md") && ls "$out"
```

- Pass the whole Markdown file (checks every ```` ```mermaid ```` fence, including indented ones). For HTML, copy sources into a scratch `.md` with one ```` ```mermaid ```` fence each and check that.
- Pass is exit 0 plus one `check-N.svg` per block; a missing file means an unchecked block (usually a wrong fence). Each SVG must contain the frontmatter's `primaryColor`; Mermaid ignores misread configs silently.
- On failure: `Parse error` points into a block, `YAMLException` into frontmatter, else the tool/browser. No Chrome download when installed: prefix `npx` with `PUPPETEER_SKIP_DOWNLOAD=1 PUPPETEER_EXECUTABLE_PATH="<path to Chrome>"` (macOS: `/Applications/Google Chrome.app/Contents/MacOS/Google Chrome`).
- If you cannot run it, say the diagrams are unchecked. Viewers use their own Mermaid version; render a block holding only `info` to see it.

## 8. Extend a diagram

Redraw the same diagram with the new part, same destination: text shapes as a `diff` adding it; Mermaid as the whole diagram again with each new node/message/edge labelled `(new)`. Fresh diagram only when the content needs a different kind.
