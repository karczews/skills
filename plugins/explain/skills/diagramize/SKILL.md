---
name: diagramize
description: Draw a diagram in the form the place it is read can show, plain text for a terminal and Mermaid for a Markdown file or an HTML page. Use when a skill or the user needs a diagram drawn for one of those places.
---

# diagramize

Draw a diagram in the form its destination can show. The caller, a skill or the user, decides what the diagram holds and where it goes. This skill picks the kind and draws it.

## 1. Inputs

- Content: the parts, steps or states and their labels. Use the names you are given and spell them the same way. Keep annotations such as `path:line` and `(?)` where the caller put them. Add nothing the caller did not give you.
- Destination: the terminal, a Markdown file, or an HTML page. With none given, use the terminal.
- Kind: pick it in step 2, unless the caller named one.

When the user calls this skill directly, the text after the invocation names the subject, and may end with `in terminal`, `in html` or `to <file>.md` to name the destination. Read the code or files the subject points at, so the diagram carries real names.

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

A chain with no branch, no fan-out, no loop and a single actor stays text in every destination. A Mermaid block for it is a sentence in boxes.

## 3. Terminal

The terminal shows Mermaid as source, so draw with indentation, box-drawing glyphs and arrows, in a `text`, `diff` or `jsonc` block or a Markdown table.

- Stay within 20 lines and 80 columns. Replace omitted branches with `…` and name what was left out.
- When the content cannot fit, such as a UI or a graph too dense for text, say so to the caller and suggest an HTML page.

### Shapes

Call tree:

```text
handleUpload                  api/upload.go:31
  validateFile                api/validate.go:12
  storeBlob
    pickBackend (?)           # backend chosen from config at runtime
  enqueueThumbnail
```

Pseudocode:

```text
on request
  if token has expired
    refresh token
    if refresh fails
      return 401
  forward request
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

State diagram in text:

```text
idle ──start──▶ running ──done──▶ finished
                 │  ▲
            fail │  │ retry, up to 3
                 ▼  │
               retrying ──give up──▶ failed
```

File tree:

```text
internal/
├── auth/         # tokens and sessions
├── billing/      # invoices, talks to the payment provider
└── storage/      # blob backends
```

Structure tree:

```text
CheckoutScreen                ui/checkout/CheckoutScreen.kt:18
  CheckoutViewModel           # holds cart and payment state
  CartSummary
    PriceRow                  ui/components/PriceRow.kt
```

Example data, taken from a test fixture, a log line or a literal in the code, with a comment on each field that needs one:

```jsonc
{
  "id": "job_8f2c",        // IdGenerator, never reused
  "state": "retrying",     // one of the states in JobState
  "attempt": 2,            // starts at 1
  "lastError": "timeout"   // null unless retrying or failed
}
```

Comparison table, one column per option, one row per criterion that separates them:

| | Polling | Webhooks | Queue |
|---|---|---|---|
| Setup | none | public URL | broker to run |
| Latency | up to 30 s | under 1 s | under 1 s |
| Lost events | none | while the receiver is down | none |
| Code touched | `sync/poll.go` | `api/hooks.go` | new worker |

Before and after, for a change that was made or is proposed. Pick the form that keeps both states readable:

- A `diff` of the shape, when the change adds or removes parts and leaves the rest in place.
- Two visuals of the same kind, led by `Before` and `After`, when the change reorders steps or swaps actors, so a diff would be mostly `-` and `+` lines. In a tree or pseudocode pair, prefix lines in `After` with `+` for added and `~` for changed. Removed lines appear only in `Before`. The pair stays within 20 lines.
- A table with `Before` and `After` columns and one row per behaviour that changes, when the change is about what happens rather than where the code lives. For a fix, mark the broken behaviour with `✗` and what replaces it with `✓`.

Diff, keeping the shape the reader already knows:

```diff
 internal/
 ├── auth/
+│   └── refresh.go    # retries expired tokens
 └── storage/
-    └── s3.go
+    ├── backend.go    # common interface
+    ├── s3.go
+    └── gcs.go
```

Before and after pair, where a diff would rewrite every line:

```text
Before
CLI            API
 │  POST /jobs  │
 │─────────────▶│  runs the job, about 40 s
 │◀──── 200 ────│

After
CLI            API          Worker
 │  POST /jobs  │              │
 │─────────────▶│── enqueue ──▶│
 │◀── 202, id ──│              │  runs the job
 │ GET /jobs/id │              │
 │─────────────▶│              │
 │◀─── done ────│              │
