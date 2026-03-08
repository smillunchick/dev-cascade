---
name: dev-ship
description: Use when ready to release a new version — guides through version determination (semver), CHANGELOG update, ROADMAP update, version bump, tagging, build journal, and deployment. Triggers on "release", "ship it", "cut a release", "bump version", "deploy new version".
user-invokable: true
args:
  - name: version
    description: Version to release (e.g., 0.4.0). If omitted, will suggest based on changes.
    required: false
---

# Ship a Release

## Overview

Version, document, tag, deploy. In that order.

**Core principle:** A release is a promise. Document what you're promising.

## Step 1: Determine Version

If version not provided, review changes since last tag and suggest:

```bash
git log $(git describe --tags --abbrev=0 2>/dev/null || echo HEAD~20)..HEAD --oneline
```

| Change Type | Bump | When |
|-------------|------|------|
| Breaking change to config format or API | MAJOR | Schema changes, contract changes |
| New engine, integration, or feature | MINOR | New capabilities |
| Bug fix, performance improvement | PATCH | Existing behavior fixed |

State the suggested version and reasoning. Get user confirmation.

## Step 2: Final Verification

All gates must pass. No exceptions for releases.

```bash
npm run build && npm run typecheck && npm test
```

## Step 3: Update CHANGELOG.md

Add entry for the new version, grouped by type:

```markdown
## [X.Y.Z] - YYYY-MM-DD

### Added
- Description of feat commits

### Fixed
- Description of fix commits

### Changed
- Description of refactor/perf commits
```

Derive from `git log` since last tag. Be specific — "add TTL-based memory expiration" not "memory improvements."

## Step 4: Bump Version

```bash
npm version X.Y.Z --no-git-tag-version
```

## Step 5: Update ROADMAP.md

- Move completed items to done
- Update status markers
- Set target version for remaining items

## Step 6: Build Journal

For MINOR+ releases with significant changes (new engines, integrations, architectural shifts):
- Add entry to `docs/build-journal.md`
- Document key decisions, anti-patterns avoided, lessons learned

Skip for PATCH releases unless there's something noteworthy.

## Step 7: Commit and Tag

```bash
git add -A
git commit -m "chore: bump version to X.Y.Z"
git tag vX.Y.Z
```

## Step 8: Deploy

Follow the project's deployment process documented in `CLAUDE.md` or `docs/operations.md`.

After deploy, verify the service is running:
- Check service status
- Tail logs for startup errors
- Smoke test core functionality

## Release Checklist

- [ ] Version determined and confirmed
- [ ] All gates pass (build, typecheck, test)
- [ ] CHANGELOG.md updated with grouped entries
- [ ] Version bumped in package.json
- [ ] ROADMAP.md updated
- [ ] Build journal updated (MINOR+ only)
- [ ] Committed: `chore: bump version to X.Y.Z`
- [ ] Tagged: `vX.Y.Z`
- [ ] Deployed and verified

## Cascade

**After shipping:**
- On a feature branch → `/dev-finish` to merge
- Starting next work → `/dev-start`

**All dev skills:** `~/.claude/skills/dev-*` — run `/dev-teach` to set up a new project.
