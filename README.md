# spec2ship

A [Claude Code](https://claude.com/claude-code) skill that builds a whole
project from a spec file and takes it all the way to a published,
installable release in one run.

You write `Spec.md`. Claude reads it, asks you a few questions once
(license, repository, whether to publish), then works through every phase
without stopping:

| Phase | What happens |
| --- | --- |
| 0. Read and ask once | Lists milestones, dependencies, platforms and the definition of done; asks the open decisions in one go |
| 1. Verify before you build | Captures real payloads from the tools the project integrates with and commits them as fixtures |
| 2. Scaffold | Repo layout, `Makefile` with a `check` target, CI on Linux, macOS and Windows, docs skeleton, GitHub repo |
| 3. Build in milestones | Code, tests and docs per milestone; `make check` must pass before the next one; one commit each |
| 4. Try it for real | Installs the binary and runs the README quickstart as a new user would, in isolated data dirs |
| 5. Cross-platform | Gets CI green on all three OSes |
| 6. Audit | Dead code, heavy dependencies, trust boundaries, docs vs. code |
| 7. Release | `install.sh` / `install.ps1` with checksum checks, tagged release workflow, verifies the public install |
| 8. Launch kit | Drafts a tweet thread, Show HN post and subreddit posts (never posts them) |
| 9. Report | What was built, tested, decided, skipped, and what you still need to do by hand |

Decisions the spec leaves open are recorded in `docs/DECISIONS.md` as they
are made, and progress is tracked in `.ship/progress.md` so long runs
survive context compaction.

## Install

As a plugin, from inside Claude Code:

```
/plugin marketplace add aaxis-em/spec2ship
/plugin install spec-to-ship@spec2ship
```

Or copy the skill by hand:

```sh
git clone https://github.com/aaxis-em/spec2ship
mkdir -p ~/.claude/skills
cp -r spec2ship/skills/spec-to-ship ~/.claude/skills/
```

## Use

In the directory that holds your spec:

```
/spec-to-ship            # reads Spec.md
/spec-to-ship docs/my-spec.md
```

The skill only runs when you invoke it; Claude will not start it on its own.

The spec always wins over the skill. Where the spec is silent, the skill's
defaults apply: small changes, standard library before new dependencies,
commits that follow Chris Beams' seven rules, and only reporting what was
actually run.

## Requirements

- Claude Code
- `git`, and the [GitHub CLI](https://cli.github.com/) (`gh`) logged in, for
  creating the repository and the release
- The toolchain for whatever language your spec uses
- Optional: `wine`, for a faster Windows test loop on Linux

## License

MIT. See [LICENSE](LICENSE).
