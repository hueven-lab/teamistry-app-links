# Teamistry App Links Release Guide

This is the source of truth for releasing Teamistry association files,
deep-link fallbacks, and public legal pages hosted at
`https://app.teamistryclub.com`. `AGENTS.md` contains the operator guardrails;
this file contains the production runbook.

Production changes are externally visible and can change how already released
iOS and Android apps handle links. A normal content release updates static files
through GitHub Pages. It does not authorize mobile, backend, Supabase, DNS,
domain, store, or GitHub repository-setting changes.

## Production Shape

- Repository: `hueven-lab/teamistry-app-links`.
- Production hostname: `app.teamistryclub.com`.
- Hosting: GitHub Pages legacy branch deployment.
- Production source: `main`, repository root (`/`).
- Custom-domain source file: `CNAME`.
- Association endpoints:
  - `https://app.teamistryclub.com/.well-known/apple-app-site-association`
  - `https://app.teamistryclub.com/.well-known/assetlinks.json`
- Current public route fallbacks include `/event-master` and
  `/payments/details` when their files are present on deployed `main`.
- Public legal pages include `/privacy-policy` and `/terms-of-service`.

Verify this shape through GitHub and the live origin before each production
release. Do not assume repository settings have remained unchanged.

## Release Identity And Tag Policy

The deployed `main` commit SHA is the canonical identity of an App Links
release. This repository does not use a root `VERSION` file and does not claim
historical release versions.

Tags are optional and must not be created unless the user asks for them. If the
project adopts release tags, use annotated date-based tags in the form:

```text
app-links-YYYY-MM-DD.N
```

For example, `app-links-2026-07-30.1`. Treat published tags as immutable. Do not
move, replace, or force-push a release tag.

## Release Modes

Use the smallest mode that matches the change.

| Mode | Typical Changes | Cross-Repo Gate |
| --- | --- | --- |
| Association metadata | AASA components, Android package/fingerprint statements | Confirm compatible mobile routing, identifiers, signing state, and store rollout. |
| Deep-link fallback | New or changed route `index.html` | Confirm exact mobile path/query contract and privacy-minimal fallback behavior. |
| Legal publication | Privacy policy or terms | Require approved wording and effective date; verify stable public URL. |
| Hosting configuration | `CNAME`, Pages source, HTTPS, DNS, visibility | Separate explicit authorization and a dedicated rollback plan. |

Combining modes does not remove any mode's gate.

## Release Completion Policy

A release is complete only when:

1. the intended commit is on `main`;
2. GitHub Pages reports a successful build for that state;
3. live origin association files match the deployed commit;
4. changed public fallback or legal URLs return the intended content;
5. relevant iOS, Android, browser, and no-script checks are complete or clearly
   reported as pending because of platform caching or device availability; and
6. the final report records the production commit and all remaining follow-ups.

A successful Pages build alone does not prove that Universal Links or App Links
are active on devices.

## 1. Define The Release Contract

Before editing or promoting a release, record the exact intended contract:

- public path;
- case-sensitive query keys and whether they are required;
- matching mobile route and route builder;
- Apple Team ID and bundle/app ID;
- Android package name and signing-certificate fingerprint;
- App Store and Play Store destinations;
- minimum compatible mobile version;
- confirmed store publication/rollout state;
- older-client behavior;
- activation order, monitoring plan, and rollback behavior.

For a new link, the safe default order is:

```text
compatible mobile implementation
  -> mobile store release and sufficient rollout
  -> hosted association activation and fallback publication
  -> real-device verification
```

If the compatible mobile build is not yet available to users, stop before
association activation unless the user explicitly approves a staged rollout and
its consequences.

### `/join-team` receiver-first gate

The short-lived team-invite contract is
`https://app.teamistryclub.com/join-team#token=<43-character Base64URL>`. The
fragment is a bearer secret and is never sent to this static host. Its fallback
has a stricter privacy contract than the existing event fallback: it must not
read, preserve, encode, forward, render, store, log, or append the fragment to
either store destination or any other network request.

Keep the local candidate inactive until all of these release facts are known:

1. a receiver-capable mobile version is available at the required iOS and
   Android rollout level and safely handles valid, invalid, expired, revoked,
   signed-out, and already-member links;
2. the additive database resolve/redeem contract used by that version is live;
3. permanent team-code joining remains available to older apps;
4. the sender-generation flag and public association activation order are
   explicitly approved; and
5. the release owner grants fresh permission for the actual merge/push/Pages
   deployment after reviewing the candidate commit and rollout facts.

