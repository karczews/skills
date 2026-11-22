---
name: code-simplifier
description: Simplify recently changed code for clarity and consistency without changing what it does
disable-model-invocation: true
---

# code-simplifier

Refine code so a reader follows it faster. Every edit is a pure refactor: outputs, side effects, errors and public signatures stay identical.

## 1. Find the scope

- Text after the invocation (`/code-simplifier:code-simplifier`, `/code-simplifier`, `@code-simplifier`) names the scope: files, a directory, a commit range or a branch.
- Otherwise the scope is the code you changed in this session.
- With no changes in this session, it is the uncommitted changes: staged, unstaged and untracked files.
- With none of these, ask the user what to simplify, and stop.

When the scope is a set of changes, simplify the changed lines and the functions that hold them. Note simplifications elsewhere for the report.

## 2. Hand off when you can

A subagent reads code you wrote with fresh eyes. When you can launch one, give it the scope, one sentence on what the change is for, and this file's path (or the text of steps 3-6 when you lack the path). Tell it to run steps 3-6 itself, skipping step 2, and to return its step 6 report. Check each edit in that report against the code, revert any that changes behavior, and pass the report on with your reverts added.

Without subagents, run steps 3-6 yourself.

## 3. Read the standards

Read the project's agent instructions (`AGENTS.md`, `CLAUDE.md`, and any in the changed files' directories), its linter and formatter configs, and the code around each change. They set naming, imports, error handling and function style, and they win where step 4 disagrees.

## 4. Simplify

Apply every rule to every function in scope:

- Flatten nesting with early returns and guard clauses.
- Remove dead branches, duplicated logic, variables that only rename, and abstractions with a single trivial use.
- Name each variable and function for what it holds or does.
- Merge related logic that sits apart for no reason.
- Delete comments that restate the code; keep comments that explain why.
- Write a choice between three or more branches as an if/else chain or a switch; keep ternaries for a single two-way choice.
- Match the patterns the surrounding code already uses.

The measure is how fast a reader new to the file follows it, not line count. Keep abstractions that name a concept or are reused, one concern per function, and intermediate values a debugger can stop on. Spell out logic that a dense one-liner would hide.

## 5. Verify

Run the project's tests, type checks and linters that cover the scope. When a check fails, revert your edits in the failing file one at a time until it passes. When it still fails with all of them reverted, it was already failing: report it. Done when every check you ran passes or is reported.

## 6. Report

List each edit that changes how the code reads, one line each: `path:line`, what changed, why it reads better. Then the checks you ran with their results, and the simplifications you left outside the scope. Leave out formatting-only edits.
