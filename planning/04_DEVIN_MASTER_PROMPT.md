# Devin master instruction — Jobbmatch

You are responsible for implementing and handing over a working, tested recruitment platform from the attached planning package. Treat this as a phased engineering delivery, not a request for a visual prototype or an architecture-only answer. Product copy is Swedish. Code, engineering documentation and PRs may be English. Provide founder-facing progress summaries in plain Swedish.

## Context and authoritative specification

The founder wants a swipe-based recruiting product that removes repeated application paperwork and shortens the path to a real conversation. Build a candidate app for iOS and Android, a responsive employer web portal, and a protected platform administration interface.

Read these files completely before implementation:

- `planning/START_HAR.md`
- `planning/01_PRODUKT_OCH_AFFAR.md`
- `planning/02_TEKNISK_SPECIFIKATION.md`
- `planning/03_BYGGPLAN_OCH_ACCEPTANS.md`
- `planning/05_MAC_XCODE_OCH_PUBLICERING.md`
- `planning/06_KALLOR_OCH_BESLUT.md`

If files are supplied as attachments instead, place their exact contents under `planning/`. Do not silently discard requirements. If a mandatory attachment is genuinely missing, request that specific file and continue only with independent setup. This prompt sets engineering execution rules; the specification sets detailed product behavior. Record contradictions in `docs/decisions.md` and resolve routine implementation choices without waiting. Ask only when missing business identity, credentials, legal decisions, cost authorization, or a material product contradiction prevents safe progress.

Working product name: Jobbmatch. This is not a cleared trademark. Initial pilot: adults, restaurant/café jobs in Stockholm, private employers. Do not claim the idea is unique, hiring is guaranteed, or the software is legally certified.

## Mandatory stack and scope

- React Native + Expo + TypeScript + Expo Router for candidate iOS/Android apps.
- Next.js + TypeScript for the employer portal, public support/privacy/deletion pages, and protected admin.
- Supabase PostgreSQL, Auth, Realtime and server-side functions for the backend.
- Specific EU database region where available; document all other processor/data flows separately.
- GitHub-owned source, migrations and CI; Expo EAS for native builds and test distribution when authorized credentials exist.
- Monorepo with `apps/mobile`, `apps/web`, shared domain/API types/design tokens, `supabase`, and `docs`.
- Pin a currently stable compatible toolchain. Expo determines compatible React Native and native package versions. Do not upgrade independently to incompatible latest versions.

Do not substitute a website/PWA for the native apps. Do not use Flutter unless you identify a concrete blocking incompatibility and obtain an explicit architecture change. Do not introduce AI ranking, candidate scoring, video/CV upload, open candidate discovery, automatic rejections, payments, or a general HR suite. Employer native screens and candidate web are later scope; the employer portal must work on desktop, tablet and mobile browsers.

## Non-negotiable product behavior

1. Candidates can browse published jobs before registration. Email one-time-code login establishes their account. Build a short structured reusable profile without photos, birth dates, government ID numbers or exact home addresses.
2. Job cards show job title, employer, place, salary range with currency/unit, hours, employment type and requirements. Detail screens show the complete ad.
3. Candidate swipe-right or the explicit `Sök jobbet` button submits an application. Explain what profile information is shared before the first submission. Swipe-left means pass; bookmark means save. Accessible buttons provide every gesture action.
4. Employers are verified manually by platform admins before publishing. Ads have explicit moderation, status, revisions and expiry. Employer users can access only their own organization.
5. Employers review actual applications. Their right swipe/`Visa intresse` is a human decision that opens a match and a conversation for that application. Employer left swipe means `Senare`, never a hidden rejection. Explicit rejection uses confirmation and a respectful notification.
6. One candidate–job pair has at most one application in v1. One application has at most one match and conversation. Database constraints and transactions enforce this under concurrency, not only disabled buttons.
7. No unsolicited employer messages to people who have not applied. No public candidate directory. No candidate attractiveness, popularity or inferred fit scoring.
8. Implement durable text chat, reconnect synchronization, interview proposals/acceptance/cancellation, notifications and user preferences. No attachments in v1.
9. Withdrawal, blocking, job closure, organization suspension and membership removal have server-enforced effects. Pending notifications must respect current state before delivery.
10. Implement in-app and web-accessible account deletion, data export, retention jobs, reporting, moderation and auditability. Soft delete alone is insufficient.

Use exact lifecycle rules from the technical specification. Job closure stops applications; existing matched conversations have a limited continuation window. A withdrawn application cannot be resubmitted to the same job in v1. Record job revision and minimal profile snapshot at submission, and include those copies in deletion/retention handling.

## Design standard

Create a calm professional Swedish interface with clear salary/hours information, readable typography, generous touch targets and restrained animation. Use a coherent design-token system. Do not copy Tinder branding, visual identity or dating language.

Candidate navigation: Hitta jobb, Sparat, Ansökningar, Meddelanden, Profil. Employer navigation: Översikt, Jobb, Ansökningar, Meddelanden, Intervjuer, Team/Inställningar. Admin must be separately authorized.

All relevant screens need loading, empty, validation, network-failure, session-expired and permission-denied states. Support large text, screen readers, keyboard on web, reduced motion and accessible action labels. Do not show success until durable server acceptance. Use optimistic UI only when rollback and retries are correct and understandable.

## Security and data engineering

