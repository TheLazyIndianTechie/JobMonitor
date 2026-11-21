# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

As of 2026-11-21 this repo holds documentation only for IGG Job Monitor (MWAP): a summary
`README.md` and `IGG_MWAP_Project_Status_and_Roadmap_20_Nov_2026.md`. The Google Apps
Script code and the Google Sheet it drives live outside this repo, so there are no
build, lint or test commands. Update this file when code lands here.

The status and roadmap document is the source of truth for what is built, what is
tested and what comes next (roadmap steps 10.1 to 12). Read it before proposing work.

## Deployment

The GitHub repo is connected to the Vercel project `igg-job-monitor` on the Pro team
`lateprofessors-projects`. Pushes to `master` deploy to production; other branches get
preview deployments. On Pro, a commit deploys only if its author is a member of the
Vercel team, and each developer seat is billed.

Vercel serves only `public/`, a hand-written static landing page (`public/index.html`,
no build step). The page is public, so it carries the README-level summary only and
links to the full docs on GitHub. The Markdown docs at the repo root are deliberately
not served; don't move them into `public/` without the user's approval. Keep the
landing page in step with `README.md` when the summary changes.

## What the system does

MWAP discovers videogame-industry vacancies in India, verifies they are genuine and
active, removes duplicates, routes each job to the right WhatsApp Community group, and
distributes it only after a human approves it.

## Architecture

- **Google Sheets is the database.** Tabs cover jobs, sources, employers, categories,
  group mappings, the approval queue, the distribution log, configuration and
  audit/error logs. Key tabs: `JOBS_MASTER`, `APPROVAL_AUDIT_LOG`, `DISTRIBUTION_LOG`.
- **A standalone Google Apps Script project is the engine.** It is connected to the
  spreadsheet by ID, not bound to it.
- **Pipeline order:** discovery (ASGC job-listings endpoint, source `SRC001`) →
  eligibility filtering (employer, role, India relevance) → vacancy verification →
  duplicate detection → database preview → production writer → approval queue →
  distribution.
- **WhatsApp posting is not built.** The candidate is WAHA Core, an unofficial
  self-hosted WhatsApp Web gateway, run locally in Docker with the existing personal
  number.

Rules that span components:

- A vacancy counts as active only on positive vacancy-specific evidence. A successful
  HTTP response is not enough. Platform-specific verifiers exist for Workday,
  Greenhouse, SmartRecruiters and EA, with a generic fallback; inconclusive results
  stay unverified.
- Duplicates are matched on employer + requisition ID and employer + specific
  application URL. Title similarity plus location compatibility flags possible
  duplicates. Source history must be preserved.
- Routing uses `Job_Discipline` and needs exactly one enabled group mapping. Missing or
  conflicting mappings put the job on hold.
- Approval is separate from publishing. An approval decision (`PENDING`, `APPROVED`,
  `REJECTED`, `HOLD`) updates `JOBS_MASTER.Review_Status`; `Publish_Status` is tracked
  separately and approval never sends a message.
- An initial decision is applied automatically. A changed decision is marked
  `REPROCESSING_REQUIRED` and is applied only when `Reprocess_Confirmed` is `TRUE`; the
  flag resets to `FALSE` afterwards and the change is written to `APPROVAL_AUDIT_LOG`.
- Newly promoted employers start disabled pending review.

## Operating constraints

The system is in development and validation. It is not production-ready, and these
settings must not be changed without the user's explicit approval:

- `EMERGENCY_PAUSE` stays `TRUE` and `MONITORING_ENABLED` stays `FALSE`.
  `OPERATING_MODE` is `APPROVAL_REQUIRED`.
- Source `SRC001` (ASGC) stays disabled pending a terms and permission review.
- No production collection or insertion of genuine vacancies. Production activation
  needs separate approval.
- WAHA testing starts read-only: connect the account and list groups. Send nothing to
  any group without explicit authorisation, and never to a real distribution group as
  a test.
- Never expose the WAHA interface publicly without authentication and network access
  controls. Protect the linked-device session and API credentials.
- Don't buy hosting or plan a VPS until local compatibility and delivery are validated.
