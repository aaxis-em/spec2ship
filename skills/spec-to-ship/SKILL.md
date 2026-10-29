---
name: spec-to-ship
description: Build a whole project end to end from a spec file - research, build in milestones, test against the real tools, cross-platform, docs, audit, release and launch kit - in one run. Use only when the user runs /spec-to-ship or asks to build and ship a project from a spec.
argument-hint: "[path to spec, default Spec.md]"
disable-model-invocation: true
---

# Spec to ship

Build the project described in `$ARGUMENTS` (default `Spec.md` in the current directory) and take it all the way to a published, installable release. Run every phase below in order without stopping, except at the two places that say **ask**.

The spec wins over this skill. Where the spec is silent, follow this skill.

## Ground rules (all phases)

- **Code style:** smallest complete change. Before writing, list every place the change must reach (callers, tests, fixtures, config, docs). Prefer what exists: the codebase, then the standard library, then an installed dependency. Never add a dependency for a few lines. No abstractions, options or "for later" code. Comment only the why that code cannot show. A known limit gets `// shortcut: <limit>, <when to upgrade>`.
- **Commits:** Chris Beams' seven rules. Subject ≤ 50 characters, imperative, capitalized, no period. Body wrapped at 72 characters, explaining what and why. End with the attribution trailer from the system reminder. One commit per milestone or fix, never a giant final commit.
- **Decisions:** every choice the spec leaves open goes into `docs/DECISIONS.md` as `## Dn. Title` with **Context**, **Choice**, **Why**. Write it when you decide, not at the end.
- **Honesty:** report only what you ran and saw. A failing test is reported with its output. A skipped step is named as skipped.
- **Others' work:** never commit, revert or stash uncommitted changes you did not make. Stage only your own hunks.
- **User data:** test with isolated data directories (`XDG_DATA_HOME`, `XDG_CONFIG_HOME`, a temp `HOME` when needed) and throwaway repos in the scratchpad. Never write into the user's real config or data to test.
- **Long runs:** run slow test suites in the background and wait for the notification. Do not poll with sleep.
- **Context:** keep `.ship/progress.md` (add `.ship/` to `.git/info/exclude`) with the phase, the milestone, what is done and what is next. Update it after each milestone, so the run resumes cleanly after context compaction.

## Phase 0 - Read and ask once

1. Read the whole spec. List the milestones, the external tools and formats it depends on, the target platforms, and its definition of done.
2. Check the environment: language toolchain, `git`, `gh` auth, the external tools the spec integrates with (and their versions), `wine` or other cross-platform test options.
3. **Ask** all decisions that belong to the user in one AskUserQuestion call (up to 4 questions), only for those the spec does not already answer:
   - license (Apache-2.0 / MIT / proprietary) and the copyright holder name;
   - GitHub repository: name, public or private, push as you go or only at the end;
   - publish a release at the end (yes / build only);
   - anything in the spec that is truly ambiguous and changes the design.

   The answers are the authorization for the push, release and publish steps later. Do not ask again for those.
4. Write `.ship/progress.md` with the milestone list.

## Phase 1 - Verify before you build

For every external format the project reads or writes (agent transcripts, hook payloads, APIs, file formats, CLI output):

1. Look at the real thing: run the installed tool, capture real payloads (for example a temporary debug hook that tees stdin to a file), read the tool's source or schema. Docs alone are not enough. Real tools differ from their docs, and new versions add modes (for example Codex "code mode" wraps commands in an `exec` script).
2. Scrub the samples (paths, usernames, code, secrets) and commit them as test fixtures.
3. Record the observed schema, with the tool version, in `docs/ARCHITECTURE.md`.
4. Parsers tolerate unknown fields and entry types. They log, not fail.

## Phase 2 - Scaffold

Create the repository layout from the spec, with:

