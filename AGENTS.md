# AGENTS.md

Instructions for AI coding agents (Claude Code, Codex, Copilot, Gemini, and others) working in this repository.
Human team members should read it too.

This is the team project (tP) of CS2103T, AY2026/27 Semester 1, team AY2627S1-CS2103T-T12-4. The product is
**TAssist**, a desktop app for CS2040S teaching assistants to track tutorial attendance, class participation, and
assignment submissions. It is built by evolving the AddressBook-Level3 (AB3) codebase.

The course sets the rules below, and some of them are graded. When this file conflicts with an agent's defaults, follow
this file. When it is silent, follow the surrounding code. Course site:
<https://nus-cs2103-ay2627-s1.github.io/website/>

## 1. Non-negotiable rules

1. Code must follow the SE-EDU Java coding standard, basic and intermediate rules (section 4).
2. Git commits must follow the SE-EDU Git conventions (section 5):
   <https://se-education.org/guides/conventions/git.html>
3. Never work on or send a PR from `master`. One task is one issue, one branch, and one PR (section 6).
4. Never break the product constraints (section 3).
5. Do not add a feature, change behavior, or restructure code that the user did not ask for. The course penalizes
   features that do not fit the product and team members who go "rogue".

## 2. Project facts

- Language and tooling: Java 25 (no other version), Gradle (`./gradlew`), JavaFX, JUnit 5, Checkstyle 14.1.0, JaCoCo,
  Codecov. Everything runs on Windows, Linux, and macOS.
- Source layout: `src/main/java/seedu/address/{commons,logic,model,storage,ui}`, tests under `src/test/java` mirroring
  the main packages, test helpers in `testutil`. The package name `seedu.address` is from AB3. Renaming packages is
  optional and should be done by one person, so do not rename without the team agreeing.
- Docs: `docs/` is a Jekyll site (kramdown), published at <https://ay2627s1-cs2103t-t12-4.github.io/tp/>. Diagrams are
  PlantUML sources in `docs/diagrams`, rendered images in `docs/images`.
- Repos: the team repo is `AY2627S1-CS2103T-T12-4/tp`. Each member works from their own fork and sends PRs to the team
  repo's `master`.
- Data: stored locally as a human-editable JSON file (see the constraints below).

### Useful commands

```
./gradlew clean test                    # run all tests
./gradlew checkstyleMain checkstyleTest # coding standard check
./gradlew check coverage                # what CI runs (plus .github/run-checks.sh)
./gradlew shadowJar                     # build the JAR (build/libs/tassist.jar)
sh .github/run-checks.sh                # EOF newline, LF line endings, trailing whitespace
```

CI (`.github/workflows/gradle.yml`) runs the repository checks, `./gradlew check coverage`, and uploads coverage to
Codecov. Keep CI passing. Extended CI failures can cost marks, and a Checkstyle violation costs code quality marks.

### Schedule (milestones are weekly iterations)

The default deadline is Thursday 23:59 of the iteration's week, unless the iteration says otherwise. Choose the
milestone from today's date. From v1.3, the "iteration on time" item on the course dashboard stays red if the
milestone is not wrapped up by the deadline.

| Milestone | Week | Deadline | Focus |
|---|---|---|---|
| v1.1 | 7 | Thu 1 Oct 2026, 23:59 | Workflow practice, README, site-wide settings, DG requirements |
| v1.2 | 8 | Thu 8 Oct, 23:59 | First functionality increment; each member merges at least one PR |
| v1.3 | 9 | Thu 15 Oct, 23:59 | MVP, GitHub release with JAR and release notes |
| v1.4 | 10 | Thu 22 Oct, 23:59 | Alpha: a version of every planned feature; release |
| v1.5 | 11 | Thu 29 Oct, 23:59 | Release candidate; alpha testing, UG and DG up to date; release |
| v1.6 | 12 | Tue 3 Nov, 14:00 | Final deliverables; practical exam Fri 6 Nov 12:00 to Sat 7 Nov 12:00 |

Each week's task list is at `https://nus-cs2103-ay2627-s1.github.io/website/admin/tp-w<N>.html` (for example `tp-w8`).
Check it before planning work.

**Feature freeze:** it starts at the v1.5 deadline (Thu 29 Oct 23:59). During v1.6, no more than 15% or 100 lines
(whichever is higher) of the functional code attributed to a member may change. The limit covers all changes under
`src/main`, including blank lines and comments, and does not cover tests, docs, or non-code files. Penalty: at least
-2 per member. Plan features before then.

## 3. tP constraints (must not be violated)

Violating these counts as a feature flaw or bug during peer testing.

