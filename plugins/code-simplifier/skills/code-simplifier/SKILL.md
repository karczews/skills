---
name: code-simplifier
description: Simplify recently changed code for clarity and consistency without changing what it does
disable-model-invocation: true
---

# code-simplifier

Refine code so a reader follows it faster. Every edit is a pure refactor: outputs, side effects, errors and public signatures stay identical for every input, including empty, null and error cases.

## 1. Find the scope

- Text after the invocation (`/code-simplifier:code-simplifier`, `/code-simplifier`, `@code-simplifier`) names the scope: files, a directory, a commit range or a branch.
- Otherwise, the code you changed in this session.
- Otherwise, the uncommitted changes: staged, unstaged and untracked files.
- Otherwise, ask the user what to simplify, and stop.

For a set of changes, simplify the changed lines and the functions that hold them; leave the rest for the report.

## 2. Hand off when you can

When you can launch a subagent, hand it the pass for fresh eyes: give it the scope, one sentence on what the change is for, and this file's path (or the text of steps 3-6), and have it run steps 3-6 and return its step 6 report. First save a copy of each file in scope that has uncommitted changes, so you can tell its edits apart. When it returns, check its whole diff, revert any edit that changes behavior, and pass the report on with your reverts added.

Without subagents, run steps 3-6 yourself.

## 3. Explore

- Read the project's agent instructions (`AGENTS.md`, `CLAUDE.md`, and any in the changed files' directories), its linter and formatter configs, and the code around each change. They win over step 4 on naming, imports, error handling and function style.
- Search nearby modules and shared utility code for helpers the scope re-implements.
- Run the tests, type checks and linters that cover the scope; note which already fail.

## 4. Simplify

Check every function in scope against every rule:

- Flatten nesting with early returns and guard clauses.
- Remove branches that provably cannot run, duplicated logic, and variables that only rename.
- Replace re-implemented logic with a helper step 3 found.
- Remove speculative generality: options nobody sets, parameters every caller passes the same value.
- Name each variable and function for what it holds or does.
- Merge related logic that sits apart for no reason.
- Delete comments that restate the code; keep those that explain why.
- Use an if/else chain or a switch for three or more branches; keep ternaries for one two-way choice.
- Match the surrounding code's patterns.

The measure is how fast a reader new to the file follows it, not line count. Inline an abstraction unless that would spread its complexity across its callers. Keep one concern per function and intermediate values a debugger can stop on; spell out logic a dense one-liner would hide.

Leave for the report any simplification that would change a public signature, remove a defensive check, move code between modules, or swap in a helper that behaves differently on some input.

## 5. Verify

Rerun the step 3 checks. When one that passed now fails, revert your edits in that file one at a time until it passes. Leave checks that already failed unfixed. Done when every check that passed in step 3 passes again.

## 6. Report

List each edit beyond formatting on one line: `path:line`, what changed, why it reads better. Then list:

- the checks you ran, with their results before and after your edits;
- the parts of the scope no check covers;
- every simplification you left out, with the reason: outside the scope, or excluded by step 4.
