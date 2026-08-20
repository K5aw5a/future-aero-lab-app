# Future Aero Lab Workspace

**Internal team platform — from a client-only tool to an authenticated, HubSpot-integrated backend**

*Role: Technical lead / sole engineer. Stack: Node.js (zero runtime dependencies), SQLite, vanilla JS frontend, GitHub Actions, self-hosted Linux (Hetzner).*

---

## Context

Future Aero Lab runs an aviation/energy startup accelerator, tracking startups, mentors, investors, sessions, and documents across Dropbox, Google Drive, HubSpot, and Mailchimp. The team's internal tool started as a static, client-only app: every person's browser held its own copy of the data in local storage, with no login and no way to push an edit back to the systems the data came from.

I led the project to turn that into a real shared platform — a proper backend, authenticated access for the whole team, and a safe, human-reviewed way to sync changes back to HubSpot — without adding a build step or a framework, and without a single day of downtime for a team actively using the tool.

## What I built

**A shared backend, not a rewrite.** Replaced per-browser local storage with a small self-hosted Node service (SQLite, session-based auth, a REST API) that the existing frontend was migrated onto incrementally — reads first, writes behind an optimistic-concurrency guard, then a login gate — so the cutover could be staged and verified at each step rather than shipped as one large, risky change.

**Two-way HubSpot sync, human-reviewed by design.** Rather than syncing changes automatically, every edit to a HubSpot-sourced record is captured as a reviewable diff in a sync queue. A person previews the live HubSpot record and the exact fields a push would change, then explicitly approves it — and a conflict check blocks the push outright if the record changed in HubSpot since the edit was made, rather than silently overwriting it. I shipped this for one entity type first, proved the pattern against a real HubSpot account, then extended it to a second — and when extending it to a third would have meant two different local records able to push conflicting edits to the same underlying HubSpot record, I designed around that risk instead of building the parallel path.

**Verification against the real system, not just tests.** Every integration was checked against a live HubSpot account with disposable test data before being trusted — not just a passing test suite. That step caught a real, non-obvious bug: one HubSpot object type used a differently-named timestamp field than the other, which would have silently caused every update to one entity type to be falsely flagged as a conflict. A mocked test alone could not have surfaced this, because the mock had made the same wrong assumption as the code it was testing.

**Production infrastructure.** Set up and administered the Linux host end to end: systemd service, nginx reverse proxy, a CI pipeline that gates every deploy on the test suite passing, and off-site encrypted backups with a deliberate cost/tradeoff comparison between providers.

**A staged, communicated rollout.** Login enforcement is a real behavior change for every team member — it shipped with advance notice, a defined sequence (build and verify locally, set real credentials, confirm the team was ready, then deploy), and diagnosis of two live issues that surfaced during rollout (a misrouted database path, a case-sensitivity bug in login) traced from symptoms to root cause rather than guessed at.

## Outcome

- Migrated an entire team off per-browser local state onto one authenticated, shared backend with zero data loss
- Shipped a working, human-reviewed write-back path to a real third-party CRM, verified against production data
- Left the project with living architecture documentation, a decision log, and an automated test suite covering both backend logic and frontend behavior — grown from zero to several hundred checks over the course of the project

## Stack

Node.js · SQLite · vanilla JavaScript (no framework, no build step) · GitHub Actions · nginx · systemd · HubSpot API · Backblaze B2

## Skills demonstrated

Backend architecture and API design · third-party API integration (OAuth-scoped credentials, conflict detection) · Linux server administration · CI/CD · authentication and session security · staged/risk-managed production rollouts · live debugging and root-cause diagnosis · technical documentation