- **Brownfield:** evolve the AB3 codebase in small steps, each leaving a working product.
- **Incremental:** deliver breadth-first with a steady rate. Do not leave the project to a last-minute burst.
- **Typing-preferred and CLI-first:** the target user types fast. Input is a text command; the GUI mainly gives
  feedback. Offer a CLI alternative for anything done by mouse. Prefer short, easy-to-type command formats, with a
  one-shot form for any multi-step command.
- **Single user:** no shared computer, no data file shared between users, no login or accounts.
- **Human-editable file:** store data locally as a human-editable text file, with at least AB3's level of support for
  manual edits (valid edits work; a wrong edit may lose data). Encrypting the file is not recommended.
- **No DBMS:** do not use a database system to store data.
- **Object-oriented:** follow the OO paradigm primarily.
- **Platform independent:** works on Windows, Linux, and macOS. Avoid OS-specific libraries and features.
- **Java 25:** must work on a machine with Java 25 and no other Java version.
- **Portable:** no installer. **Single file:** package everything as one JAR (or one ZIP if impossible).
- **File size:** the JAR/ZIP is at most 100 MB. Do not bloat it (oversized images, unnecessary libraries, JavaFX
  WebView). Do not use WebView just to show the User Guide.
- **No remote server:** the product must not depend on a server the team runs.
- **External software:** third-party libraries must be free, open-source, permissively licensed, need no user
  installation, and need prior approval from the teaching team (post the request in the forum before using one).
  Once approved for one team, any team may use it.
- **Screen resolution:** the GUI works well at 1920x1080 and above with scales 100% and 125%, and stays usable at
  1280x720 and above and at scale 150%. Be wary of auto-sizing windows.
- **Avoid drastic changes to `build.gradle`.**
- **Recommended:** keep network use minimal (add a fallback), avoid hard-to-test features (remote APIs, audio,
  accounts), and keep the usage scenario realistic.

TAssist product facts (see the Developer Guide appendix for the full requirements): the target user is a CS2040S TA
working alone on a laptop with about 20 students per tutorial group. The MVP covers help, tutorial groups, students,
weekly attendance, weekly participation, assignments, and automatic local persistence. Do not add features outside the
agreed scope without the team's consent.

### Product behavior that peer testers file bugs for

- A user mistake (a missing space, a wrong prefix, a repeated parameter) must never crash the app, corrupt data, or make
  it unusable. Failed commands must not change data.
- Error messages must be specific: they say what is wrong and how to fix it. Do not call an invalid value a format
  error, or the other way round.
- Do not validate too strictly. Warn instead of blocking when a deviation might be intentional, unless accepting it
  would hinder operation. Restrict lengths only to reasonable limits.
- Case sensitivity follows the real world: names, group names, and search keywords are not case-sensitive.
  Treat near-duplicates (different case, extra spaces) as duplicates, or warn.
- Do not make command formats harder to type than necessary: avoid long keywords, hard-to-type symbols, and
  unnecessary case sensitivity.
- Very long input must not break the layout in a way that hinders the user.
- Behavior must match the User Guide. If they differ, fix whichever is wrong.
- Keep terminal output presentable. Do not print misleading or alarming messages there.

## 4. Java coding standard (required)

