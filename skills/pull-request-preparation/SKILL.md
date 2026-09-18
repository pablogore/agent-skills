---
name: pull-request-preparation
description: Discovers a repository's actual contribution gates before opening a pull request. Use when opening, preparing, or fixing a PR. Use when a checklist tells you to apply a label, fill a template, or tick a check you have not confirmed exists. Use when working across multiple repositories that enforce different rules.
---

# Pull Request Preparation

## Overview

Templates, labels, approval gates, required checks, and branch rules are repository policy, not universal GitHub behavior. A PR workflow that worked in one repository is a hypothesis everywhere else.

Two failures follow from assuming instead of discovering:

- **Inventing gates.** You apply a label that does not exist, or perform ceremony against a workflow the repository never installed.
- **Claiming unearned checks.** You tick "tests pass" from a template you copied rather than a command you ran. A checklist filled without execution is worse than no checklist: it manufactures confidence.

Discover first, then write. When a gate is absent, report it as absent.

## When to Use

- Opening or preparing a pull request in any repository
- Returning to a repository you have not submitted to recently
- Following a PR checklist that came from a template, another project, or a skill
- A PR was rejected by a check you did not know existed

Skip when pushing directly to a branch with no PR, or when a maintainer has already specified the exact body and labels.

## Discovery

Run these before writing a single line of the PR body. Record what each returns.

```bash
REPO="$(gh repo view --json nameWithOwner --jq .nameWithOwner)"
BASE="$(gh repo view --json defaultBranchRef --jq .defaultBranchRef.name)"
gh repo view --json isFork,parent --jq '.isFork, .parent.nameWithOwner'
```

`gh` targets a fork's **upstream** by default. On a fork, pass `--repo "$REPO"` explicitly to every `gh pr` and `gh label` call, or you will open the PR against someone else's project.

| Question | Command | How to read the result |
|---|---|---|
| Is there a template? | `fd -H -t f 'PULL_REQUEST_TEMPLATE' .github .` | Found: fill its sections verbatim. Absent: write a plain body. |
| Which labels exist? | `gh label list --repo "$REPO" --limit 100` | Apply only what this returns. Nothing else is a label. |
| What is actually enforced? | `rg -l 'pull_request' .github/workflows` | Only jobs in these files are gates. No match means nothing is enforced. |
| What are the house rules? | `fd -H -t f -d 2 'CONTRIBUTING.md\|AGENTS.md\|CLAUDE.md'` | Repo guidance outranks any skill, including this one. |
| How do I verify? | `rg -n '^[a-z][a-z-]*:' Makefile`, or the CI workflow's own steps | Run the repo's commands, not invented ones. |

## Rules

- Apply only labels that `gh label list` returned. Never create a label to satisfy a checklist.
- Claim a check passed only after running it and reading its output.
- Never apply an approval label to your own PR or its linked issue. Report a missing approval and let a human grant it.
- Link the issue the work closes with `Closes #N` when one exists. When none exists, say so rather than inventing a reference.
- Never add `Co-Authored-By` or AI-attribution trailers to commits.
- When a prescribed gate does not exist in this repository, say so in your report instead of performing it.

## Writing the Body

When a template exists, fill it. When none exists:

```markdown
Closes #N

## Summary
<1-3 bullets: what changed and why>

## Changes
| File | Change |
|------|--------|

## Test plan
- [x] <command actually run> — <real result>
```

Every test-plan line names a command you ran and its real outcome. Delete lines you cannot back with output — an honest short plan beats a complete fictional one.

## See Also

- `git-workflow-and-versioning` — branching, atomic commits, and change sizing before the PR exists
- `code-review-and-quality` — splitting a PR that exceeds the review budget
- `ci-cd-and-automation` — what the workflows you discovered are actually doing

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "Every repo needs an approved issue linked." | That is one repo's CI rule. Most repositories enforce nothing of the kind. Check the workflows. |
| "I'll add the `type:bug` label like last time." | Labels are per-repository. `gh label list` takes one second and tells you the truth. |
| "The template says tests pass, so I'll tick it." | A ticked box you did not earn is a false report. Run the command or delete the line. |
| "There's no CONTRIBUTING.md, so there are no rules." | Rules live in workflows, templates, and AGENTS.md too. Absence of one file proves nothing. |
| "It's my own fork, so the base is obvious." | `gh` defaults to the upstream parent. Pass `--repo` or you will file against the original project. |
| "Discovery is overhead for a one-line change." | The commands take seconds. A PR rejected by an unknown gate costs a full round trip. |

## Red Flags

- A label in your PR that did not appear in `gh label list`
- A ticked checkbox with no command output behind it
- A PR body section copied from another repository's template
- `gh pr create` without `--repo` in a fork
- Performing an approval step on your own work
- Reporting "all checks pass" when no workflow triggers on `pull_request`
- A test plan naming commands that do not exist in this repository

## Verification

Before opening the PR:

- [ ] Ran the discovery commands and recorded repo, base branch, template, labels, and enforced checks
- [ ] Every label applied appeared in `gh label list` output
- [ ] Every ticked check corresponds to a command run, with its real result
- [ ] `--repo` is explicit if this repository is a fork
- [ ] Linked issue exists, or the PR states that none does
- [ ] No `Co-Authored-By` trailer in any commit
- [ ] Gates that do not exist here are reported as absent, not performed
