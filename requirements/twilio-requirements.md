# Twilio Public Website — Assurance Requirements

## Source application

Twilio public website, English United States locale, anonymous (logged-out) experience only:

https://www.twilio.com/en-us

## Source document

These requirements are extracted from the reverse-engineered PRD committed alongside this
file as `PRD-Twilio-Website-and-Test-Cases.docx`. That document is the ingested source of
record for the Kane CLI assurance store; this markdown file is the human-readable
requirement slice that the committed `_test.md` suite actually proves.

Baseline captured: September 11, 2026.

## Scope constraint

No account credentials are available for this engagement. Every requirement below is
verifiable as an anonymous visitor. Where a requirement reaches a conversion form, coverage
ends at verifying that the form renders and validates correctly. **The submit action is
never performed**, because submitting the sales form would generate a real commercial lead.

## Requirement RQ-CONV-03 — Contact sales path

*Derived from PRD `FR-CONV-03` (P0): "Every Contact sales control leads to the sales contact
page, which renders its form fields and validates them client-side."*

An anonymous visitor should be able to follow a Contact sales call to action from the public
homepage to the sales contact page, confirm the form renders, and see client-side validation
reject empty and malformed input — stopping before submission so that no real sales lead is
created.

### Acceptance Criteria

- **AC-1** — Every visible Contact sales control on the public homepage opens the sales contact page.
- **AC-2** — The sales contact page renders interactive sales form fields.
- **AC-3** — The sales form displays inline validation when empty input is triggered.
- **AC-4** — The sales form displays inline validation when malformed input is triggered on a format-constrained field.
- **AC-5** — After empty required sales-form input triggers validation, the sales form remains visible on the sales contact page for correction.
- **AC-6** — After malformed sales-form input triggers validation, the sales form remains visible on the sales contact page for correction.

### Supporting PRD test cases

| PRD case | Requirement | Expectation |
| --- | --- | --- |
| TC-CONV-009 | FR-CONV-03 | Click a contact sales control. The sales contact page loads and renders its form fields. |
| TC-CONV-010 | FR-CONV-03 | Trigger validation on the sales form with empty and malformed input. Inline validation appears for each field; the form is not submitted. |
| TC-NAV-016 | FR-NAV-07 | Inspect the header at each breakpoint. Contact sales and Start for free are both present and tappable. |

### Test data

Synthetic input only. No form is ever submitted. The invalid inputs exercised by this suite
are drawn from the PRD's suggested set: an empty string, a single space, an address missing
the at sign, and an address with a trailing dot.

## Traceability

Requirement → Use Case → Acceptance Criterion → Scenario → `_test.md`

| Use case | Acceptance criteria | Scenario | Committed test |
| --- | --- | --- | --- |
| uc-2 Contact sales | AC-1, AC-2 | Reach the sales contact page from a homepage Contact sales control | [`reach-the-sales-contact-form-from-every-visible-homepage_test.md`](../.testmuai/tests/reach-the-sales-contact-form-from-every-visible-homepage_test.md) (t-3) |
| uc-2 Contact sales | AC-2, AC-3, AC-5 | Show inline validation for empty required sales-form input | [`validate-empty-required-sales-form-input-without-submission_test.md`](../.testmuai/tests/validate-empty-required-sales-form-input-without-submission_test.md) (t-1) |
| uc-2 Contact sales | AC-2, AC-4, AC-6 | Show inline validation for malformed sales-form input | [`validate-malformed-sales-form-input-without-submission_test.md`](../.testmuai/tests/validate-malformed-sales-form-input-without-submission_test.md) (t-2) |

Each committed test carries an `assurance:` frontmatter block naming its graph id (`t-1`,
`t-2`, `t-3`) and the definition hash it was designed against. Individual steps are tagged
`@verifies ac-N`, so a step-level failure names the acceptance criterion it broke.

## Extracted but not yet designed

The requirements ingest proposed ten use cases from the PRD. Only **uc-2 Contact sales** has
been carried all the way through design to a committed, executable test suite. The remaining
nine are extracted and awaiting design, and therefore have **no coverage in this repository**:

| Use case | Related PRD requirements | Status |
| --- | --- | --- |
| uc-1 Begin a free trial | FR-CONV-01, FR-CONV-02 | Extracted, not designed |
| uc-3 Browse products, solutions, resources, and pricing | FR-NAV-03 … FR-NAV-06, FR-NAV-11 | Extracted, not designed |
| uc-4 Evaluate the platform with homepage content | FR-HOME-01 … FR-HOME-05 | Extracted, not designed |
| uc-5 Evaluate integration options with code samples and documentation | FR-HOME-06, FR-HOME-07 | Extracted, not designed |
| uc-6 Assess Twilio credibility and sources | FR-HOME-08 … FR-HOME-11 | Extracted, not designed |
| uc-7 Switch the website language | FR-NAV-08, FR-NAV-09 | Extracted, not designed |
| uc-8 Search the public website | FR-NAV-10 | Extracted, not designed |
| uc-9 Reach support, status, and login entry points | FR-CONV-05 | Extracted, not designed |
| uc-10 Reach company, legal, and social destinations | FR-FOOT-01 … FR-FOOT-05 | Extracted, not designed |

Stating what is *not* covered is part of the assurance record. Coverage claims in this
repository are limited to RQ-CONV-03.
