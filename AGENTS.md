# AGENTS.md - Teamistry App Links

This repository hosts Teamistry's production mobile association files,
deep-link fallback pages, and public legal pages at
`app.teamistryclub.com`. Small changes can alter how released iOS and Android
apps open public URLs, so preserve external contracts and follow `RELEASE.md`
for every production change.

## Repo Orientation

- `CNAME` declares the production custom domain.
- `.nojekyll` allows GitHub Pages to publish the `.well-known/` directory.
- `.well-known/apple-app-site-association` is the iOS Universal Links
  association document. It intentionally has no file extension.
- `.well-known/assetlinks.json` is the Android App Links association document.
- `event-master/index.html` is the Event Details browser/store fallback.
- `payments/details/index.html` is the Payment Details browser/store fallback.
- `privacy-policy/index.html` and `terms-of-service/index.html` are public legal
  pages.
- A fallback page's directory must match the corresponding public mobile route.
- There is no build system or package-manager toolchain. GitHub Pages publishes
  static files from the repository root.

Do not edit generated, local, or temporary files such as `.DS_Store` unless the
task explicitly requires it.

## Related Local Repositories

Related Teamistry repositories live under `~/Local/teamistry/` on this machine:

- `~/Local/teamistry/teamistry-app-mobile` owns mobile route handling, route
  builders, associated-domain entitlements, Android intent filters, app/package
  identifiers, signing configuration, and store releases.
- `~/Local/teamistry/teamistry-app-backend` may produce links in notifications
  and messages, but it does not own this host's association files.
- `~/Local/teamistry/teamistry-supabase` owns production database schema, RLS,
  RPCs, and schema-level behavior. This repository owns no database changes.

When a task depends on behavior in another Teamistry repository, inspect that
repository's current local code and documentation before planning or editing.
Do not rely only on summaries or prior knowledge for cross-repository work.

## Before Editing

- Check `git status --short --branch` before changing files.
- Check `git worktree list` before moving, deleting, or treating a nested
  directory as disposable. This repository may have linked worktrees under
  `tmp/`; they are not ordinary scratch directories.
- Read `RELEASE.md` before changing association files, public routes, `CNAME`,
  legal pages, or GitHub Pages behavior.
- For a deep-link change, inspect the corresponding mobile route, route builder,
  native association configuration, and released-client behavior first.
- Confirm the exact case-sensitive public path and query names. Do not normalize,
  rename, or repurpose them casually.
- Keep changes narrowly scoped and preserve unrelated association entries and
  fallback pages.
- Do not change GitHub Pages settings, DNS, the custom domain, HTTPS enforcement,
  repository visibility, or branch protection unless the user explicitly asks
  for that external configuration change.

## Public Contract And Backward Compatibility

Treat all of the following as external production contracts:

- public URL paths and case-sensitive query keys;
- the `app.teamistryclub.com` hostname;
- Apple Team ID and bundle/app identifiers;
- Android package names and SHA-256 signing-certificate fingerprints;
- App Store and Play Store application identifiers and URLs;
- association-file shapes and matching rules;
- fallback-page redirect and no-script behavior;
- public legal-page URLs.

Keep changes backward-compatible by default. Mobile releases and hosted
association changes may not roll out at the same time, and older app versions
can remain active. Prefer additive association entries and new route-specific
fallbacks. If a breaking change or removal is unavoidable, document the mobile
version gate, rollout order, deprecation period, fallback behavior, and rollback
plan before implementation.

## Association And Rollout Safety

- The mobile app must understand a new public route before this host activates
  the route in an association file.
- Being merged to the mobile repository is not by itself proof that a compatible
  build is available to users. Confirm the relevant store release and rollout
  state, or obtain explicit approval for staged activation.
- Keep iOS AASA components query-constrained when the route contract requires
  specific query keys.
- Keep Android package and fingerprint changes additive during signing-key
  transitions unless the old signing identity is conclusively retired.
- Do not remove a live association merely to resolve a short-lived client issue;
  Apple, Android, browser, CDN, and device caches can make deactivation
  non-immediate.
- Association changes do not authorize changes to mobile code, backend link
  producers, Supabase, store configuration, DNS, or GitHub repository settings.

## Fallback Pages, Privacy, And Legal Content

- Fallback pages may preserve the original public URL for app/store handoff, but
  must not fetch, render, infer, or expose private event, team, payment, member,
  or expense data.
- Do not add analytics, pixels, third-party scripts, or logging of full deep-link
  URLs or query values without explicit privacy review and approval.
- Keep desktop and no-script behavior usable and privacy-minimal.
- Reuse the established Teamistry store URLs and fallback behavior unless the
  task explicitly changes the product contract.
- Do not invent, substantively reinterpret, or silently modernize privacy-policy
  or terms-of-service wording. Legal changes require user-provided or explicitly
  approved text and an approved effective date.
- Never commit signing material, access tokens, private keys, store credentials,
  user data, or production secrets.

## Documentation Maintenance

- Keep this file aligned with repository ownership, validation, and safety
  boundaries.
- Keep `RELEASE.md` aligned with the actual GitHub Pages source branch/path,
  deployment checks, rollout gates, rollback procedure, and tag policy.
- Do not add a root `VERSION` file or invent release history unless the project
  explicitly adopts a versioning scheme.
- If a README or changelog is added later, keep it aligned without duplicating
  the detailed release runbook.

## Release Process

- Read `RELEASE.md` before any production merge, Pages deployment, public-route
  activation, association update, legal-page publication, rollback, or tag.
- Production is currently published by GitHub Pages from `main` at repository
  root. Verify that configuration at release time rather than assuming it has
  remained unchanged.
- Do not merge or push `main`, create or move tags, change external hosting
  settings, or perform a rollback unless the user explicitly asks for the
  production operation.
- Use a new reviewed revert commit for rollback. Never force-push or rewrite
  deployed `main` history.

## Validation

Use the narrowest validation that covers the change. At minimum for association
or fallback changes, run:

```bash
jq empty .well-known/apple-app-site-association
jq empty .well-known/assetlinks.json
git diff --check
```

Also verify as applicable:

- `CNAME` remains exactly `app.teamistryclub.com`.
- Changed public paths and query keys match the mobile route contract.
- Association documents retain every unrelated live entry.
- Fallback HTML works for desktop, iOS, Android, and no-script behavior.
- Store URLs and app identifiers are still correct.
- Legal-page titles, contact details, and approved effective dates are correct.

After a production release, validate the live origin files and relevant fallback
URLs, then report Apple/Android cache or real-device checks that remain. Do not
claim Universal Links or App Links are active solely because GitHub Pages built
successfully.