- a `Makefile` (or the stack's equivalent) with `build`, `test`, `race`, `lint`, `docs-lint` and one `check` target that runs them all;
- CI on Linux, macOS and Windows that runs `check`;
- `.gitattributes` with `* text=auto eol=lf`, so Windows CI does not fail on CRLF;
- `docs/` skeleton: README, USAGE, ARCHITECTURE, DECISIONS, GLOSSARY, CONTRIBUTING, plus what the spec requires;
- a docs linter if the spec sets a writing style.

`git init`, first commit, create the GitHub repository with the visibility from Phase 0 (`gh repo create`), push.

## Phase 3 - Build in milestones

For each milestone in the spec's order:

1. Build it, with tests: unit tests per package, table-driven positive and negative cases, golden tests for parsers and reports, and end-to-end tests that drive the real binary in a temp repo.
2. Update the docs for what the milestone adds. Do not leave docs to the end.
3. Run `make check`. Do not start the next milestone with a failing check.
4. Commit and push. Update `.ship/progress.md`.

A bug fix always gets a test that fails without the fix. Verify that by reverting the fix once.

## Phase 4 - Try it for real

Tests prove the code matches your understanding. This phase proves your understanding matches reality.

1. Build the binary and install it into a scratch `bin` directory.
2. In a throwaway repo with isolated data directories, run the real integrations end to end, for example a real agent session with the hooks installed, a real `curl | sh` install.
3. Use the tool as a new user would: the quickstart from the README, step by step, with no knowledge you have from building it.
4. Every surprise is a bug, a doc fix or a decision. Fix it, add the regression test, commit.
5. Check the setup states a user can get stuck in (hooks installed but not active, tool not restarted, wrong scope, imported vs live data). The tool's `doctor` or status output must name each one and say what to do.

## Phase 5 - Cross-platform

1. Make CI green on all three operating systems. Read failing CI logs (`gh run view --log-failed`) and fix the cause, not the test.
2. Watch for: path separators and drive letters, `.exe` suffixes, CRLF, file locking and deletion of open files on Windows, shell quoting in PowerShell, the user PATH in the Windows registry (read and write the raw `REG_EXPAND_SZ` value, never the expanded one).
3. If `wine` is available, run the Windows test binaries locally for a faster loop.

## Phase 6 - Audit

Before the first release, audit the whole repository in this order and fix what you find:

1. Dead code (for Go: `go run golang.org/x/tools/cmd/deadcode@latest ./...`) and duplicated helpers.
2. Heavy dependencies: for each large one, check whether a small embedded file or the standard library does the job. Measure binary size and dependency count before and after.
3. Correctness at trust boundaries: input validation, error paths that could lose data, secrets in logs, lock files left behind.
4. Docs match the code: every command, flag and config key in the docs exists, and the reverse.

Commit the fixes in small commits.

## Phase 7 - Release

1. Installers: `install.sh` (Linux, macOS) and `install.ps1` (Windows). They detect OS and architecture, download the release archive, verify the SHA-256 checksum, install to a user directory, and print the next step. Add installer tests that run in CI against a local server that serves a fake release.
2. A release workflow on tag `v*`: run the tests, build archives for each OS and architecture, publish them with `checksums.txt`. Include LICENSE, NOTICE and README in each archive.
3. LICENSE, NOTICE and the README license section name the copyright holder and the year from Phase 0.
4. Tag only a commit whose CI is green. Push the tag and wait for the release workflow.
5. Verify the published release: run the public one-line install command in a scratch directory and run `<tool> version`.
6. Never move or re-push a published tag. A fix goes into the next version.
7. Install the released binary for the user only if they asked.

Skip this phase if the user chose "build only" in Phase 0.

## Phase 8 - Launch kit

Write `.ship/launch.md` (not committed):

- a launch tweet under 280 characters plus a 4-post thread, with at most 2 hashtags at the end; every claim must be true of the release;
- a "Show HN" title and first comment;
- 2 to 3 subreddits that fit, each with a one-paragraph post;
- the steps to set the GitHub social preview image (Settings → Social preview) if the repository has a logo;
- an idea for a 15-second demo recording of the main feature.

Do not post anything. The user posts.

## Phase 9 - Report

End with a short report to the user:

- what was built, and the release link and the install command;
- what was tested for real (Phase 4), and on which platforms CI passed;
- the decisions made without asking (link `docs/DECISIONS.md`);
- what was skipped or not verified, and known risks;
- what the user must do by hand (for example approve hooks in a tool, set the social preview, post the launch).

Delete `.ship/progress.md` only after the report.