Follow the SE-EDU Java coding standard, **basic and intermediate** rules:
<https://se-education.org/guides/conventions/java/intermediate.html>. The advanced rules are optional. For anything the
standard does not cover, use the [Google Java style guide](https://google.github.io/styleguide/javaguide.html) and the
surrounding code. The Checkstyle config in `config/checkstyle/checkstyle.xml` enforces part of this; the rest is checked
by hand and counts toward the code quality grade.

**Naming**
- Packages: all lowercase. Classes and enums: nouns in `PascalCase`. Variables: `camelCase`. Constants:
  `SCREAMING_SNAKE_CASE`, with a shared prefix for related constants (`COLOR_RED`, `COLOR_GREEN`).
- Methods: verbs in `camelCase`.
- Test methods may use underscores in the form `featureUnderTest_testScenario_expectedBehavior()`. Parts two and
  three may be dropped (`sortList_emptyList()`, `sortList()`).
- Boolean variables and methods must sound like booleans: `isFound`, `hasData`, `canEvaluate()`, `shouldAbort`.
  Boolean setters look like `void setFound(boolean isFound)`.
- Collections have plural names (`points`, `values`).
- Abbreviations are not all-uppercase inside names: `exportHtmlSource()`, not `exportHTMLSource()`.
- Names are in English. Use long names for large scopes and short names only for small scopes.
  Loop indices may be `i`, `j`, `k`, with `j` and `k` for nested loops only.

**Layout**
- 4 spaces, no tabs. Wrapped lines are indented 8 spaces more than the parent line.
- Lines: at most 120 characters, aim for under 110.
- When wrapping, break after a comma and before an operator (including `.`, `&`, `|`). Keep a method name attached to
  its opening parenthesis. Prefer higher-level breaks.
- K&R (Egyptian) brackets: the opening brace on the same line.
- Always use braces for `if`, `else`, `for`, `while`, and `do`, even for one statement. Put the condition's body
  on its own line (never `if (x) doIt();`).
- Spaces around operators, after commas and keywords (`while (`), and after `;` in `for`.
- One blank line between logical units inside a block.
- A `switch` needs a `default`, and a `case` without `break` needs a `// Fallthrough` comment.

**Statements**
- Every class is in a package. Imports are explicit (no `import java.util.*;`) and in a consistent order: static
  imports, then `java.*` and `javax.*`, then `org.*`, then other third-party packages such as `com.*` and
  `javafx.*` (the SE-EDU example, enforced by Checkstyle's `CustomImportOrder`).
- Array brackets go with the type (`int[] a`).
- Declare a variable where it is first needed, in the smallest scope, and initialize it there.
- Fields are not `public`, except constants and pure data classes.

**Comments**
- English, American spelling, no local slang.
- Every class and every public method has a Javadoc header, except getters, setters, overriding methods whose parent
  Javadoc applies exactly, and test code.
- Javadoc form: `/**` on its own line, a short first sentence that starts with a verb in the third person
  (`Returns ...`, `Adds ...`, `Sends ...`, never `Return ...` or `Returning ...`), a blank line before the tags,
  punctuation after each tag description, and no blank line between the comment and the declaration.
  Give `@param` for all parameters or none; `@return` may be dropped when obvious.
- One-line field Javadoc is fine: `/** Number of connections to this database. */`.
- Indent comments with the code they describe.

### Code quality (graded individually)

- Show evidence of **logging**, **exceptions**, **assertions**, and **defensive coding**. Log with `LogsCenter`
  (`java.util.logging`) at the right level: `SEVERE` for critical problems, `WARNING` for continuing with caution,
  `INFO` for noteworthy actions, `FINE` for debugging detail.
  Enable assertions (`-ea`) in your IDE run configurations, as the course asks.
- Apply SLAP (Single Level of Abstraction Principle): no long methods, no deeply nested code. Extract helper methods.
- No noticeable code duplication, especially in test code. Extract shared parts.
- Follow the code quality guidelines taught in the course. Keep the existing architecture (`Ui`, `Logic`, `Model`,
  `Storage`, and the command and parser pattern) unless the team agrees to change it.
- Prefer small, focused classes and methods. Do not add dead code, commented-out code, or debug output.

## 5. Git conventions (required)

Source: <https://se-education.org/guides/conventions/git.html>

**Commit subject (mandatory)**
- Imperative mood: `Add README.md`, not `Added` or `Adding`.
- Capitalize the first letter. No period at the end.
- At most 50 characters if you can, never more than 72.
- An optional `scope:` prefix is allowed: `Find command: make matching case-insensitive`, `chore: Update release date`.

**Commit body (optional, but if you write one, follow these)**
- Leave a blank line after the subject. Wrap lines at 72 characters.
- Explain WHAT changed and WHY, not HOW (the diff shows how). Use present tense for the current situation, imperative
  for the change, and avoid "currently" and "originally".
- Bullet points are fine. If the message needs to be very long, split the commit.

**Commits**
- One logical change per commit. Keep commits small and the product working after each one.
- Never commit generated files, local logs, build output, or secrets.

**Branches**
- Kebab-case with meaningful keywords: `refactor-ui-tests`. Related to an issue: `issueNumber-keywords`, such as
  `1234-ui-freeze-error`.
- Never use `master` for a PR. Do not reuse a branch for an unrelated task.

**Authorship**
- Each commit's author is the team member responsible for it. Commit with that member's configured identity
  (`git config user.name` and `user.email`). Do not override it.
- Do not add `Co-Authored-By` trailers to commits or PR descriptions.

Example:

```
Add attendance mark command

The TA cannot record whether a student attended a tutorial.

Let's add a command that marks a student present or absent for a given
week, replacing any earlier mark for that week.
```

## 6. Team workflow

Applies to every change, including docs and small fixes.

1. **Issue first.** Create an issue for the task (label `type.Task`, or `type.Story`, `type.Bug`, and so on, as
   appropriate), assign it to the person doing it, and set the **current milestone** (section 2). One issue per person
   per task: do not assign one issue to several members, split it instead.
2. **Branch.** Create a new branch from the latest team `master`, in your own fork (or in the team repo once the team
   agrees). Name it as in section 5.
3. **Small PRs.** Prefer small PRs that each move the product forward and leave it working. Start with a very simple
   version of a feature, and improve it in follow-up PRs.
4. **Open a PR to the team repo's `master`.** Title in the same style as a commit subject. Description explains what and
   why and links the issue with `Fixes #<issue>`. Assign the PR to its author and the same milestone. A PR is one
   change: never mix unrelated work (for example, a photo and an About Us edit).
5. **Review.** Get at least one teammate to review the PR before it is merged. Give real review comments, not a quick
   approve, and review others' PRs whenever you can.
6. **Merge, then close.** Merge (resolving conflicts on the PR branch), confirm the issue is closed, and sync your fork.
7. **Parallel PRs** from the same author are normal, and every member must have at least one during the tP.
8. **Wrap up the milestone** at the end of an iteration: move unfinished issues and PRs to the next milestone and close
   the milestone. Only v1.3 onward need a GitHub release (with JAR and detailed release notes).

Other rules:
- Update code, tests, and docs together in the same PR whenever practical. Early iterations may skip new tests or docs
  but should keep existing tests and CI passing.
- Do not force-push to shared branches or rewrite history that others have pulled.
- Every member should have commits in at least four of weeks 7 to 12 and do a fair share of team tasks. Each member
  should contribute to code, tests, UG, and DG.
- Do not delete failing tests to get CI green. Disabling tests temporarily is acceptable only while code is still very
  volatile, and they must be re-enabled. If Codecov reports a coverage drop in code that is hard to test, say so in the
  PR (an admin may merge it); otherwise add tests.
- Keep one `docs/DeveloperGuide.md`. Do not split it into multiple files.
- Keep the scope small. Do only the requested task: do not reformat unrelated files, rename things, upgrade
  dependencies, or edit `build.gradle` unless asked.

## 7. Documentation

- User Guide (`docs/UserGuide.md`): written for the target user, with clear command formats, examples, and screenshots.
  It must match the real behavior; remove or label ("Coming soon") features not yet implemented.
- Developer Guide (`docs/DeveloperGuide.md`): architecture, component design, implementation, requirements (target user,
  value proposition, user stories, use cases, NFRs, glossary), manual testing, Acknowledgements, and optionally
  "Appendix: Planned Enhancements". UML must use the notation taught in the course, with PlantUML sources in
  `docs/diagrams`. Keep diagrams simple and high-level, and use short code snippets only.
- When a change alters behavior, update every place in the DG that describes it, not just the section nearest the
  change. The same behavior is often described in the implementation notes, the use cases, the manual-testing steps,
  and the diagrams. Search `docs/` for the old message text, command word, and class names, and fix every match.
- Check docs for broken links, typos, messy formatting, and inconsistency with the product. These are all graded as
  documentation bugs.
- Docs style: markdown compatible with Jekyll/kramdown. The SE-EDU Markdown style guide and the Google developer
  documentation style guide are optional.
- Keep to the existing writing style so the UG and DG read as one cohesive document.

## 8. Testing

- Write automated tests for new behavior (unit, integration, and hybrid, as in `docs/Testing.md`). There is no minimum
  coverage, but weak tests risk undetected bugs, which cost marks in the practical exam.
- Mirror the main package structure under `src/test/java`, reuse `testutil` builders, and avoid duplicated test code.
- Run `./gradlew clean test checkstyleMain checkstyleTest` before pushing.
- Test the real JAR before releases: put it in an empty writable folder and run it with `java -jar "name.jar"` on
  Java 25. Check invalid, missing, duplicated, and unexpected parameters, long inputs, and different resolutions.

## 9. Before you open a PR

- [ ] There is an issue, and the PR links it with `Fixes #n`; both are assigned and set to the current milestone.
- [ ] The PR comes from a new kebab-case branch, not `master`, and is a single focused change.
- [ ] Commit subjects follow section 5 and are authored by the responsible member, with no `Co-Authored-By` trailer.
- [ ] `./gradlew clean test checkstyleMain checkstyleTest` passes and `sh .github/run-checks.sh` is clean.
- [ ] Javadoc exists on new classes and public methods, and names, braces, imports, and line length follow section 4.
- [ ] Docs are updated where behavior changed (UG, DG, diagrams), including every DG section that describes it
      (implementation, use cases, manual testing), found by searching `docs/` for the old wording (section 7).
- [ ] A teammate has been asked to review.

## 10. Reference links

- Standards page: <https://nus-cs2103-ay2627-s1.github.io/website/admin/standardsAndConventions.html>
- tP constraints: <https://nus-cs2103-ay2627-s1.github.io/website/admin/tp-constraints.html>
- tP expectations and grading: `tp-expectations.html`, `tp-grading.html`, `tp-deliverables.html` under the same admin path
- Reuse policy: `appendixB-policies.html` under the same admin path
- SE-EDU Java: <https://se-education.org/guides/conventions/java/intermediate.html> and
  <https://se-education.org/guides/conventions/java/logging.html>
- SE-EDU Git: <https://se-education.org/guides/conventions/git.html>
