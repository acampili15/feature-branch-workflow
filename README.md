# feature-branch-workflow

A [Claude Code](https://claude.com/claude-code) skill that encodes a
disciplined, one-branch-per-change git/GitHub workflow: branch fresh from
the remote default branch, discuss design first for anything non-trivial,
verify with real tests before committing, write commit messages that
explain *why*, open a pull request with a Summary and a checkboxed Test
Plan, hand off exact pull/test/merge commands, and confirm merges via the
GitHub API rather than assuming.

It exists because of a pattern that comes up constantly in real
collaboration: the person using Claude Code often can't verify a change
themselves until it's in front of them — they're testing on different
hardware, a real device, production infrastructure, or just prefer
reviewing a diff before it lands. This skill treats every change as
"propose on a branch, let them look, then they merge," so that review step
stays real instead of theatrical.

## What it does

- Branches from `origin/<default-branch>`, never local state alone, since
  local state can be stale if the user merges things elsewhere in the
  meantime.
- Names branches by intent: `feature/<kebab-case>` or `fix/<kebab-case>`.
- Flags when to stop and discuss design before writing code — new API
  surface, a real tradeoff, genuine ambiguity — versus when it's fine to
  just build (a small, obvious fix).
- Verifies changes like someone who didn't write them would have to: the
  test suite, linters, and an actual end-to-end smoke test where
  feasible, not just "the unit tests passed."
- Writes commit messages that explain the *why*, not just the what, and
  keeps AI model identity out of everything pushed to the repo (commit
  messages, PR titles, PR bodies) — attribution trailers are left to
  whatever the current session's own environment specifies, not
  hardcoded by this skill.
- Opens PRs with a `## Summary` and a checkboxed `## Test Plan`, including
  at least one explicit item the user needs to verify themselves.
- Hands off the exact commands to pull, test, and later merge the branch
  — using the `origin/<branch>` form specifically, since a branch that
  was never locally checked out can't be merged by its bare name.
- Confirms a merge actually happened by checking the GitHub API, rather
  than inferring it from local git state.
- Does one thing at a time: doesn't start the next branch until the
  current one is confirmed merged (or told explicitly to move on).

## Install

Claude Code loads **personal skills** from `~/.claude/skills/<name>/SKILL.md`
— available to any session, in any repository, once installed.

```sh
git clone https://github.com/acampili15/feature-branch-workflow \
  ~/.claude/skills/feature-branch-workflow
```

(If you've already got something at that path, just copy `SKILL.md` in
instead of cloning over it.)

Claude Code picks it up automatically on the next session — no further
configuration needed. You can confirm it's loaded by asking Claude Code
to list its available skills.

## Usage

Nothing to invoke by name — it's written to trigger naturally whenever a
task is "implement something in a git repo, ship it, and have it
reviewed via a pull request," which covers most real feature/bug-fix work
without the user needing to say "use the feature-branch-workflow skill"
explicitly.

## License

MIT
