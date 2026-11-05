---
name: red-green-audit
description: Audit whether tests really test what they claim, via hand-made mutation testing ("red-green audit"). For each test, state its claim, make the smallest production-code change that breaks exactly that claim, run only that test, expect red tied to the claim, restore, confirm green. Runs with one sequential subagent in a tree of the user's choosing, writes an incremental Markdown report, reports progress, and only then asks whether to strengthen flagged tests. Project-agnostic; works for a few tests or a whole suite in batches. Use when the user says "red-green audit", "mutation audit", "mutation test these", "do these tests really test anything", "audit test quality", "are these tests vacuous", "would this test catch a bug", or wants to verify a test suite before trusting it. Keywords: red-green, mutation testing, test quality, vacuous tests, weak assertions, test audit.
---

# Red-Green Audit

A test is only worth something if it goes red when the behavior it claims to protect breaks. This skill checks that by hand: one test, one targeted mutation, one run, restore, confirm. It is not a coverage tool and not an automated mutation framework — it is a deliberate, readable audit whose output is a report a human can act on.

Verdicts are evidence, not opinions. Every row in the report must say what was changed, where, and what the runner printed.

## When to use

- Before trusting a suite that was written quickly, generated, or ported.
- When a bug shipped despite "having tests" and you want to know which tests are vacuous.
- Reviewing a PR whose tests look suspiciously green-only.
- Periodic hygiene on a directory or feature area.

Do not use it as a substitute for writing tests; it audits existing ones.

## Intake (ask before anything runs)

Ask these in one message, with sensible defaults offered. Do not start until the user confirms the resolved scope.

1. **Scope** — in their own words: a file, a directory, a name filter (`-k`, `-t`, `-run`), a feature area, "tests touching this diff", or "everything, in batches". Resolve it to concrete test ids (collect-only / `--list-tests` / `go test -list`), show the list or the count per file, and confirm. For large scopes propose batching by directory and note the audit is resumable from the report.
2. **Test command** — the exact per-test invocation and how to select a single test, e.g. `pytest path::test_name`, `npx jest path -t "name"`, `go test ./pkg -run '^TestName$'`, `cargo test name -- --exact`. Also how to run the baseline for the scope.
3. **Where to run** — the current tree, or an isolated copy the user already has or wants (worktree, clone, container; whichever tool they prefer). The skill never creates one itself. Also ask whether the env needs setup steps (deps, build, db, containers, env files).
4. **Per-test timeout** — default 120s unless the suite is known to be slower.
5. **Report location** — must be untracked. Default: the session scratchpad, or a gitignored dir the user names. Never inside tracked paths.
6. **Linter autofix** — does the project run a formatter/linter with autofix on save or in the test command? If yes, it must never run in fixing mode while a mutation is live.

## Baseline (once)

In the chosen tree, after any env setup the user listed:

1. Record the baseline: `git status --porcelain`, `git diff`, and a checksum list of every production file the scope could touch (`shasum -a 256` / `sha256sum`). Save these next to the report as `baseline/`.
2. Run the baseline test command for the scope. Any test that fails, times out, or needs an unavailable service is recorded as SKIPPED with the reason and excluded from mutation. The audit does not stop for them.

The tree may contain uncommitted work — the checksum compare at the end is what proves nothing was lost.

## Subagent brief

One subagent runs all mutations strictly sequentially. It may compact its context; the report file is its durable memory, so it must re-read the report on resume and continue from the first test without a verdict. If the subagent cannot write files, it returns the rows as text after each batch and the main session appends them to the report — state which mode is in effect at kickoff.

Reusable prompt template (fill the angle-bracket fields):

