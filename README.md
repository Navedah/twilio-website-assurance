# Twilio Public Website — Kane CLI Assurance

Requirements-to-evidence assurance for the Twilio public website, driven by
[Kane CLI](https://www.npmjs.com/package/@testmuai/kane-cli).

Application under test:
https://www.twilio.com/en-us — English United States locale, **anonymous (logged-out)
experience only**.

## Learning objective

Demonstrate this lifecycle end to end:

```text
Requirement → Use Case → Acceptance Criteria → Scenario → test.md → Execution → Evidence → Coverage
```

## What this suite proves

Exactly one use case has been carried all the way to a committed, executable suite:
**uc-2 Contact sales**, derived from PRD requirement `FR-CONV-03`.

| Test | Verifies | Type |
| --- | --- | --- |
| `reach-the-sales-contact-form-from-every-visible-homepage_test.md` (t-3) | AC-1, AC-2 | happy path |
| `validate-empty-required-sales-form-input-without-submission_test.md` (t-1) | AC-2, AC-3, AC-5 | negative |
| `validate-malformed-sales-form-input-without-submission_test.md` (t-2) | AC-2, AC-4, AC-6 | negative |

The other nine extracted use cases are **not covered**. They are listed explicitly in
[`requirements/twilio-requirements.md`](requirements/twilio-requirements.md), because
stating what is unverified is part of the assurance record.

### Test boundary

No credentials are used and **no form is ever submitted**. Submitting the sales form would
create a real commercial lead, so every conversion test stops at verifying that the form
renders and that client-side validation fires. Invalid input is synthetic: an empty string,
a single space, an address missing the at sign, and an address with a trailing dot.

## Repository layout

```text
twilio-website-assurance/
├── .github/
│   └── workflows/
│       ├── twilio-assurance.yml      ← execution + coverage
│       └── requirements-drift.yml    ← maintain, plan-only
├── requirements/
│   ├── PRD-Twilio-Website-and-Test-Cases.docx   ← ingested source of record
│   └── twilio-requirements.md                   ← requirement slice + traceability
├── .testmuai/
│   └── tests/
│       ├── reach-the-sales-contact-form-from-every-visible-homepage_test.md
│       ├── validate-empty-required-sales-form-input-without-submission_test.md
│       └── validate-malformed-sales-form-input-without-submission_test.md
├── .context/                             ← assurance graph (committed, see below)
├── .gitignore
└── README.md
```

### Why `.context/` *is* committed

The Kane CLI assurance store under `.context/` is append-only and single-writer, so it is not
a Git-mergeable artifact. It is committed anyway, deliberately, because two capabilities need
the live graph and are worth more than the merge risk:

- `kane-cli cover` measures completeness against the graph — without it there is no coverage
  axis, only a pass/fail list.
- `kane-cli maintain reconcile` diffs a changed PRD against the graph to show suite drift.

The rule that follows: **never write to `.context/` from two places at once**, and on a
conflict regenerate rather than hand-merge. CI only ever reads it.

What still gets committed from the design phase is its **reviewed output** — the `_test.md`
files and the requirement documents.

Each committed test carries an `assurance:` frontmatter block naming its graph id and the
definition hash it was designed against:

```yaml
---
assurance:
  id: t-3
  base: sha256:86db92fa9a73dd2551d1984870b5d7a3ab42ca97f3f74cb26e6fdc77a8c7400e
---
```

Individual steps are tagged `@verifies ac-N`, so a step-level failure names the acceptance
criterion it broke.

## Phase split

The **design** phase is performed by a human on a workstation:

1. Ingest the requirements.
2. Extract use cases.
3. Review and promote the proposed use cases to trusted.
4. Design acceptance criteria, scenarios and tests.
5. Review the generated design.
6. Commit the reviewed `_test.md` files.

GitHub Actions then performs the repeatable **execution** phase:

1. Install Kane CLI.
2. Authenticate using GitHub Secrets.
3. Run the committed `_test.md` suite headless.
4. Validate the generated evidence pack.
5. Upload evidence and reports as workflow artifacts.

## Prerequisites

- TestMu AI account with Kane CLI access, plus a username and access key.
- Node.js 18+ (this suite is exercised on Node 20).
- Google Chrome, for local execution.
- Kane CLI 0.6.1 or later for the assurance commands. Authored against 0.8.11.

```bash
npm install -g @testmuai/kane-cli
kane-cli --version
```

## Phase 1 — Local assurance design

```bash
kane-cli login
# or, non-interactively:
kane-cli login --username "<username>" --access-key "<access-key>"
```

### 1. Ingest requirements

```bash
kane-cli context ingest ./requirements/PRD-Twilio-Website-and-Test-Cases.docx
```

This snapshots the requirement document into the local assurance store under `.context/`.

### 2. Extract use cases

```bash
kane-cli context extract
```

Kane proposes use cases from the requirement document and cites the source material. For
this PRD the proposal landed as `uc-1` through `uc-10`.

### 3. Review

```bash
kane-cli context review
```

Promote the useful proposals to trusted; edit or reject the rest.

### 4. Design tests

For each trusted use case:

```bash
kane-cli design tests --use-case uc-2
```

Review the generated acceptance criteria, scenarios and `_test.md` files before they become
part of the trusted suite. The files under `.testmuai/tests/` are the reviewed result of
this step for `uc-2`.

### 5. Review the design

```bash
kane-cli context review
```

Do not skip this. The generated design is derived content and should be reviewed before it
is trusted.

## Phase 2 — Local execution

List what is committed:

```bash
kane-cli testmd list
```

Run one test:

```bash
kane-cli testmd run \
  ./.testmuai/tests/reach-the-sales-contact-form-from-every-visible-homepage_test.md \
  --agent
```

CI-style local execution of a single test:

```bash
kane-cli testmd run \
  ./.testmuai/tests/reach-the-sales-contact-form-from-every-visible-homepage_test.md \
  --agent \
  --headless \
  --on-lock-conflict wait \
  --retry
```

Plan the full suite without executing it:

```bash
kane-cli testrun run --dry-run --headless
```

Run the full suite:

```bash
kane-cli testrun run --headless --on-failure continue
```

A batch `testrun` produces one sealed evidence pack for the whole suite.

Filter by tag instead of path — every test carries `contact-sales`, plus `smoke`/`happy-path`
or `validation`/`negative`:

```bash
kane-cli testrun run --headless --tags validation
```

## Phase 3 — Inspect evidence

After a run, look under `.testmuai/evidence/`:

```bash
kane-cli evidence validate .testmuai/evidence/<execution-id>.evidence --json
kane-cli evidence serve .testmuai/evidence/<execution-id>.evidence
```

The pack contains the test definitions, results, screenshots, console and network logs and
failure information. Per-test reports land in `.testmuai/tests/output-*/Result.md`.

Merge several packs into one archive for distribution:

```bash
kane-cli evidence merge .testmuai/evidence/*.evidence --run-id local-1 --out merged.evidence
```

Measure coverage against the committed graph — from the **batch** pack that `testrun` sealed,
not the merged one:

```bash
kane-cli cover --from .testmuai/evidence/<execution-id>.evidence
```

> `evidence merge` does not carry `coverage/usecases.yaml` through, so `cover` on a merged
> pack fails with *"carries no coverage/usecases.yaml"*. Merge for distribution; measure
> coverage from the batch pack. The workflow reads the execution id out of the `testrun`
> output to pick it automatically.

A per-member `testmd run` pack only proves its own criteria — in one run the three members
scored 17%, 17% and 33% individually, while the batch pack scored 100%.

Do not commit `.testmuai/evidence/`.

## Phase 4 — GitHub Actions

Create these repository secrets under **Settings → Secrets and variables → Actions**:

| Secret | Value |
| --- | --- |
| `LT_USERNAME` | TestMu AI username |
| `LT_ACCESS_KEY` | TestMu AI access key |

The workflow fails fast with a clear error if either is missing.

### Test Manager destination

`kane-cli testrun run` seals evidence locally but reports `upload: skipped` — **on its own it
publishes nothing to Test Manager**. Test cases appear in Test Manager because the workflow
runs `kane-cli testmd run` per member first, which authors, replays and publishes each case.

To control where those cases land, set two **repository variables** (Settings → Secrets and
variables → Actions → *Variables*, not Secrets — these are ids, not credentials):

| Variable | Meaning |
| --- | --- |
| `KANE_PROJECT_ID` | Test Manager project id. Blank = account default. |
| `KANE_FOLDER_ID` | Folder id within that project. Blank = account default. |

Find them locally with `kane-cli config show`. The workflow sets them with
`kane-cli config project <id>` / `kane-cli config folder <id>`.

Set `publish_to_test_manager: false` on a manual dispatch to skip publishing and only produce
local evidence.

### The execution workflow

[`.github/workflows/twilio-assurance.yml`](.github/workflows/twilio-assurance.yml) runs on
every pull request, on pushes to `main`, and on manual dispatch. It is also callable from
another repository via `workflow_call`. It:

1. Checks out the repository and installs Node.js 20, with the npm cache restored.
2. Installs Kane CLI and verifies the toolchain, including Chrome.
3. Logs into TestMu AI using the GitHub Secrets.
4. Points Kane at the configured Test Manager project and folder.
5. Lists and plans the committed suite (`--dry-run`) before executing anything.
6. Publishes and replays each test with `testmd run`, collecting Test Manager links.
7. Runs the whole suite headless as one sealed `testrun`.
8. Validates every generated evidence pack.
9. Merges the packs into one sealed pack for the run.
10. Produces a two-axis **coverage** report with `kane-cli cover --from`.
11. Writes execution, coverage and Test Manager tables to the job summary.
12. Uploads evidence, reports, coverage and outputs as artifacts, retained 30 days.

Steps 8–11 run under `if: always()`, so a failing suite still yields evidence and coverage.

### Coverage

`kane-cli cover` reports two axes, and the job summary renders both:

- **Depth** — of the acceptance criteria the graph knows about, which were actually proven by
  this evidence pack.
- **Completeness** — what the graph still owes: use cases with no scenario, criteria with no
  test. This is how the nine undesigned use cases stay visible instead of being forgotten.

### Requirement drift

[`.github/workflows/requirements-drift.yml`](.github/workflows/requirements-drift.yml) runs
when anything under `requirements/` changes on a pull request. It runs
`kane-cli maintain reconcile --plan`, which lands the head move and **stages** every proposed
suite change without committing any of it, then posts the diff to the job summary.

Nothing in CI promotes a change to trusted. Approving the staged plan is a human action on a
workstation:

```bash
kane-cli maintain reconcile --apply
```

### Manual dispatch inputs

| Input | Default | Purpose |
| --- | --- | --- |
| `on_failure` | `continue` | `continue` runs the whole suite and produces complete evidence; `fail-fast` stops at the first failure. |
| `match` | *(empty)* | Regex over the project-relative test path, to run a subset. |

The default is `continue` rather than `fail-fast` so that a single failure still yields
evidence for every member of the suite.

### A note on the runner

The suite drives a real Chrome browser against the live public Twilio site. It is therefore
subject to the live site changing, and to network conditions on the runner. A failure means
one of three things, and the evidence pack is what tells them apart:

- a **product assertion** genuinely failed;
- an **environment or test problem** prevented verification;
- the **requirement itself changed** and the suite is now out of date.

## Discussion prompts

1. Which requirement does this test prove?
2. Which acceptance criteria are covered, and which are not?
3. What evidence proves the criterion?
4. If the test fails, is the product wrong or is the environment broken?
5. What remains unverified?
6. What happens to the suite if the requirement changes?
