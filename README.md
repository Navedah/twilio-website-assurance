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
│       └── twilio-assurance.yml
├── requirements/
│   ├── PRD-Twilio-Website-and-Test-Cases.docx   ← ingested source of record
│   └── twilio-requirements.md                   ← requirement slice + traceability
├── .testmuai/
│   └── tests/
│       ├── reach-the-sales-contact-form-from-every-visible-homepage_test.md
│       ├── validate-empty-required-sales-form-input-without-submission_test.md
│       └── validate-malformed-sales-form-input-without-submission_test.md
├── .gitignore
└── README.md
```

### Why `.context/` is not committed

The Kane CLI assurance store under `.context/` is append-only and single-writer. It is not a
Git-mergeable artifact, so it stays out of version control. The design phase happens on a
workstation; what gets committed is the **reviewed output** of that phase — the `_test.md`
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

## Phase 3 — Inspect evidence

After a run, look under `.testmuai/evidence/`:

```bash
kane-cli evidence validate .testmuai/evidence/<execution-id>.evidence --json
kane-cli evidence serve .testmuai/evidence/<execution-id>.evidence
```

The pack contains the test definitions, results, screenshots, console and network logs and
failure information. Per-test reports land in `.testmuai/tests/output-*/Result.md`.

Do not commit `.testmuai/evidence/`.

## Phase 4 — GitHub Actions

Create these repository secrets under **Settings → Secrets and variables → Actions**:

| Secret | Value |
| --- | --- |
| `LT_USERNAME` | TestMu AI username |
| `LT_ACCESS_KEY` | TestMu AI access key |

The workflow fails fast with a clear error if either is missing.

[`.github/workflows/twilio-assurance.yml`](.github/workflows/twilio-assurance.yml) runs on
every pull request, on pushes to `main`, and on manual dispatch. It:

1. Checks out the repository and installs Node.js 20.
2. Installs Kane CLI and verifies the toolchain, including Chrome.
3. Logs into TestMu AI using the GitHub Secrets.
4. Lists and plans the committed suite (`--dry-run`) before executing anything.
5. Runs the suite headless.
6. Validates every generated evidence pack.
7. Writes a per-test status table to the job summary.
8. Uploads evidence, `Result.md` files and test outputs as artifacts, retained 14 days.

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
