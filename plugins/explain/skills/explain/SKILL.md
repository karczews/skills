---
name: explain
description: Explain a topic with a short answer and a diagram traced from the code, in the terminal or on an HTML page
disable-model-invocation: true
---

# explain

Answer with at most two sentences and one visual sized to the question, in the terminal or on an HTML page. Names in a visual come from code you read, not from memory.

## 1. Find the topic

- Text after the invocation (`/explain retry logic`, `@explain retry logic`) is the topic.
- With no text, the topic is the question currently open in the conversation.
- With neither, ask the user what to explain, and stop.
- Text that ends in `to <file>.md` or `to docs` is a request to save the answer; the rest is the topic. See step 7.
- Text that ends in `in html` or `in terminal` says where to show the answer; the rest is the topic. See step 4.

## 2. Read before you draw

When the topic involves code, open the files it touches and trace the path you are about to draw. When that path spans many files and you can start a subagent, give it the search: ask for the entry point and the path through the code with `path:line` for each step. Then open the lines you will cite and check them before drawing.

In the visual:

- Spell symbols and paths exactly as the code does.
- Add `path:line` to the entry point and to the nodes that answer the question.
- Mark a step you inferred but did not trace (dynamic dispatch, config lookup, reflection, generated code, a network hop) with `(?)` and say in one line why it is unverified.
- Names that do not exist yet appear only as `+` lines in a diff.

## 3. Pick the format

| The question is…                                     | Show                                 |
|------------------------------------------------------|--------------------------------------|
| what calls what at runtime                           | call tree                            |
| what the logic decides, step by step                 | pseudocode                           |
| who talks to whom, in what order                     | sequence diagram in text             |
| what states something can be in, and what moves it   | state diagram in text                |
| where something lives, which module owns what        | shallow file tree with comments      |
| how pieces compose (UI components, classes, modules) | structure tree                       |
| what the data looks like                             | one real example, annotated          |
| which option to pick                                 | comparison table                     |
| what changes, or what would change                   | before and after (see Shapes)        |
| the exact code to write                              | full code block                      |
| how a UI looks, or a graph too dense for text        | one HTML page (step 8)               |

## 4. Ask where to show it

Once you know what the answer holds, and before you write it, ask the user whether to show it in the terminal or on an HTML page. Offer the two as choices, with the harness's tool for asking the user a question when it has one, and mark one as recommended:

- HTML page, when the explanation is big: it needs more than one visual, a visual longer than 20 lines or wider than 80 columns, or the format in the last row of the table.
- Terminal, otherwise.

Skip the question in these cases:

- The topic named the place (`in html`, `in terminal`), or asked to save to a file (step 7). Use that.
- The question is a follow-up (step 6). Use the place the previous reply went to, unless the user names another.
- The harness cannot ask the user. Use the recommended choice.

For the terminal, write the reply as in step 5. For HTML, build the page as in step 8.

## 5. Write the reply

The reply is, in order:

1. The answer, in one or two sentences. They say what happens or where it lives; the visual shows how. Steps, formulas and lists of parts go in the visual, never in these sentences. For a change, one sentence says how it worked before and one says how it works after.
2. The visual. One by default; up to three when the question has separate parts, each with a one-line lead. A before and after pair counts as one visual.
3. At most three lines on what to notice, or the open decision.

Visuals stay within 20 lines and 80 columns. Replace omitted branches with `…` and name what was left out.

The reply is read in a terminal (Claude Code, OpenCode), where Mermaid shows as source. Draw with indentation, box-drawing glyphs and arrows. Mermaid goes only into files that render it (step 7) and the HTML page (step 8).

### Prose

These rules cover the answer, the notes, saved files and the HTML page. Visuals keep their compact notation.

- Name the mechanism: which function calls which, what value it returns, what fails and how. A sentence that could appear unchanged in another project's docs says nothing about this one. Cut it.
- Name the actor. Write "the loader parses the file", not "the file is parsed". Use the passive only when nothing in the code does the action.
- Put one idea in each sentence. Write whole sentences with articles and verbs. Arrows, symbols and abbreviations belong in the visual.
- Call each thing by its name in the code, and keep using that name.
- Prefer plain words: use, help, many, if, is, has.
- Prefer concrete nouns to metaphors. Avoid surface, vector, primitive, paradigm, substrate, scaffolding, flywheel.
- Replace an adverb with a number or a stronger verb: "cuts startup from 2 s to 300 ms", not "significantly improves startup".
- Hedge once, and only where you are unsure. Mark unverified code paths with `(?)` instead of softening the sentence.
- List as many items as there are. Groups of three, "not just X but Y" and trailing "-ing" clauses ("…, ensuring reliability") pad the point.
- Punctuate with periods and commas. No em dashes. Use a colon only before a list or an example. No bold labels and no emoji.
- Start with the answer and end after the last note. No greeting, praise, summary line or offer of more help.

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
- Two visuals of the same format, led by `Before` and `After`, when the change reorders steps or swaps actors, so a diff would be mostly `-` and `+` lines. In a tree or pseudocode pair, prefix lines in `After` with `+` for added and `~` for changed. Removed lines appear only in `Before`. The pair stays within 20 lines.
- A table with `Before` and `After` columns and one row per behaviour that changes, when the change is about what happens rather than where the code lives. For a fix, mark the broken behaviour with `✗` and what replaces it with `✓`.

Diff, keeping the shape the user already knows:

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

Write the code out in full when the change is mostly new, or when a diff's missing lines would leave the reader unsure where it goes.

## 6. Follow-ups

When your previous reply in this conversation drew a visual and the new question extends it ("and on failure?", "where does the token come from?"), draw that same visual again as a `diff` that adds the new part. Start a fresh visual only when the new question needs a different format.

## 7. Save to a file

When the topic asked to save the answer (step 1), write it to the named file, or to `docs/explain/<topic-slug>.md` for `to docs`. The file holds the same answer, visuals and notes, with each text sequence, state or flow diagram rewritten as a Mermaid block. Trees, pseudocode, tables and diffs stay as they are. Reply with the two-sentence answer and the file path.

## 8. HTML page

When the answer goes to an HTML page (step 4).

- The page holds the same answer, visuals and notes as a terminal reply. The 20-line and 80-column limits do not apply, so draw the whole path instead of cutting branches to `…`. Draw sequence, state and flow diagrams in Mermaid.
- One self-contained file with inline CSS and JS. For Mermaid, load `https://cdn.jsdelivr.net/npm/mermaid@12.2.0/dist/mermaid.min.js`.
- Draw from the real code: actual labels, values and states. In a UI project, take colours, fonts and component names from its theme or design system. Check that it fits a narrow window. For a UI change, show the before and after screens side by side, stacked on a narrow window.
- Write it outside the repository, to `${TMPDIR:-/tmp}/explain/<topic-slug>.html`.
- Open it with `open` on macOS or `xdg-open` on Linux. If your harness has a tool that shows HTML to the user, use that instead.
- Reply with the two-sentence answer and the file path.