```

Before and after table, for a fix:

| | Before | After |
|---|---|---|
| Expired token | ✗ caller gets 401 | ✓ refreshed once, request retried |
| Refresh fails | no refresh attempted | caller gets 401 |
| Requests per call | 1 | up to 2 |

## 4. Markdown file

- Draw the kinds in the right column of step 2 as Mermaid blocks. Trees, pseudocode, tables, diffs and examples keep their terminal shapes.
- Start each Mermaid block with a frontmatter `config` holding the keys of `references/mermaid-light.json`, next to this file, written as YAML. A Markdown block cannot follow the viewer's colour scheme, and the light config keeps all text on light fills, so it reads on light and dark pages:

  ```text
  ---
  config:
    theme: base
    look: classic
    …                  # the rest of references/mermaid-light.json
  ---
  flowchart TD
    …
  ```

- Under each Mermaid block, write one line that says what the diagram shows. Terminal tools, email and many mobile apps show the block as source, and that line is what their readers get.
- Follow the Mermaid rules in step 6 and check the file as step 7 says.

## 5. HTML page

- One self-contained file with inline CSS and JS. Load `https://cdn.jsdelivr.net/npm/mermaid@12.2.0/dist/mermaid.min.js`, and follow the viewer's colour scheme. Read `references/mermaid-light.json` and `references/mermaid-dark.json`, pick one with `matchMedia("(prefers-color-scheme: dark)")`, and pass it to `mermaid.initialize({ startOnLoad: true, deterministicIds: true, ...config })`. Give the page a matching background, `#ffffff` for light and `#0d1117` for dark, with `@media (prefers-color-scheme: dark)`. Without `deterministicIds`, diagrams drawn in the same millisecond get the same id and land on top of each other.
- The 20-line and 80-column limits do not apply. Draw the whole path instead of cutting branches to `…`.
- Draw the kinds in the right column of step 2 as Mermaid. Trees, pseudocode and examples go in `<pre>` blocks; tables become HTML tables.
- When the caller hands you text for the page, such as an answer or notes, put it around the diagrams unchanged.
- For a UI, draw the screens from the real code: actual labels, values and states, with the colours, fonts and component names of the project's theme or design system. For a UI change, show the before and after screens side by side, stacked on a narrow window. Mermaid diagrams keep the configs from `references/`.
- Check that the page fits a narrow window.
- Write it to the path the caller gives, or to `${TMPDIR:-/tmp}/diagramize/<topic-slug>.html`, outside the repository.
- Follow the Mermaid rules in step 6 and check the diagrams as step 7 says.
- Open the page with `open` on macOS or `xdg-open` on Linux. If your harness has a tool that shows HTML to the user, use that instead. Hand the path back to the caller.

## 6. Mermaid rules

One broken label turns a whole block into an error message, so:

- Quote every node and edge label: `A["parse(file)"]`, `-->|"on timeout"|`. Unquoted brackets, parentheses and colons break the parse.
- Break a line inside a quoted label with `<br/>`.
- Write `End`, not `end`, as a flowchart label. Lowercase `end` closes a block.
- Write characters that quoting cannot hold as numeric codes: `#35;` for `#`, `#quot;` for a double quote.
- Start comments with `%%` at the beginning of a line, and keep braces out of them.
- Give each node shape one meaning: `["step"]`, `{"decision"}`, `(["start or end"])`, `[("store")]`. Draw a boundary, such as a process or a permission scope, as a `subgraph`.
- In a `sequenceDiagram`, `->>` is a call, `-->>` a return and `-x` a failed call. Put the reason for an ordering in a `Note over A,B:`.
- Keep a diagram to about a dozen nodes. More means it answers more than one question; split it.
- Mark new or changed parts with a label such as `(new)`. Colour alone is lost on readers who cannot see it and in some themes.
- Use only the types in step 2. Newer types carry exact keywords, such as `architecture-beta`, and many viewers cannot draw them; a `flowchart` with `subgraph` boundaries carries the same information.
- Put links in the text next to the diagram. Viewers that render untrusted Markdown turn off `click` links.
- Copy config keys from `references/mermaid-light.json` and `references/mermaid-dark.json` exactly. Mermaid ignores a misspelled key without an error, so a rendered diagram does not prove the config applied.

## 7. Check that it renders

When you can run `npx`, check every Mermaid block before you hand the result back:

```bash
npx -y --prefer-offline @mermaid-js/mermaid-cli@12.2.0 -i FILE.md -o "${TMPDIR:-/tmp}/diagramize/check.md"
```

- Pass the whole Markdown file. The tool checks every block, including blocks indented inside a list. For an HTML page, copy its Mermaid sources into a scratch `.md` file and check that.
- Read the exit status. Zero means every block parsed. Anything else comes with `Parse error on line N` and no output files.
- The tool drives a Chrome browser. If it reports that it cannot find one, set `PUPPETEER_EXECUTABLE_PATH` to an installed Chrome, on macOS `/Applications/Google Chrome.app/Contents/MacOS/Google Chrome`.
- If you cannot run the check, say that the diagrams are unchecked.

Each viewer draws Markdown diagrams with its own Mermaid version, so details can differ from the HTML page. To see which version a viewer runs, render a Mermaid block that holds only the word `info`.

## 8. Extend a diagram

When the caller extends a diagram drawn earlier in the conversation ("and on failure?", "where does the token come from?"), draw that same diagram again as a `diff` that adds the new part. Start a fresh diagram only when the new content needs a different kind.