```
You are running a red-green audit. Work ONLY in <tree path>. Run tests strictly one at a time, in order.

Report file: <report path> (append rows as you go; re-read it first and resume after the last row with a verdict).
Baseline checksums: <baseline dir>.
Test command for a single test: <command template>. Per-test timeout: <N>s (wrap with `timeout <N>` or the runner's own flag).
Linter autofix: <never run / not applicable>. Never run `<lint cmd>` without its no-fix flag while a mutation is live.

Tests to audit (in this order):
<list of test ids>

Per test:
1. Read the test. Write its CLAIM in one sentence ("X returns Y when Z", "handler does not call send when the flag is off").
2. Pick the smallest production-code change that breaks exactly that claim: flip a comparison, drop a condition or a call, change a constant, swap an argument, return early. Prefer a one-line change. Note file:line and the before/after.
3. Before editing: copy the file to <backup dir>/<same relative path>, and write <tree path>/CURRENT_MUTATION.txt with `file=<path>` and `backup=<backup path>`.
4. Apply the mutation. Run ONLY that test with the timeout.
5. Classify:
   - Assertion failure clearly tied to the claim -> VALID.
   - Red only for some parametrized cases, or red only via a crash/internal assert rather than the test's own assertion -> PARTIAL.
   - Still green -> WEAK. Record the surviving mutation precisely.
   - ImportError / syntax error / unrelated crash -> INVALID-MUTATION; pick a different mutation and retry once; if still invalid, record it.
   - Runner cannot execute it -> BLOCKED with reason.
6. Restore by copying the backup back. Verify the checksum matches the baseline. Delete CURRENT_MUTATION.txt. Run the test again and confirm green. If it is not green, STOP and report; do not continue.
7. For "nothing happened" assertions (assert_not_called, empty result, no exception), ALSO mutate so the thing DOES happen (add the call, return an item, raise) and run again; a negative assertion that cannot fail is a vacuous negative.
8. Append the row to the report immediately.

After every <batch size> tests (or every file), append a progress line to the report and return a short progress message: done/total, counts per verdict, current file.

Forbidden during the audit: git checkout, git restore, git stash, git reset, git clean, git add, git commit, git push. Restore only via the backup copy. Never touch files outside the tree. Never edit test files during the audit phase. Never commit.

If you find CURRENT_MUTATION.txt on start, a previous run crashed mid-mutation: restore from the backup it names, verify checksum, delete the marker, then resume.

At the end: run `git status --porcelain` and `git diff --stat`, recompute the checksums, and write the comparison against the baseline into the report's "Baseline integrity" section. Return the summary.
```

## Per-test loop (reference)

```
claim      -> one sentence, taken from the test name + assertions, not from what the code happens to do
mutate     -> smallest change that falsifies the claim; one file, ideally one line
run        -> only this test, with timeout
observe    -> assertion failure tied to the claim? which assertion? which params?
restore    -> copy backup back, checksum, re-run, green
negatives  -> for assert_not_called / empty / no-raise, also mutate to make it happen
record     -> row appended
```

Verdicts:

| Verdict | Meaning |
| --- | --- |
| VALID | Goes red with an assertion failure that matches the claim; green after restore. |
| PARTIAL | Red only for some parametrized cases, or only through a crash / internal assert rather than the test's own assertion. |
| WEAK | Stays green under a mutation that breaks the claim. The test does not protect what it says. |
| INVALID-MUTATION | The change produced an import/syntax/unrelated error, so it proves nothing about the test. Retry once with a different mutation. |
| BLOCKED | Cannot be run in this environment (missing service, platform, credentials). |
| SKIPPED | Failing, flaky, or timing out at baseline, before any mutation. |

Patterns to look for beyond the verdict (record them in a separate section):

- **Redundant guards** — two conditions where removing either alone keeps the test green. Note which pair.
- **Fixtures too uniform** — one rule, one key, all tokens the same length, all ids equal: mutations that swap or reorder cannot be observed. Suggest the varied fixture.
- **Vacuous negatives** — `assert_not_called` / "result is empty" on something that could never happen in that setup. Suggest splitting so each assertion can independently fail.
- **Equivalent mutants** — mutations that do not change observable behavior (dead branch, redundant normalization). Note them; do not force a test to exist for them.
- **Over-mocked** — the mutated code is never executed because the mock replaces it. Note it; the verdict is WEAK with reason "mocked out".

## Progress reporting

