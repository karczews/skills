---
name: explain
description: Explain a topic with a short answer and a diagram traced from the code, in the terminal or on an HTML page
disable-model-invocation: true
---

# explain

Answer with at most two sentences and one visual sized to the question, in the terminal or on an HTML page. Names in a visual come from code you read, not from memory. The `diagramize` skill draws every diagram: load it before step 3. It ships in the same plugin as this skill. If it is not available, tell the user to install it next to this skill, and stop.

## 1. Find the topic

- Text after the invocation (`/explain retry logic`, `@explain retry logic`) is the topic.
- With no text, the topic is the question currently open in the conversation.
- With neither, ask the user what to explain, and stop.
- Text that ends in `to <file>.md` or `to docs` is a request to save the answer; the rest is the topic. See step 7.
- Text that ends in `in html` or `in terminal` says where to show the answer; the rest is the topic. See step 4.

## 2. Read before you draw

When the topic involves code, open the files it touches and trace the path you are about to draw. When that path spans many files and you can start a subagent, give it the search: ask for the entry point and the path through the code with `path:line` for each step. Then open the lines you will cite and check them before drawing.

What goes into the visual:

- Symbols and paths spelled exactly as the code does.
- `path:line` on the entry point and on the nodes that answer the question.
- `(?)` on a step you inferred but did not trace (dynamic dispatch, config lookup, reflection, generated code, a network hop), with one line on why it is unverified.
- Names that do not exist yet only as `+` lines in a diff.

## 3. Pick the visual

- For the exact code to write, the visual is the full code block. Write the code out in full when the change is mostly new, or when a diff's missing lines would leave the reader unsure where it goes.
- For everything else, find the row of `diagramize`'s "Pick the kind" table that matches what the question asks. `diagramize` draws that row's kind for wherever the answer goes. When the Terminal shape would not fit the terminal limits, step 4 recommends the HTML page.

## 4. Ask where to show it

Once you know what the answer holds, and before you write it, ask the user whether to show it in the terminal or on an HTML page. Offer the two as choices, with the harness's tool for asking the user a question when it has one, and mark one as recommended:

- HTML page, when the explanation is big: it needs more than one visual, a visual that does not fit the limits in `diagramize`'s "Terminal" section, or a kind that only an HTML page can show.
- Terminal, otherwise.

Skip the question in these cases:

- The topic named the place (`in html`, `in terminal`), or asked to save to a file (step 7). Use that.
- The question is a follow-up (step 6). Use the place the previous reply went to, unless the user names another.
- The harness cannot ask the user. Use the recommended choice.

For the terminal, write the reply as in step 5. For HTML, build the page as in step 8.

## 5. Write the reply

The reply is, in order:

1. The answer, in one or two sentences. They say what happens or where it lives; the visual shows how. Steps, formulas and lists of parts go in the visual, never in these sentences. For a change, one sentence says how it worked before and one says how it works after.
2. The visual, drawn by `diagramize` for the terminal. One by default; up to three when the question has separate parts, each with a one-line lead. A before and after pair counts as one visual.
3. At most three lines on what to notice, or the open decision.

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

## 6. Follow-ups

When your previous reply in this conversation drew a visual and the new question extends it ("and on failure?", "where does the token come from?"), have `diagramize` extend that visual, as its "Extend a diagram" section says.

## 7. Save to a file

When the topic asked to save the answer (step 1), write it to the named file, or to `docs/explain/<topic-slug>.md` for `to docs`. The file holds the same answer and notes, with the visuals drawn by `diagramize` for a Markdown file from the same rows of its table, so a sequence or state diagram becomes Mermaid. Reply with the two-sentence answer and the file path.

## 8. HTML page

When the answer goes to an HTML page (step 4), have `diagramize` build the page for the visuals, and hand it the answer and the notes to place around them. The page holds the whole explanation. Write it to `${TMPDIR:-/tmp}/explain/<topic-slug>.html`. Reply with the two-sentence answer and the file path.
