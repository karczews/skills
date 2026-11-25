---
name: code-simplifier
description: Simplify recently changed code for clarity and consistency without changing what it does
disable-model-invocation: true
---

# code-simplifier

Refine code so a reader follows it faster. Every edit is a pure refactor: outputs, side effects, errors and public signatures stay identical for every input, including empty, null and error cases.

## 1. Find the scope

- Text after the invocation (`/code-simplifier:code-simplifier`, `/code-simplifier`, `@code-simplifier`) names the scope: files, a directory, a commit range or a branch.
- Otherwise the scope is the code you changed in this session.
- With no changes in this session, it is the uncommitted changes: staged, unstaged and untracked files.
- With none of these, ask the user what to simplify, and stop.

When the scope is a set of changes, simplify the changed lines and the functions that hold them. Leave simplifications elsewhere for the report.

## 2. Hand off when you can

A subagent reads code you wrote with fresh eyes. When you can launch one, give it the scope, one sentence on what the change is for, and this file's path (or the text of steps 3-6 when you lack the path). Tell it to run steps 3-6 itself, skipping step 2, and to return its step 6 report.

When files in scope already have uncommitted changes, save a copy of each before the handoff, so you can tell the subagent's edits from the existing ones. When it returns, check its whole diff against the code, revert any edit that changes behavior, and pass the report on with your reverts added.

Without subagents, run steps 3-6 yourself.

## 3. Explore

- Read the project's agent instructions (`AGENTS.md`, `CLAUDE.md`, and any in the changed files' directories), its linter and formatter configs, and the code around each change. They set naming, imports, error handling and function style, and they win where step 4 disagrees.
- Search the modules around the scope and the project's shared utility code for helpers the code in scope re-implements.
- Run the project's tests, type checks and linters that cover the scope, and note which already fail.

## 4. Simplify

Check every function in scope against every rule:

- Flatten nesting with early returns and guard clauses.
- Remove branches that provably cannot run, duplicated logic, and variables that only rename.
- Replace re-implemented logic with a helper step 3 found.
- Remove speculative generality: options nobody sets, parameters every caller passes the same value.
- Name each variable and function for what it holds or does.
- Merge related logic that sits apart for no reason.
- Delete comments that restate the code; keep comments that explain why.
- Write a choice between three or more branches as an if/else chain or a switch; keep ternaries for a single two-way choice.
- Match the patterns the surrounding code already uses.

The measure is how fast a reader new to the file follows it, not line count. Keep an abstraction when inlining it would spread its complexity across its callers; inline it otherwise. Keep one concern per function and intermediate values a debugger can stop on. Spell out logic that a dense one-liner would hide.

Leave out, for the report, any simplification that would change a public signature, remove a defensive check, move code between modules, or swap in a helper that behaves differently on some input.

## 5. Verify

Rerun the checks from step 3. When one that passed there now fails, revert your edits in the failing file one at a time until it passes. Leave checks that already failed unfixed. Done when every check that passed in step 3 passes again.

## 6. Report

List each edit that changes how the code reads, one line each: `path:line`, what changed, why it reads better. Leave out formatting-only edits. Then list:

- the checks you ran, with their results before and after your edits;
- the parts of the scope no check covers;
- every simplification you left out, with the reason: outside the scope, or excluded by step 4.
