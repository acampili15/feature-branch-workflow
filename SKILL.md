---
name: feature-branch-workflow
description: One-branch-per-change git/GitHub workflow — branch fresh from the remote default branch, discuss design first for anything non-trivial, verify with real tests before committing, write an explanatory commit message, open a PR with a Summary and a checkboxed Test Plan, hand the user exact pull/test/merge commands, and confirm merges via the GitHub API rather than assuming. Use this whenever starting feature or bug-fix work in a git repo that's headed toward a pull request — not just when the user explicitly says "open a PR," but any time the plan is to implement something, ship it, and have someone else (or the user, on their own machine) review and merge it. Especially relevant when the user tests on different hardware than this session runs on, or merges through their own terminal rather than letting the session merge directly.
---

# Feature branch workflow

A disciplined one-change-per-branch workflow for shipping work through pull
requests, rather than committing straight to the default branch or batching
several unrelated changes into one branch. It exists because of a pattern
that shows up constantly in real collaboration: the person using this
session often can't verify a change themselves until it's in front of them
— they're testing on a different OS, a real device, production
infrastructure, or just prefer reviewing a diff before it lands. Treating
every change as "propose on a branch, let them look, then they merge" keeps
that review step real instead of theatrical.

## 1. One change, one branch

Before writing any code, make sure the branch is actually fresh:

```sh
git fetch origin <default-branch>
git checkout -b <new-branch-name> origin/<default-branch>
```

Branch from the **remote** tracking ref, not local `main`/`master` — if the
user merges other work through their own terminal while this session is
running, local state can be behind without anything saying so. Always
check which branch actually is default (`main`, `master`, `develop`, etc.)
rather than assuming.

Name branches descriptively and by intent:
- `feature/<kebab-case-description>` — new functionality
- `fix/<kebab-case-description>` — bug fixes

One logical change per branch. If something else needs fixing along the
way (a real bug noticed mid-task, a flaky test), consider whether it's
small enough to fold in or deserves its own branch — don't let scope creep
turn one PR into three unrelated ones.

## 2. Discuss design before building, when it matters

Not every change needs a conversation first — a one-line bug fix or an
obvious, requested tweak can just get built. But stop and talk it through
*before* writing code when:

- the change adds new API surface (a new endpoint, a new config key, a new
  file format) — these are expensive to walk back once something depends
  on them
- there's a real design choice with tradeoffs (storage shape, UI pattern,
  what counts as "done") rather than one obviously correct approach
- the request is ambiguous enough that two reasonable implementations
  would look very different

When discussing, propose options with a clear recommendation rather than
an open-ended "what do you think" — give the user something concrete to
react to or redirect. Move to implementation once they've actually
confirmed a direction, not once they've merely acknowledged the question.

## 3. Implement, then actually verify it

Before committing, verify the change like someone who didn't write it
would have to:

- run the existing test suite
- run relevant linters/syntax checks
- where feasible, do an end-to-end smoke test of the real behavior, not
  just unit tests — actually run the script/server/function being
  changed against real or seeded input and look at the real output

"The tests pass" and "I watched this actually do the right thing" are
different claims — prefer being able to make the second one, especially
for anything touching I/O, timing, or external state.

## 4. Write a commit message that explains why

A good commit message here earns its length — it's often the only
record of *why* a change exists, read later by someone (or some future
session) with none of this conversation's context. Include:

- what changed, briefly
- **why** — the reasoning, the bug it fixes, the tradeoff it resolves
- what was actually verified (which tests, which manual check)

Leave out anything that would date or misattribute the commit:
- no AI model name or version anywhere in the commit message, PR title,
  or PR body — keep model identity out of everything pushed to the repo
- attribution trailers (`Co-Authored-By:`, session links, etc.) are
  dictated by whatever this session's own environment/system instructions
  say to append, not by this skill — follow those verbatim and don't
  invent or hardcode a different one

## 5. Push and open the PR

```sh
git push -u origin <branch-name>
```

Open a pull request with this shape:

```markdown
## Summary

- What changed, as a short bullet list
- Why, where it's not obvious from the bullet itself

## Test Plan

- [x] Ran the full test suite — <result>
- [x] <any other verification already done, checked off>
- [ ] <something the user specifically needs to verify themselves,
      with the exact command(s) to run>
```

The unchecked item matters — it's an explicit handoff, not filler. If
everything was already fully verified, there may be nothing left unchecked,
but that should be true because verification was thorough, not because the
checklist was written loosely.

Check for an existing PR template in the repo
(`.github/pull_request_template.md` or similar) and follow its structure
if one exists, filling it in rather than ignoring it.

## 6. Hand off exact commands, then wait

Give the user the literal commands to pull and try the branch:

```sh
git fetch origin
git checkout <branch-name>
```

— plus whatever manual steps actually matter for this change (what to run,
what to click, what result to expect).

Once they're satisfied, give the merge commands too — using the
`origin/<branch-name>` form specifically, since a branch that was never
locally checked out can't be merged by its bare name:

```sh
git fetch origin
git checkout <default-branch>
git pull origin <default-branch>
git merge origin/<branch-name>
git push origin <default-branch>
```

Don't merge it from this session unless the user has clearly said that's
how they want it handled — in this workflow, the merge is usually the
user's own action, often on hardware or a setup this session can't fully
reproduce or verify itself.

## 7. Confirm, don't assume

When asked to confirm something merged, check the actual PR state via the
GitHub API/MCP tool (look for `"merged": true`) rather than inferring it
from local git log or branch state — a PR can look merged locally from a
stale fetch, or not show as merged yet even after the user says "done" if
they haven't pushed the final step.

## 8. One thing at a time

Don't start the next branch until the current one is confirmed merged (or
the user explicitly says to move on to something else in parallel). This
keeps review manageable — one diff to look at, one thing to test, one
clear merge — rather than several half-finished branches competing for
attention.
