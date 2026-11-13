<!--
Sync Impact Report
- Version change: unratified template → 1.0.0
- Modified principles:
  - [PRINCIPLE_1_NAME] → I. Untrusted Input and Locked-Down Origins
  - [PRINCIPLE_2_NAME] → II. Backend and Frontend Boundaries
  - [PRINCIPLE_3_NAME] → III. Phase-Scoped Features
  - [PRINCIPLE_4_NAME] → IV. Proven Local Configuration
  - [PRINCIPLE_5_NAME] → V. Buildable, Dead-Code-Free Changes
- Added sections:
  - Technology and Security Constraints (replaces [SECTION_2_NAME])
  - Quality Gates (replaces [SECTION_3_NAME])
- Removed sections: none (placeholder headings were replaced in place)
- Follow-up TODOs: none
-->

# RSS Feed Reader Constitution

## Core Principles

### I. Untrusted Input and Locked-Down Origins

Feed URLs and feed content are untrusted. A single-user local reader does
not relax input handling.

- CORS MUST allow only the frontend origins used for local development.
  Defaults are `http://localhost:5213` and the matching HTTPS launch
  origin. A wildcard origin is forbidden.
- The Blazor client MUST read `ApiBaseUrl` from
  `frontend/RSSFeedReader.UI/wwwroot/appsettings.json`. Components MUST
  NOT hardcode the API host.
- Feed fetches, once added, MUST run on the ASP.NET Core API. The browser
  MUST NOT receive a general-purpose outbound proxy.
- Item title and link MUST render as plain text. Feed HTML MUST NOT be
  injected into the page until HTML sanitization is an explicit scheduled
  phase.
- Secrets and connection strings MUST NOT be committed. Checked-in
  configuration is limited to local ports and public sample feed URLs.

Rationale: Broad CORS and unsanitized feed markup become defects as soon
as the app requests third-party feeds. These rules are checkable on the
MVP, before that code exists.

### II. Backend and Frontend Boundaries

`RSSFeedReader.Api` owns subscription state and, from Extended-MVP onward,
feed retrieval and parsing. `RSSFeedReader.UI` owns the form, the
subscription list, manual refresh, and item display.

- The canonical subscription list MUST live in the API. The UI MUST change
  that list only through the HTTP API.
- MVP storage MUST be an in-memory collection on the API. Losing that data
  when the process exits is accepted for MVP and Extended-MVP.
- EF Core, SQLite, and `BackgroundService` MUST wait until a post-MVP
  phase explicitly adds persistence or polling.
- Feed parsing MUST use `System.ServiceModel.Syndication` once fetching
  exists. A second parser MUST NOT be added for the same RSS/Atom job.

Rationale: One owner per responsibility keeps the proof of concept small
and leaves room for persistence and polling without a rewrite.

### III. Phase-Scoped Features

A feature ships only in the phase that names it.

- MVP MUST add a subscription by URL and show the list. It MUST NOT fetch,
  parse, validate, delete, or persist subscriptions.
- Extended-MVP MUST add a manual refresh, show each item's title and link,
  and report failure as "Failed to load feed". It MUST NOT poll in the
  background.
- Post-MVP work (delete, database, read/unread, folders, OPML, rich HTML,
  site-to-feed discovery, notifications) MUST stay out of the codebase
  until that phase is scheduled in the feature spec.

Rationale: Stakeholder scope defines a working MVP as add-and-list. Work
outside that phase hides wiring bugs and slows the proof of concept.

### IV. Proven Local Configuration

A change is not done until the local stack is wired and observed.

- The backend listen URL, frontend listen URL, `ApiBaseUrl`, and CORS
  origins MUST name the same hosts and ports. Defaults are API
  `http://localhost:5151`, UI `http://localhost:5213`, and `ApiBaseUrl`
  `http://localhost:5151/api/`.
- The browser console MUST show no connection errors when the UI loads.
- Blazor template pages `Home.razor`, `Counter.razor`, and `Weather.razor`
  MUST be deleted, and their nav links MUST be removed, before any feature
  page is added. Exactly one page MUST use `@page "/"`.
- An ambiguous-route exception means cleanup is incomplete. Feature work
  MUST stop until `dotnet clean` and `dotnet build` of the UI project
  succeed and the UI starts without that exception.
- Extended-MVP fetch behavior MUST be checked with
  `https://devblogs.microsoft.com/dotnet/feed/`.

Rationale: Port drift and leftover template routes are the blocking
failures called out for local development. Both are visible in a browser
session.

### V. Buildable, Dead-Code-Free Changes

- Every change MUST compile with `dotnet build` for each project it
  touches before it is treated as complete.
- Demo pages and unused template members MUST be deleted. Commented-out
  template code MUST NOT remain in the project.
- The MVP HTTP contract is add-subscription and list-subscriptions. A
  route or payload change MUST be written into the feature spec in the
  same change.
- MVP MUST NOT add error UI for network calls that do not exist.
  Extended-MVP MUST show "Failed to load feed" and MUST NOT dump remote
  response bodies into the UI.

Rationale: A green build, one contract, and no leftover demo code are the
quality bar this project can enforce on every change.

## Technology and Security Constraints

- The stack is ASP.NET Core Web API plus Blazor WebAssembly, in C#, on
  Windows, macOS, or Linux. A different UI framework or API stack requires
  a constitution amendment before use.
- MVP MUST NOT take a dependency on `HttpClient` or a syndication package.
  Those dependencies enter with Extended-MVP only.
- Outbound feed requests, once added, MUST use a finite `HttpClient`
  timeout and MUST request only a URL already stored as a subscription.
- Logs MUST NOT record full feed bodies. Logs MAY record the subscription
  URL and a short failure reason.
- The app stays single-user and local. Authentication, multi-user access,
  and remote deployment require an amendment before implementation.

## Quality Gates

Before implementation of a feature phase:

1. Confirm template-page cleanup from Principle IV is already done.
2. State which phase the work belongs to (Principle III).
3. Name the API operations the UI will call.

Before the phase is called complete:

1. `dotnet build` succeeds for the API project and the UI project.
2. The UI loads with no console connection errors.
3. MVP: adding a URL shows that URL in the subscription list.
4. Extended-MVP: a manual refresh of
   `https://devblogs.microsoft.com/dotnet/feed/` shows titles and links,
   and a failed fetch shows "Failed to load feed".
5. Review confirms CORS origins, `ApiBaseUrl`, plain-text rendering, and
   the absence of features reserved for a later phase.

## Governance

This constitution overrides conflicting engineering practice in specs,
plans, and implementation. Product intent lives in
`StakeholderDocuments/ProjectGoals.md`,
`StakeholderDocuments/AppFeatures.md`, and
`StakeholderDocuments/TechStack.md`. If those documents and this
constitution disagree on an engineering rule, this constitution wins
until it is amended.

Amendments are made by editing `.specify/memory/constitution.md` and
bumping the version line in the same change. Reviewers MUST reject a
change that breaks a MUST rule unless the amendment is already merged.

Versioning follows semantic versions:

- MAJOR: a principle is removed or redefined so that existing specs would
  violate it.
- MINOR: a principle or section is added, or a rule is materially
  expanded.
- PATCH: wording, clarification, or a typo only.

Every feature plan and implementation pass MUST check phase scope, origin
lockdown, the API/UI boundary, local wiring, and a successful build.
Behavior beyond the current phase is out of scope unless the feature spec
justifies it and names the phase that allows it.

**Version**: 1.0.0 | **Ratified**: 2026-11-13 | **Last Amended**: 2026-11-13
