---
name: changelog
description: Generate a user-facing changelog or release notes from git history between two refs. Categorizes by type, translates technical commits into customer-friendly summaries, and groups breaking changes at the top. Use when cutting a release.
disable-model-invocation: true
allowed-tools: "Bash, Read, Write, Grep"
---

# Changelog — Release Notes Generator

## Step 1 — Determine range

Ask the user (if not provided):
- From ref (default: latest tag — `git describe --tags --abbrev=0`).
- To ref (default: `HEAD`).
- Audience: end-user, library consumer, or internal? (changes voice).

## Step 2 — Read history in parallel

1. `git log <from>..<to> --pretty=format:'%H|%s|%b|||'` — commits with bodies.
2. `git log <from>..<to> --merges --pretty=format:'%H %s'` — merge commits (often have PR titles).
3. `git diff <from>..<to> --stat` — scale of the release.
4. Read existing `CHANGELOG.md` if present — match the existing format/voice.

## Step 3 — Categorize

Bucket each commit into:

- **Breaking changes** — anything with `!` after type or `BREAKING CHANGE:` footer.
- **Features** — `feat:` commits.
- **Bug fixes** — `fix:` commits.
- **Performance** — `perf:` commits.
- **Security** — anything mentioning CVE, vuln, or in a security path.
- **Deprecations** — explicit deprecation notes.
- **Internal** — `refactor`, `chore`, `test`, `ci`, `build`, `docs` (collapse into one line unless the audience is "internal").

Drop noise:
- Lockfile-only commits.
- "Merge branch …" with no other content.
- Reverts where the reverted commit is also in the range (cancel out).

## Step 4 — Translate

For each surviving entry:
- Strip the conventional-commit prefix.
- Rewrite from "added X" → "X" (changelog voice).
- For features and breaking changes, write what the *user* notices, not what the diff did.
- Cite PR or commit short SHA at the end of the line.

## Step 5 — Compose

Use this template (or match `CHANGELOG.md` if present):

```markdown
## <version> — <date>

### ⚠ BREAKING CHANGES
- <change> — what users must do — (#PR)

### Added
- <feature> — (#PR)

### Fixed
- <bug fix in user-facing terms> — (#PR)

### Performance
- <improvement, ideally with a number> — (#PR)

### Security
- <CVE / advisory addressed> — (#PR)

### Deprecated
- <feature> — replaced by <X> — removal planned in <version>

### Internal
<one-line summary like "12 refactors, 8 doc updates, 4 CI improvements">

### Upgrade notes
<concrete steps if BREAKING CHANGES is non-empty, else omit>
```

## Step 6 — Write and surface

Write/prepend to `CHANGELOG.md` (or `RELEASE_NOTES.md` for a one-off). Surface to the user:
1. Total commits processed and dropped.
2. The new section.
3. Confirmation prompt to prepend vs write to new file.

## Stop conditions

- Never invent user-facing language for a commit whose message is opaque — flag those for manual review with a `<!-- review: SHA -->` marker.
- Never drop a `BREAKING CHANGE:` entry, even if the commit body is unclear.
- Never publish a release tag or push — generation only.