Local implementation permission, a prepared feature branch, or an earlier
general release request is not fresh deployment permission. If compatible store
rollout or database readiness is unproved, stop with the candidate local.

## 2. Prepare The Candidate

Start from the intended candidate branch, normally `develop` or a focused
feature branch:

```bash
git fetch origin
git checkout develop
git pull --ff-only origin develop
git status --short --branch
git worktree list
```

The worktree must contain only intentional release changes. Do not delete or
modify another linked worktree to clean the release candidate.

For an association change:

- preserve every unrelated live association entry;
- update the route-specific fallback in the same candidate when needed;
- keep changes additive unless a reviewed deprecation plan says otherwise;
- confirm the mobile route against the current local mobile repository.

For a legal release, use only the approved text and date. Do not silently edit
legal meaning during formatting or cleanup.

## 3. Local Preflight

Run from this repository root:

```bash
jq empty .well-known/apple-app-site-association
jq empty .well-known/assetlinks.json
test "$(cat CNAME)" = "app.teamistryclub.com"
git diff --check
git status --short --branch
```

Review the complete production delta:

```bash
git diff --stat origin/main...HEAD
git diff origin/main...HEAD -- . ':!tmp'
```

Check every affected fallback locally in a browser or a simple static server.
Verify:

- desktop text and links;
- iOS and Android store redirect selection;
- preservation of the original public URL when that is part of the fallback
  contract;
- no-script store links;
- no private-data rendering, network fetches, analytics, or query logging; and
- valid HTML structure and the intended page title.

For `/join-team`, start a local server from the candidate root and use only a
synthetic token:

```bash
python3 -m http.server 8000 --bind 127.0.0.1
```

Open
`http://127.0.0.1:8000/join-team#token=Abcdefghijklmnopqrstuvwxyz0123456789_-ABCDE`
in Chrome. Verify the desktop guidance and both no-script store links. Repeat
with iPhone/iPad and Android user agents while recording requested/navigation
URLs. The mobile redirect must equal the established platform store URL with no
query or fragment suffix. The synthetic token must occur in no request,
redirect destination, page text, console message, storage entry, analytics, or
referrer. Confirm the page makes no fetch/XHR/beacon/pixel request and works at
narrow and wide widths. Keep screenshots and browser logs local and untracked.

## 4. Cross-Repository Compatibility Gate

Inspect `../teamistry-app-mobile` before releasing a deep-link change. Confirm:

- the exact route is registered;
- its route builder emits the same host, path, and case-sensitive query keys;
- malformed or unauthorized links fail safely;
- the iOS associated-domain entitlement still includes the host;
- the Android verified-link intent filter still includes the host;
- app/package identifiers match both association documents; and
- a compatible mobile build has reached the required store rollout state.

If the backend produces the affected link, inspect
`../teamistry-app-backend` for the same URL contract. Association work does not
authorize backend edits or deployment.

No Supabase schema or data change is implied. Database work belongs in
`../teamistry-supabase` and follows that repository's release workflow.

## 5. Approve And Merge To `main`

Use a pull request when possible. Before the production merge, report:

- candidate branch and commit;
- exact files and public paths changing;
- local preflight results;
- compatible mobile version and actual rollout status;
- expected cache/propagation behavior; and
- rollback plan.

Do not merge or push production unless the user explicitly asks for the release.
After approval, update `main` from the reviewed candidate without introducing
unreviewed changes:

```bash
git checkout main
git pull --ff-only origin main
git merge --ff-only develop
git status --short --branch
git push origin main
```

If fast-forward merge is not possible, stop and review the divergence. Do not
force the merge, rewrite `main`, or silently choose a different commit.

Record the intended production SHA:

```bash
git rev-parse HEAD
```

## 6. Verify GitHub Pages Deployment

Confirm the configured Pages source and custom domain:

```bash
gh api repos/hueven-lab/teamistry-app-links/pages
gh api repos/hueven-lab/teamistry-app-links/pages/builds/latest
```

Verify that:

- status is successful/built;
- source remains branch `main`, path `/`;
- CNAME remains `app.teamistryclub.com`; and
- the latest build corresponds to the intended production commit when the API
  exposes that commit.

GitHub Pages currently serves this site through its legacy branch deployment.
A migration to GitHub Actions is a separate hosting change and is not part of a
normal content release.

HTTPS must be reachable with a valid certificate. GitHub Pages has previously
reported HTTPS enforcement disabled for this repository; record the current
value in the release report. Enabling enforcement is recommended but requires a
separately authorized repository-setting change.