- Derive user identity from verified sessions, never input user IDs.
- Default-deny RLS on exposed tables; cover views, RPCs, writes and any storage. Test direct unauthorized requests with real user roles and two organizations.
- Do not expose service-role credentials in app/web builds, logs, example files or Git. Public client configuration is not a substitute for authorization.
- Keep membership checks current; an old token must not preserve access after removal. Realtime subscriptions must not leak new events after revocation.
- Privileged SQL functions require restricted grants, fixed search paths and explicit authorization checks.
- Apply payload limits, rate limits, OTP expiry/replay protection, safe text rendering, web origin/CSRF controls where relevant and secure session storage.
- Store state changes and notification outbox entries atomically. Use idempotency keys, unique constraints, row locking/optimistic versions and retry-safe workers. Explain limits of external delivery deduplication honestly.
- Use generic lock-screen notification text without candidate details or message bodies.
- Keep logs, analytics and crash reports free of message content, profile snapshots, secrets and unnecessary contact details.
- Require admin MFA and audit moderator actions. Prevent self-promotion and last-owner lockouts.
- Use synthetic seed data only. Do not scrape real candidates or create fake production jobs.
- Write a data inventory and implement configurable retention. Defaults in planning are proposals requiring the owner's legal decision before real candidate data is processed. Do not invent legal approval or processor contracts.

## Work in milestones, and keep implementing

Create a requirement-to-test matrix and track progress in `docs/implementation-status.md`. Use the M0–M4 sequence in the build plan.

M0: reproducible repository, local backend/seed, CI, version decisions, navigable shells, threat model and configuration inventory.

M1: an end-to-end vertical slice with real auth, profile, verified employer, moderated job, application and match. Prove isolation and race safety before expanding.

M2: complete candidate and employer UX, filtering/saving, revision handling, chat, interviews, notifications and team management.

M3: moderation, privacy lifecycle, security hardening, accessibility, operational alerts, backup/restore exercise and all required negative tests.

M4: actual native build validation, device tests, authorized test distribution, pilot fixes, release materials and complete Mac handover.

At each milestone provide a concise Swedish progress report, reviewable changes, verification evidence and actual remaining blockers. Continue to the next technically independent task without repeatedly requesting permission for ordinary code changes. Do not stop after scaffolding or writing a plan. Keep PRs reviewable and keep an integration branch working. Do not bypass required repository checks or merge protections.

If an external account is absent, use a clearly labeled local adapter for local testing where useful, complete the real production adapter and document missing configuration. Never leave mocks on the production path or present untested integrations as working. Do not buy services, publish publicly, deploy production migrations or release to stores without the owner's explicit release/cost authorization. Continue local and staging preparation while those actions are pending.

## Verification requirements

Implement and execute the applicable A01–A30 acceptance tests. Security tests must use a real local/staging database, not only mocked clients. Test simultaneous submissions, interest, withdrawal and job closure. Test cross-tenant reads/writes, stale sessions, invitations, block bypass attempts, deletion of snapshots and outbox suppression.

Use unit/integration tests for domain rules; Playwright for employer/admin; a compatible native end-to-end runner for iOS and Android; actual device checks for push, deep links, keyboard, background/restart and screen-reader behavior.

Record command, result, environment, commit and limitations. A browser screenshot is not evidence of a successful iOS build. A simulator build is not a physical-device signing test. A build upload is not store approval. Never fabricate evidence.

Measure pagination/performance with synthetic load. Validate backup restoration in isolation and reapply deletion records before reactivation. Run dependency and secret scans and inspect findings. Release blockers include tenant isolation failure, broken deletion/blocking, core-flow crashes or wrong backend configuration.

## Deliverables and definition of done

Deliver all of the following, distinguishing available artifacts from externally blocked ones:

1. Complete source code in the founder-owned repository with a tested commit/tag and all migrations.
2. One-command or short reproducible local setup, pinned runtime/package manager, `.env.example`, seed data and documented checks.
3. Working candidate iOS/Android apps and responsive employer/admin portal connected to the real backend.
4. RLS/authorization matrix, threat model, data inventory, retention/deletion specification and automated tests.
5. Build profiles for development, simulator, preview/staging and production; correct separate app identifiers/environments.
6. CI workflows, monitoring setup, incident/backup/restore and backward-compatible release/rollback instructions.
7. Native build evidence: commit, build profile, build ID, artifact link, signing status, environment and device test results. If credentials are missing, mark the specific artifact blocked rather than claiming completion.
8. Store assets and accurate draft metadata, privacy/data-safety inventory, reviewer access instructions, support/deletion URLs and permission descriptions. Real owner identity must replace placeholders before submission.
9. `docs/MAC_HANDOVER.md`: clone the tested release on the owner's Mac, install the pinned tooling, configure local secrets, run the app, generate the iOS project, open the actual `.xcworkspace` in Xcode, select signing team and build. Test these instructions on macOS if access is available; state explicitly otherwise.
10. An optional clean source ZIP tied to the same commit, excluding secrets, caches, dependencies and signing keys. Source is the main handover, not only an IPA.
11. `docs/RELEASE_REPORT.md`: what passed, what failed, what was not run, known issues, owner actions and precise next steps. Include no claims of legal certification or guaranteed store approval.

Use Expo Continuous Native Generation. Preserve native customizations in configuration/plugins, not undocumented manual changes that prebuild would overwrite. Generate native projects as needed. Do not copy build files onto the owner's Mac or claim they were transferred without actual authorized access and verification. GitHub clone/pull is the standard transfer path; a signed IPA goes to TestFlight, not into Xcode as editable source.

Start now by inspecting the repository and specifications, documenting decisions and creating the M0 foundations. Then implement the complete in-scope product through the milestones, reporting truthfully and keeping any genuinely external blocker separate from work you can still complete.
