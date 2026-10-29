---
name: pr-review
description: Reviews a teammate's GitHub pull request against the repo's AGENTS.md, drafts concise, friendly review comments in chat for approval, then posts them as a Comment review with inline line comments. Use whenever the user runs /pr-review, asks to review a PR or several PRs by number or link, or wants feedback on a teammate's PR, even if they don't name the skill.
---

# PR review

Review one or more pull requests the way the user likes: checked against the repo's rules, drafted here first, and
posted only after approval as a **Comment** review with inline comments.

The arguments are the PR numbers or links (for example `/pr-review 128 129`). If none are given, ask which PR.

## Why the workflow looks like this

- A submitted GitHub review can't be deleted, only dismissed, and dismissal leaves a mark on the timeline. So nothing
  goes to GitHub until the user has seen the exact text and said to post it.
- The user prefers inline comments: each point sits on the line it is about, so the author can fix and resolve them one
  by one. One big Request changes body is harder to act on.
- PR branches are often behind `master`. A PR can pass on its own branch and still break once merged, so test the merged
  result, not just the branch.

## 1. Read the rules and fetch

1. Read `AGENTS.md` (and `CLAUDE.md`) in the repo root, and review against them: coding standard, Git conventions,
   product constraints, the team workflow, and the "Before you open a PR" checklist. If there's no `AGENTS.md`, use the
   surrounding code's conventions.
2. Fetch the latest changes from every remote (`git fetch --all --prune`). Work out the team repo (usually the
   `upstream` remote) and its default branch.
3. Fetch each PR into temporary refs without touching the user's branch or working tree, for example
   `git fetch upstream master:refs/review/master pull/<n>/head:refs/review/pr<n>`.

## 2. Review each PR

Collect the facts first:

- `gh pr view <n> --json title,author,body,headRefName,milestone,assignees,labels,commits,files,closingIssuesReferences,reviews,statusCheckRollup`
  and the linked issue (`gh issue view`). Check the issue, milestone, assignee and labels against AGENTS.md, and whether
  `Fixes #n` would close an issue the PR only partly covers.
- The diff against the merge base, plus enough of the surrounding files to judge it.
- Whether the branch is behind `master`, and whether it merges cleanly (`git merge-tree --write-tree --name-only`).
- CI status (`gh pr checks <n>`). Say so if no checks ran.

Then run the checks yourself, in a temporary `git worktree` in the scratchpad so the user's checkout stays untouched:

- On the PR branch: the test and style commands from AGENTS.md (for the tP:
  `./gradlew clean test checkstyleMain checkstyleTest` and `sh .github/run-checks.sh`).
- On the PR merged onto the latest `master`, if that merge is clean. If tests fail only there, that's a finding: name
  the failing test and why it fails.
- Run long builds in the background, and read the code while they run.

What to look for, roughly in this order of importance:

1. Correctness bugs and behavior that doesn't match the UG/DG, or features of the codebase the PR bypasses (for example
   an existing confirmation, validation or error-message mechanism that the new command should also use).
2. Product rules from AGENTS.md: specific error messages, no crashes or data changes on a failed command, sensible
   validation, case-insensitivity, the docs kept in step with behavior.
3. Code quality: duplication (including test code), SLAP, Javadoc, naming, logging, assertions.
4. Workflow: issue, milestone, labels, commit messages, scope creep, merge conflicts, CI.

Before reporting something, check that it's real: read the code that would prove it, or run it. Drop anything you can't
confirm, or label it as a question.

Clean up afterwards: remove the temporary worktrees (`git worktree remove --force`) and refs
(`git update-ref -d refs/review/...`).

## 3. Draft here and wait for approval

Show the drafts in chat, one section per PR, in this shape:

```
### #128: Comment review

**Summary:** <one or two friendly sentences, plus any point that has no changed line to sit on>

- **`path/to/File.java:170`**: <comment>
- **`path/to/Other.java:73–82`**: <comment>
```

Writing style:

- Concise and friendly. Open with something genuinely good about the PR, then the points. Phrase requests as
  suggestions or questions ("Could we…?", "Maybe…").
- One point per comment, saying what is wrong and what would fix it. Mark small things as "Nit:" or "Minor:".
- Only points that matter. Don't pad with praise or restate what the diff does.
- Anything that can't sit on a changed line goes in the summary: unchanged files (such as a stale UG section), merge
  conflicts, the PR description, CI.

Every inline comment needs a line that is inside the PR's diff hunks on the new side. Get line numbers from
`gh api repos/<owner>/<repo>/pulls/<n>/files --paginate` (the `patch` field): count lines from each `@@ +start` header,
skipping `-` lines. For a range, use the last line as `line` and the first as `start_line`, both in the same hunk.

Then stop and ask whether to post. Don't post, approve, request changes, dismiss or edit anything on GitHub until the
user says so in chat. If they change the drafts, show the new version before posting.

## 4. Post

After approval, post each PR as a single review with `event: COMMENT` (not `REQUEST_CHANGES` or `APPROVE`, unless the
user asks for one). Write the payload to a JSON file in the scratchpad and post it:

```bash
gh api repos/<owner>/<repo>/pulls/<n>/reviews --method POST --input review<n>.json
```

```json
{
  "event": "COMMENT",
  "body": "<summary>",
  "comments": [
    {"path": "src/test/java/seedu/address/MainAppTest.java", "line": 170, "side": "RIGHT", "body": "<comment>"},
    {"path": "src/test/java/.../JsonSerializableTAssistTest.java", "start_line": 73, "start_side": "RIGHT",
     "line": 82, "side": "RIGHT", "body": "<comment>"}
  ]
}
```

If GitHub rejects a line (422), fix that anchor or move the point into the summary, and show the user the change before
posting again. Afterwards, confirm with `gh api repos/<owner>/<repo>/pulls/<n>/reviews` and give the user the link to
each review.
