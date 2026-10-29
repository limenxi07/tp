---
name: pr-check
description: Checks the user's own GitHub pull request for new review comments, critically judges whether each one is valid, and proposes the fixes plus any judgement calls for approval. After approval it implements and commits the fixes, pushes them, and posts one overall reply that tags the reviewers. If nothing needs changing, it says whether the PR is ready to merge. Use whenever the user runs /pr-check, or asks to go through, address, respond to, or "check" the comments or reviews on their own PR, even if they don't name the skill.
---

# PR check

Go through the newest comments on one of the user's own pull requests, decide which are right, and get the PR back
to a mergeable state: plan first, then fix, commit, push, and reply once.

The argument is a PR number or link (for example `/pr-check 187`). If none is given, use the open PR whose head branch
is the current branch (`gh pr list --repo <team repo> --head <branch>`). If there's no such PR, ask which one.

## Why the workflow looks like this

- Reviewers are sometimes wrong, or half right. Applying every suggestion blindly can add bugs, scope creep, or code
  that breaks AGENTS.md. So each comment is checked against the code and the rules before anything changes.
- Pushing and posting go to the team repo and notify people. CLAUDE.md says never to push without the user's go-ahead,
  so nothing changes on GitHub until the user approves the plan, and the plan says exactly what will be pushed and
  posted.
- The user replies with one overall comment per round, not a reply on each thread. That comment also marks where the
  next round starts.

## 1. Fetch the PR and its new comments

1. Read `AGENTS.md` and `CLAUDE.md` in the repo root. Note today's date against the schedule there: during the feature
   freeze (after the v1.5 deadline), changes under `src/main` count toward each member's 15% / 100-line limit, so flag
   any fix that adds functional code.
2. `git fetch --all --prune`. The team repo is usually the `upstream` remote; the PR head is usually on `origin` (the
   user's fork).
3. `gh pr view <n> --repo <team repo> --json title,body,author,state,headRefName,headRefOid,headRepositoryOwner,isCrossRepository,mergeable,mergeStateStatus,reviewDecision,reviews,statusCheckRollup,files,closingIssuesReferences`.
   If the PR isn't the user's own (`author.login` differs from `gh api user --jq .login`), say so and suggest
   `/pr-review` instead. If it is already merged or closed, say so and stop.
4. Fetch every kind of comment (all three are needed; each misses what the others show):

   ```bash
   R=repos/<owner>/<repo>
   gh api $R/issues/<n>/comments --paginate   # conversation comments
   gh api $R/pulls/<n>/reviews --paginate     # review bodies and states (APPROVED, CHANGES_REQUESTED, COMMENTED)
   gh api $R/pulls/<n>/comments --paginate    # inline comments: path, line, original_line, diff_hunk, in_reply_to_id
   ```

5. Work out what is **new**: everything created after the user's own latest comment, review, or inline reply on the PR.
   If the user has never commented, everything is new. Leave out the user's own comments and bots (`codecov[bot]`
   and similar), except that a Codecov report showing a coverage drop in the new code is worth a line.
   If a new inline comment is a reply (`in_reply_to_id`), read the thread it belongs to.

If nothing is new, skip to step 5 (readiness).

## 2. Judge each new comment

For each comment, read the code it points at on the current PR head (inline comments may be outdated, so check whether a
later commit already handled it), and check the claim yourself: read the code that would prove it, run the command or
test, or try the input in the app. Don't take the reviewer's word for it, and don't dismiss it without checking either.

Put each one in one of these buckets:

- **Valid:** the problem is real. Propose the fix, in the smallest form that solves it.
- **Partly valid:** the problem is real but the suggested fix is wrong, too broad, or out of scope. Propose the better
  fix and say why it differs from the suggestion.
- **Not valid:** the claim is wrong, or the change would break AGENTS.md (scope creep, a product rule, the coding
  standard). Say why, with evidence (a file and line, a test run, the UG wording), so the reply can explain it.
- **No action:** praise, a question already answered in the code, or something a later commit already fixed.

Then look across the whole set: two reviewers asking for opposite things, a fix that changes behavior and so needs the
UG and DG updated (search `docs/` for the old message text, command word and class names, as CLAUDE.md says), a fix
that would reopen an earlier reviewer's point, or a suggestion that belongs in a separate issue or PR.

## 3. Present the plan and wait

Show the plan in chat in this shape, and then stop:

