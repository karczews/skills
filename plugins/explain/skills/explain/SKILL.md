---
name: explain
description: Explain how code works or what a change did, with a short answer and a diagram traced from the source, in the terminal, a Markdown file, or on an HTML page
disable-model-invocation: true
---

# explain

Answer with one or two sentences and visuals sized to the question: one by default, up to three when the question has separate parts (a before/after pair counts as one). Names come from code you read, not from memory. The `diagramize` skill draws every visual: load it before step 3. If unavailable, flag it and stop.

## 1. Find the topic

- Text after the invocation (`/explain:explain`, `/explain`, `@explain`) is the topic.
- With no text, the topic is the question currently open in the conversation.
- With neither, ask the user what to explain, and stop.
- Strip at most one destination suffix (`in html`, `in terminal`) and at most one save suffix (`to <file>.md`, `to <file>.html`, `to docs`) in either order. Match case-insensitively at the very end only; the rest is the topic. Never strip mid-sentence matches.
- `to docs` and `to <file>.md` request a Markdown save (step 7); `to <file>.html` requests an HTML save at that path. `in html` / `in terminal` request a place (step 4). Slug: lowercase topic, non-alphanumerics to `-`, max 8 tokens.

## 2. Read before you draw

When the topic involves code, open the files it touches and trace the path you are about to draw. When that path spans many files and subagents exist, delegate the trace, then verify cited lines yourself.

For a change, read the diff (both versions), its announcement (commit/PR/issue), covering tests, and callers of changed names.

What goes into the visual: symbols and paths spelled as in code, with caller annotations kept as `diagramize` §1 defines them (`path:line`, `(?)`). Names that do not exist yet only as `+` lines in a diff.

## 3. Pick the visual

- For the exact code to write, the visual is the full code block, only when a diff's missing lines would hide where it goes.
- Otherwise tell `diagramize` what the question asks; it picks the kind from its table and draws it for the destination.
- For a change that does not ask for exact code, the visual is `diagramize`'s before and after in the same kind, marked per its §3 conventions. When the change spans modules, pair it with a file tree diff or name the omitted parts in one note line.

## 4. Ask where to show it

Once you know what the answer holds and before you write it, ask terminal vs HTML (§5 vs §6) and mark one recommended:

- HTML page, when it needs more than three parts, overflows `diagramize`'s terminal limits (20 lines, 80 columns), or is an HTML-only kind (UI, dense graph).
- Terminal, otherwise (handles one to three visuals that fit).

Skip the question and use that place instead when the topic named it (`in html`, `in terminal`, `to <file>.md`, `to <file>.html`, `to docs`), when it is a follow-up (reuse the previous place unless renamed), or when the harness cannot ask (use the recommended choice). Ask via harness question tool, or plain question; if non-interactive use recommended.

## 5. Write the reply (terminal)

The reply is, in order:

1. The answer in 1-2 sentences (before → after for a change). Steps and parts go in the visual, never here.
2. One visual (up to three for separate parts, each with a one-line lead). Explain the asked part and name what was left out when too large for one visual.
3. At most three notice lines (for a change: tests covered, what stays unverified, open decision).

### Prose

These cover the answer, notes, saved files and HTML page. Visuals keep compact notation.

- Name the mechanism and the actor: which function calls which, what it returns, what fails and how. Write "the loader parses the file", not "the file is parsed".
- Call each thing by its code name, and keep using it. Prefer plain, measured words with numbers over adverbs and metaphors.
- Mark unverified paths with `(?)` instead of hedging the sentence. Hedge once, only where unsure.
- Start with the answer and end after the last note. No greeting, praise, summary line or offer of more help.

## 6. Follow-ups, save, HTML page

- Follow-up: when the new question extends the previous visual, have `diagramize` extend it per its §8, for the same place.
- Save: write to the named file, or to `<cwd>/docs/explain/<topic-slug>.md` for `to docs` (relative to the working directory, never into the skills install). The file holds answer, notes and Markdown visuals; reply with the two-sentence answer and the absolute path.
- HTML page: have `diagramize` build it from its `references/page-template.html`, handing it the answer and notes to place around the visuals. Always pass the explicit path `${TMPDIR:-/tmp}/explain/<topic-slug>.html` (or the `to <file>.html` path); `diagramize` writes there, opens it and hands the path back. Reply with the two-sentence answer and the path.