The subagent returns a progress message after each batch (default: every file, or every 10 tests, whichever is smaller). The main session relays it to the user verbatim in one or two lines:

```
red-green audit: 34/120 done | VALID 25, PARTIAL 3, WEAK 4, INVALID 1, BLOCKED 0, SKIPPED 1 | now: tests/billing/test_invoices.py
```

The same line is appended to the report so a resumed run knows where it was.

## Report format

File: `<report dir>/red-green-audit-<scope>-<YYYY-MM-DD>.md`, where `<scope>` is a short slug of what was audited (`billing`, `pr-1234`, `all-batch-3`). Created at kickoff with the header and empty sections; rows appended as they are produced. Rerunning on the same scope the same day appends to the existing file after a `## Resumed <time>` line rather than overwriting.

```markdown
# Red-green audit: <scope> — <date>

Tree: <path>   Commit: <sha>
Test command: <template>   Timeout: <N>s
Baseline: <n passed>, <n skipped at baseline>

## Results

| Test | Claim | Mutation (file:line — what) | Result / evidence | Verdict |
| --- | --- | --- | --- | --- |
| tests/billing/test_invoices.py::test_total_includes_tax | total = subtotal + tax | billing/invoice.py:42 — `+ tax` -> `- tax` | AssertionError: 110 != 90 | VALID |
| tests/billing/test_invoices.py::test_no_email_when_draft | send is not called for draft invoices | billing/invoice.py:77 — dropped `if not draft` | still green; also mutated to always send: still green (send is patched at module level) | WEAK |

## Progress log
<one line per batch>

## Summary
VALID n · PARTIAL n · WEAK n · INVALID-MUTATION n · BLOCKED n · SKIPPED n (total n)

## Patterns found
- <pattern> — <tests affected> — <suggestion>

## Skipped / blocked
- <test id> — <reason>

## Baseline integrity
git status: <clean / list>   git diff --stat: <empty / list>
checksums: <all match / mismatches listed>
```

The "Result / evidence" column must quote the runner's failure line (or "still green") — not a paraphrase.

## Wrap-up and fix step

When all tests have a verdict:

1. Confirm the baseline integrity section is clean. If not, show the diff and stop; nothing else happens until the tree is restored.
2. Present the summary counts, the patterns, and the WEAK/PARTIAL rows to the user.
3. **Ask** whether to fix, and how. Offer:
   - only WEAK
   - only PARTIAL
   - a particular directory, file, or pattern (e.g. "all vacuous negatives")
   - everything flagged
   - report only (default if the user is unsure)
4. The fix pass strengthens only the tests chosen. For each one: edit the test in the project's existing test style (same fixtures, factories, assertion helpers, naming; no explanatory comments), then **re-apply the surviving mutation** from the report, run the test, confirm it now goes red with an assertion tied to the claim, restore the production file via backup + checksum, run again, confirm green. Record the new evidence in the report row.
5. Never commit, push, or open a PR unless the user explicitly asks. If the audit ran in a separate tree, tell the user where the fixed test files are and offer to copy them over.

## Safety rules

- Trees may contain uncommitted work. During the audit **never** run `git checkout`, `git restore`, `git stash`, `git reset`, `git clean`, `git add`, `git commit`, `git push`.
- Restore only by copying the backup of the mutated file back and verifying its checksum against the baseline.
- Keep `CURRENT_MUTATION.txt` (file, backup path) in the tree root while a mutation is live; remove it after restore. On start, if it exists, restore first.
- One mutation live at a time. One file mutated at a time.
- Run only the selected test; never the whole suite while a mutation is live.
- Never run a linter or formatter in autofix mode while a mutation is live (e.g. `ruff check --no-fix`, `eslint` without `--fix`, `gofmt -l` not `-w`).
- Never edit test files in the audit phase; only in the fix pass, and only the tests the user selected.
- Report goes to an untracked location only.
- At the end, compare `git status`, `git diff --stat`, and the checksum list to the baseline recorded at start and write the result into the report. A mismatch is a finding that blocks everything else.
