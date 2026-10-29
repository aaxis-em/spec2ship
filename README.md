# spec2ship

A [Claude Code](https://claude.com/claude-code) skill that takes a product
idea, or a spec file, all the way to a published, installable release in
one run.

Give it a one-line idea and Claude researches the problem, the market, the
competitors and the technical and legal feasibility, then either writes the
spec or tells you the idea is not feasible and why. Give it a `Spec.md` and
it starts building straight away. Either way it asks you a few questions
once (license, repository, whether to publish), then works through every
phase without stopping:

| Phase | What happens |
| --- | --- |
| 0. From idea to spec | Only for an idea: researches users, market, competitors, feasibility and legal limits into `docs/RESEARCH.md` with sources, gives a verdict (build / build narrower / do not build), and writes `Spec.md`, or stops if it should not be built |
| 1. Read and ask once | Lists milestones, dependencies, platforms and the definition of done; asks the open decisions in one go |
| 2. Verify before you build | Captures real payloads from the tools the project integrates with and commits them as fixtures |
| 3. Scaffold | Repo layout, `Makefile` with a `check` target, CI on Linux, macOS and Windows, docs skeleton, GitHub repo |
| 4. Build in milestones | Code, tests and docs per milestone; `make check` must pass before the next one; one commit each |
| 5. Try it for real | Installs the binary and runs the README quickstart as a new user would, in isolated data dirs |
| 6. Cross-platform | Gets CI green on all three OSes |
| 7. Audit | Dead code, heavy dependencies, trust boundaries, docs vs. code |
| 8. Release | `install.sh` / `install.ps1` with checksum checks, tagged release workflow, verifies the public install |
| 9. Launch kit | Drafts a tweet thread, Show HN post and subreddit posts (never posts them) |
| 10. Report | What was built, tested, decided, skipped, and what you still need to do by hand |

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

From an idea, in an empty directory:

```
/spec-to-ship a CLI that tracks how much time I spend in each git branch
```

From a spec, in the directory that holds it:

```
/spec-to-ship            # reads Spec.md
/spec-to-ship docs/my-spec.md
```

A "do not build" verdict is given when the core of the idea needs something
that does not exist or cannot be reached, would break a law or a service's
terms, or cannot be built and verified even at a smaller scope. A crowded
market alone does not stop the run; the verdict says so and you decide.

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