## 7. Verify The Live Origin

Association endpoints must return their documents directly without a redirect:

```bash
curl -fsSI https://app.teamistryclub.com/.well-known/apple-app-site-association
curl -fsSI https://app.teamistryclub.com/.well-known/assetlinks.json
curl -fsS https://app.teamistryclub.com/.well-known/apple-app-site-association | jq empty
curl -fsS https://app.teamistryclub.com/.well-known/assetlinks.json | jq empty
```

Compare the live bodies with the deployed worktree:

```bash
diff -u .well-known/apple-app-site-association \
  <(curl -fsS https://app.teamistryclub.com/.well-known/apple-app-site-association)
diff -u .well-known/assetlinks.json \
  <(curl -fsS https://app.teamistryclub.com/.well-known/assetlinks.json)
```

GitHub Pages has historically served the extensionless AASA file as
`application/octet-stream` while serving `assetlinks.json` as JSON. Record the
actual headers. Do not claim a different MIME invariant or change hosting solely
to alter it without verifying Apple/device behavior and obtaining approval.

Check each changed fallback with representative, non-sensitive placeholder
identifiers. GitHub Pages may redirect directory paths to a trailing slash; the
query string must survive that redirect:

```bash
curl -fsSIL 'https://app.teamistryclub.com/event-master?eventId=release-check'
curl -fsSIL 'https://app.teamistryclub.com/payments/details?teamId=release-check&expenseId=release-check'
```

Do not use real event, team, member, expense, or user identifiers in release
logs or documentation.

For legal releases, also open the exact public page and verify its title,
approved effective date, organization identity, and contact details.

## 8. Verify Platform Activation

### iOS

After origin verification, inspect Apple's cached AASA result:

```bash
curl -fsS \
  https://app-site-association.cdn-apple.com/a/v1/app.teamistryclub.com | jq empty
```

Then test the exact HTTPS link on a real device with a compatible installed app.
Confirm the app opens the intended route and that malformed, missing,
unauthorized, or stale identifiers follow the mobile app's safe fallback.

Apple cache propagation can lag the GitHub Pages release. Record the origin
result, Apple CDN result, device result, and time checked. Do not repeatedly
edit or republish association files just to force cache refresh.

For `/join-team`, require the Apple cached document to contain the exact
`/join-team` component before treating iOS association activation as observed.
Origin success with an older Apple CDN body is a pending cache state, not a
failure to work around by republishing. The final Universal Link check requires
a real iPhone with the compatible installed app; a browser redirect proves only
the fallback.

### Android

Inspect Google's Digital Asset Links view when applicable, then re-verify on a
real device:

```bash
adb shell pm verify-app-links --re-verify com.huevenlab.teamistryapp
adb shell pm get-app-links com.huevenlab.teamistryapp
```

Test the exact HTTPS link and confirm the verified app opens the intended route.
Also confirm browser/store fallback behavior when the app is not installed or
verified.

## 9. Rollback

Rollback uses a new reviewed revert commit on `main`. Never force-push, reset,
or rewrite deployed history.

After explicit rollback approval:

```bash
git checkout main
git pull --ff-only origin main
git revert <bad-release-commit>
git push origin main
```

Repeat the Pages, origin, cache, and device verification steps. Association
rollback may not take effect immediately because Apple, Android, browser, CDN,
and device caches can retain earlier state. Preserve a safe fallback page while
changes propagate.

If the incident requires DNS, custom-domain, HTTPS, store, mobile, backend, or
Supabase changes, handle those as separate explicitly approved operations under
their owning repository or service.

For a team-invite rollback, first disable sender generation through its owning
mobile release/configuration process if that action is separately authorized.
Then use a reviewed revert commit for the AASA component and fallback. Preserve
permanent team-code recovery throughout. Apple and Android association caches
may continue opening `/join-team` in installed apps after origin rollback, so
the released mobile receiver must keep invalid/expired/unavailable handling;
never rely on an immediate association-cache purge as the security boundary.

## Final Report Checklist

Report these facts after every production release or rollback:

- release mode;
- candidate branch and commit;
- deployed `main` commit SHA;
- exact files and public paths changed;
- local validation results;
- compatible mobile version and confirmed store rollout state;
- GitHub Pages source, build status, CNAME, and HTTPS-enforcement status;
- live association status, headers, JSON validity, and worktree comparison;
- fallback or legal-page verification;
- Apple CDN and real-device iOS result;
- Digital Asset Links and real-device Android result;
- known cache/propagation delays; and
- remaining manual checks or follow-up work.