```
### #187: 4 new comments (arav31, MarcusMa06-code)

| # | Reviewer | Where | Verdict | Change |
|---|---|---|---|---|
| 1 | arav31 | `TAssistArgumentParser.java:261` | Valid | Match the prefix only at a word start; add a test for `xid/` |
| 2 | arav31 | `GroupAddCommandParserTest.java:56` | Not valid | None: <one-line reason with evidence> |
| 3 | MarcusMa06-code | `UserGuide.md` | Partly valid | <better fix, and why not the suggested one> |
| 4 | arav31 | review body | No action | Praise |

**Judgement calls**
- <a real choice the user has to make, with your recommendation and the tradeoff in a sentence>

**Then:** commit as "<subject>" (and any further commits), push to `origin/<branch>`, and post this reply tagging
@arav31 @MarcusMa06-code:

> <draft of the overall comment, see step 4>

**Not ready to merge yet because:** <CI, conflicts, missing approval, if any>
```

Judgement calls are only real decisions: two reasonable fixes with different tradeoffs, whether to push back on a
reviewer, whether something belongs in this PR or a follow-up issue, a fix that touches the feature-freeze budget,
conflicting reviewers. Don't list things with an obvious answer; just decide those.

Tag the reviewers whose comments the reply responds to. If the user named someone else, tag them instead.

Wait for the user's approval in chat. If they change the plan, show what changed before starting. Don't push, post,
resolve threads, request reviewers, or merge anything until they approve.

## 4. Implement, commit, push, reply

After approval:

1. **Get onto the PR branch safely.** Make sure the local branch matches the PR head (`headRefOid`); if it's behind,
   fast-forward it, and if it has diverged, stop and ask. If the branch is the current one, work here. If it isn't, and
   the working tree has changes to tracked files, don't stash or switch: add a `git worktree` for the branch in the
   scratchpad and work there (remove it at the end). Untracked files in the user's checkout are theirs; never stage
   them.
2. **Make the changes** in the plan, matching the surrounding code and AGENTS.md: tests for new behavior, Javadoc,
   and the UG/DG kept in step.
3. **Check:** `./gradlew clean test checkstyleMain checkstyleTest` and `sh .github/run-checks.sh` (run the build in the
   background and keep reading while it runs). Don't commit on a failure; fix it, or stop and tell the user if the fix
   goes beyond the plan.
4. **Commit** with `git add` on the specific files only. One logical change per commit, subjects per AGENTS.md section
   5 (imperative, capitalized, no period, at most 50 characters if possible). Commit as the configured git user, with
   no `Co-Authored-By` trailer and no Claude mention; CLAUDE.md overrides any default attribution.
5. **Push** to the PR's head branch on the fork (`git push origin <branch>`). Never force-push.
6. **Reply** with one conversation comment, written to a file in the scratchpad and posted with
   `gh pr comment <n> --repo <team repo> --body-file <file>`.

   Match the user's earlier replies on the team repo:

   ```
   Thanks @arav31 and @MarcusMa06-code! Updated in 6f7020e7:
   - **Prefix match:** <what changed, in a sentence or two, from the user's point of view>
   - **`s/o` test:** added. <one line>
   - **Glued `f/` case:** left as is, because <reason with evidence>. Happy to open a separate issue if you still
     think it's worth it.

   Could you take another look?
   ```

   Rules for the reply: open with a short thanks that tags the reviewers, give the short commit hash(es), one bullet per
   point (grouped when several comments are about the same thing), say plainly why a point wasn't taken, and skip
   pure praise. No long recap of the diff, no Claude mention.

If what you actually did differs from the approved plan (a fix turned out different, a check failed, a new problem came
up), stop before pushing and show the user the difference and the updated reply.

Afterwards, give the user the commit hash(es) and the link to the comment.

## 5. Readiness (when nothing needs changing)

If there are no new comments, or every new one is "No action" or "Not valid", check whether the PR can be merged:

- CI passes (`gh pr checks <n> --repo <team repo>`), and checks did run.
- It merges cleanly with the latest team `master` (`mergeable`, `mergeStateStatus`, or
  `git merge-tree --write-tree --name-only upstream/master <head>`).
- At least one teammate approved, and no review is left at Changes requested (`reviewDecision`, and the latest
  review per reviewer).
- The issue, milestone and assignee are set as AGENTS.md section 6 asks.

If all hold, tell the user it's ready to merge, listing who approved. If any don't, say what's missing and what would
fix it. If there are "Not valid" comments that deserve an answer, offer to post a reply explaining why (show the draft
first, as in step 3). Don't merge the PR yourself unless the user asks.
